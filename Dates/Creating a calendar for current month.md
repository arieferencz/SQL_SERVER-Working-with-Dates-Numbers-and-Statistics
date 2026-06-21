# Creating a calendar for current month

## 🎯 Exercise
Generate a visual calendar grid for the current month — with days of the week as columns (Mo, Tu, We, Th, Fr, Sa, Su) and each week as a row, showing the day number in the correct cell.

---

## 📝 Note

> The output shown below was captured in **December 2024**. Since `GETDATE()` is used, the calendar will automatically reflect the current month when the query is run.

---

## 💡 Solution

### Approach
We build the calendar in 4 steps:
1. Generate all calendar dates for the current month using `GENERATE_SERIES()`
2. Assign each date its weekday number, day of month, and week number using `DATEPART()`
3. Pivot the day numbers across 7 columns (one per weekday) using `CASE` statements
4. Collapse each week into a single row using `GROUP BY WK` and `MAX()` to remove `NULL`s

### T-SQL functions, case expressions, and clauses used

| Function | Purpose |
|---|---|
| `DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()), 0)` | Calculates the first day of the current month |
| `DATEADD(MONTH, DATEDIFF(MONTH, -1, GETDATE()), -1)` | Calculates the last day of the current month |
| `GENERATE_SERIES(start, stop, step)` | Generates sequential integers — one per day of the month |
| `DATEADD(DAY, value, start_date)` | Converts each integer into a calendar date |
| `DAY(date)` | Returns the day number of the month (1–31) |
| `DATEPART(WEEKDAY, date)` | Returns the weekday number (1 = Sunday, 2 = Monday, ... 7 = Saturday) |
| `DATEPART(WEEK, date)` | Returns the week number of the year |
| `DATEPART(ISO_WEEK, date)` | Returns the ISO week number |
| `DATENAME(WEEKDAY, date)` | Returns the weekday name (e.g. `'Monday'`) |
| `CASE WHEN DayOfWeek = 1 THEN WEEK - 1 ELSE WEEK END` | Adjusts week numbers so Sunday rows are grouped with the correct week |
| `CASE DayOfWeek WHEN n THEN DayOfMonth END` | Pivots each day number into its correct weekday column |
| `MAX(CASE ...)` | Collapses multiple rows per week into one — removing `NULL`s |
| `SELECT DISTINCT` | Ensures subqueries return a single date value |

### Table used

| Schema | Table |
|---|---|
| `Person` | `BusinessEntity` |

---

### T-SQL code — Final query

```sql
SELECT
    MAX(CASE Y.DayOfWeek WHEN 2 THEN Y.DayOfMonth END) AS Mo
  , MAX(CASE Y.DayOfWeek WHEN 3 THEN Y.DayOfMonth END) AS Tu
  , MAX(CASE Y.DayOfWeek WHEN 4 THEN Y.DayOfMonth END) AS We
  , MAX(CASE Y.DayOfWeek WHEN 5 THEN Y.DayOfMonth END) AS Th
  , MAX(CASE Y.DayOfWeek WHEN 6 THEN Y.DayOfMonth END) AS Fr
  , MAX(CASE Y.DayOfWeek WHEN 7 THEN Y.DayOfMonth END) AS Sa
  , MAX(CASE Y.DayOfWeek WHEN 1 THEN Y.DayOfMonth END) AS Su
FROM (
    SELECT
        DATENAME(WEEKDAY, X.GeneratedDates)        AS GeneratedWeekDays
      , X.GeneratedDates
      , DAY(X.GeneratedDates)                      AS DayOfMonth
      , DATEPART(MONTH,   X.GeneratedDates)        AS CurrentMonth
      , DATEPART(WEEKDAY, X.GeneratedDates)        AS DayOfWeek
      , DATEPART(ISO_WEEK, X.GeneratedDates)       AS ISOWeek
      , CASE
            WHEN DATEPART(WEEKDAY, X.GeneratedDates) = 1
            THEN DATEPART(WEEK, X.GeneratedDates) - 1
            ELSE DATEPART(WEEK, X.GeneratedDates)
        END                                        AS WK
    FROM (
        SELECT
            CAST(DATEADD(DAY, value,
                (SELECT DISTINCT DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()), 0)
                 FROM [AdventureWorks2022].[Person].[BusinessEntity])
            ) AS DATE) AS GeneratedDates
        FROM GENERATE_SERIES(
            0,
            DATEDIFF(DAY,
                (SELECT DISTINCT DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()), 0)
                 FROM [AdventureWorks2022].[Person].[BusinessEntity]),
                (SELECT DISTINCT DATEADD(MONTH, DATEDIFF(MONTH, -1, GETDATE()), -1)
                 FROM [AdventureWorks2022].[Person].[BusinessEntity])
            ),
            1
        )
    ) AS X
) AS Y
GROUP BY Y.WK
ORDER BY Y.WK
```

---

### Output — December 2024

```
Mo    Tu    We    Th    Fr    Sa    Su
NULL  NULL  NULL  NULL  NULL  NULL  1
2     3     4     5     6     7     8
9     10    11    12    13    14    15
16    17    18    19    20    21    22
23    24    25    26    27    28    29
30    31    NULL  NULL  NULL  NULL  NULL

Warning: Null value is eliminated by an aggregate or other SET operation.
(6 rows affected)
```

> The warning is expected — it appears because `MAX()` aggregates columns that contain `NULL` values (the empty calendar cells). This does not affect the correctness of the result.

---

## 🔍 Step-by-step explanation

### Query 1.1 — Calculate the first and last day of the current month

```sql
SELECT CAST(DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()), 0) AS DATE)    AS FirstDayofCurrentMonth
SELECT CAST(DATEADD(MONTH, DATEDIFF(MONTH, -1, GETDATE()), -1) AS DATE)  AS LastDayofCurrentMonth
```

```
FirstDayofCurrentMonth   LastDayofCurrentMonth
2024-12-01               2024-12-31
```

These are the same boundary calculations used in the **"Calculate first day, last day and second to last day of current month"** exercise.

---

### Query 1.2 — Generate all dates for the current month

`GENERATE_SERIES(0, 30, 1)` generates integers from `0` to `30` (31 days in December). `DATEADD(DAY, value, '2024-12-01')` converts each into a calendar date.

**Output (truncated):** 31 rows — every day in December 2024.

```
GeneratedDates
2024-12-01
2024-12-02
...
2024-12-31
(31 rows affected)
```

---

### Query 1.3 — Add weekday and week number columns

We add several calculated columns to each date to prepare for pivoting:

| Column | Function | Purpose |
|---|---|---|
| `DayOfMonth` | `DAY(date)` | The calendar day number (1–31) — this is what appears in the calendar cell |
| `DayOfWeek` | `DATEPART(WEEKDAY, date)` | Weekday number (1=Sun, 2=Mon, ..., 7=Sat) — determines which column |
| `ISOWeek` | `DATEPART(ISO_WEEK, date)` | ISO week number — for reference |
| `WK` | `CASE` on `DATEPART(WEEK, date)` | Week number adjusted so Sundays group with the correct calendar row |

**Why adjust `WK` for Sundays?**
In SQL Server, `DATEPART(WEEKDAY, date)` treats Sunday as day `1`. This means a Sunday that falls at the end of a week (e.g. Dec 8) would get a different `WEEK` number than the Monday-Saturday days in the same calendar row. The `CASE` statement subtracts `1` from the week number when `DayOfWeek = 1` (Sunday), keeping it aligned with the rest of its calendar week.

**Output (truncated):**

```
GeneratedWeekDays  GeneratedDates  DayOfMonth  CurrentMonth  DayOfWeek  ISOWeek  WK
Sunday             2024-12-01      1           12            1          48       48
Monday             2024-12-02      2           12            2          49       49
Tuesday            2024-12-03      3           12            3          49       49
Wednesday          2024-12-04      4           12            4          49       49
Thursday           2024-12-05      5           12            5          49       49
Friday             2024-12-06      6           12            6          49       49
Saturday           2024-12-07      7           12            7          49       49
Sunday             2024-12-08      8           12            1          49       49
Monday             2024-12-09      9           12            2          50       50
...
Monday             2024-12-30      30          12            2          1        1
Tuesday            2024-12-31      31          12            3          1        1
(31 rows affected)
```

---

### Query 1.4 — Pivot day numbers across 7 weekday columns using `CASE`

For each row, 7 `CASE` statements check `DayOfWeek` and place `DayOfMonth` in the matching column, leaving `NULL` in all others.

**Output (truncated):** 31 rows — one per day, each with its day number in one column and `NULL` in the other six.

```
Mo    Tu    We    Th    Fr    Sa    Su
NULL  NULL  NULL  NULL  NULL  NULL  1
2     NULL  NULL  NULL  NULL  NULL  NULL
NULL  3     NULL  NULL  NULL  NULL  NULL
...
NULL  NULL  NULL  NULL  NULL  NULL  29
30    NULL  NULL  NULL  NULL  NULL  NULL
NULL  31    NULL  NULL  NULL  NULL  NULL
(31 rows affected)
```

---

### Final Query — Collapse rows by week using `GROUP BY WK` and `MAX()`

`GROUP BY Y.WK` groups all rows belonging to the same calendar week. `MAX(CASE ...)` on each column picks up the one non-`NULL` day number per weekday column — collapsing 31 rows into 6 week rows and removing all `NULL`s within each week.

**Final output — December 2024 calendar:**

```
Mo    Tu    We    Th    Fr    Sa    Su
NULL  NULL  NULL  NULL  NULL  NULL  1     ← week of Dec 1 (Sunday only)
2     3     4     5     6     7     8
9     10    11    12    13    14    15
16    17    18    19    20    21    22
23    24    25    26    27    28    29
30    31    NULL  NULL  NULL  NULL  NULL  ← week of Dec 30-31 (Mon-Tue only)
(6 rows affected)
```

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
