# Retrieving the start and end dates for all 4 quarters in a year

## 🎯 Exercise
Retrieve the start and end dates for all 4 quarters of the current year — dynamically, without hardcoding any dates.

---

## 📝 Note

> The output shown below was captured in **2024**. Since `GETDATE()` is used, the query automatically reflects the current year when run.
>
> This exercise is related to [Retrieving the start and end dates for a specific quarter](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/Dates/Retrieving%20start%20and%20end%20dates%20for%20a%20specific%20quarter.md), which uses `UNION ALL` and hardcoded year-quarter values. This exercise instead uses an iteration approach with `ROW_NUMBER()` and a `WHERE` clause — making it fully dynamic.

---

## 💡 Solution

### Approach
We calculate two anchor dates — the start of Q1 and Dec 31 of the previous year — using `DATEADD(QUARTER, ...)` and `DATEADD(DAY, -1, ...)`. We then use a Cartesian Product with an `Iteration` subquery that generates sequential position numbers. Adding the position to the quarter-based `DATEADD()` calls shifts each anchor forward by that many quarters. A `WHERE` clause limits the iteration to 4 rows — one per quarter.

### T-SQL functions and clauses used

| Function | Purpose |
|---|---|
| `GETDATE()` | Returns the current date and time |
| `DATEDIFF(YEAR, 0, GETDATE())` | Number of years between `1900-01-01` and today |
| `DATEADD(YEAR, n, 0)` | Jan 1 of the current year |
| `DATEDIFF(QUARTER, 0, Jan1ThisYear)` | Number of quarters between `1900-01-01` and Jan 1 this year |
| `DATEADD(QUARTER, n, 0)` | Adds `n` quarters to `1900-01-01` — landing on a quarter start date |
| `DATEADD(DAY, -1, Jan1ThisYear)` | Dec 31 of the previous year — used as the end-date anchor |
| `DATEADD(QUARTER, Position, EndDateAnchor)` | Shifts the end-date anchor forward by `Position` quarters |
| `ROW_NUMBER() OVER (ORDER BY BusinessEntityID)` | Generates sequential position numbers for iteration |
| `WHERE Iteration.Position <= 4` | Limits the result to 4 rows — one per quarter |
| `SELECT DISTINCT` | Collapses the source table into a single anchor row |

### Table used

| Schema | Table |
|---|---|
| `Person` | `BusinessEntity` |

---

### T-SQL code

```sql
SELECT
    Iteration.Position AS QuarterNum
  , CAST(DATEADD(QUARTER,
        DATEDIFF(QUARTER, 0, DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0))
        + Iteration.Position,
        0) AS DATE) AS QtrStartDate
  , CAST(DATEADD(QUARTER,
        Iteration.Position,
        DATEADD(DAY, -1, DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0))) AS DATE) AS QtrEndDate
FROM (
    SELECT DISTINCT
        DATEADD(QUARTER,
            DATEDIFF(QUARTER, 0, DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0)) + 0,
            0) AS Quarter1StartDate
      , DATEADD(DAY, -1, DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0)) AS Quarter1EndDate
    FROM [AdventureWorks2022].[Person].[BusinessEntity]
) AS QuarterStartEndDates,
(
    SELECT ROW_NUMBER() OVER (ORDER BY BusinessEntityID) AS Position
    FROM [AdventureWorks2022].[Person].[BusinessEntity]
) AS Iteration
WHERE Iteration.Position <= 4
```

---

### Output

```
QuarterNum  QtrStartDate  QtrEndDate
1           2024-04-01    2024-03-31
2           2024-07-01    2024-06-30
3           2024-10-01    2024-09-30
4           2025-01-01    2024-12-31
(4 rows affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — Calculate the two anchor dates

We calculate two anchor dates that drive all quarter date calculations:

**T-SQL code:**
```sql
SELECT DISTINCT
    DATEADD(QUARTER,
        DATEDIFF(QUARTER, 0, DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0)) + 0,
        0) AS Quarter1StartDate
  , DATEADD(DAY, -1, DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0)) AS Quarter1EndDate
FROM [AdventureWorks2022].[Person].[BusinessEntity]
```

**Output:**

```
Quarter1StartDate             Quarter1EndDate
2024-01-01 00:00:00.000       2023-12-31 00:00:00.000
```

**How the two anchors work:**

**Anchor 1 — `Quarter1StartDate` = Jan 1 of the current year:**
`DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0)` calculates Jan 1 of the current year (the same technique used in earlier exercises). `DATEDIFF(QUARTER, 0, Jan1ThisYear)` counts the quarters between `1900-01-01` and Jan 1 this year. `DATEADD(QUARTER, n + 0, 0)` lands back on Jan 1 of the current year — adding `0` quarters keeps it at Q1 start.

**Anchor 2 — `Quarter1EndDate` = Dec 31 of the previous year:**
`DATEADD(DAY, -1, Jan1ThisYear)` subtracts 1 day from Jan 1 this year — giving Dec 31 of last year. This is used as the end-date anchor because adding `n` quarters to Dec 31 always lands on the last day of the corresponding quarter.

---

### Query 1.1.1 — How the anchor dates shift per quarter

Adding `Position` quarters to each anchor produces the correct start and end date for each quarter:

**Start dates** (adding quarters to Jan 1 this year):
**T-SQL code:**
```sql
	-- 1-JAN-2024
SELECT DATEADD(QUARTER, DATEDIFF(QUARTER, 0, DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0)) + 0, 0)		-- <--- used as QuarterStartEndDatesLevel1, see Query #1.1

	-- 1-APR-2024
SELECT DATEADD(QUARTER, DATEDIFF(QUARTER, 0, DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0)) + 1, 0)

	-- 1-JUL-2024
SELECT DATEADD(QUARTER, DATEDIFF(QUARTER, 0, DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0)) + 2, 0)

	-- 1-OCT-2024
SELECT DATEADD(QUARTER, DATEDIFF(QUARTER, 0, DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0)) + 3, 0)
```

```sql
-- Q1 start: + 0 quarters → 2024-01-01
DATEADD(QUARTER, n + 0, 0)
-- Q2 start: + 1 quarter  → 2024-04-01
DATEADD(QUARTER, n + 1, 0)
-- Q3 start: + 2 quarters → 2024-07-01
DATEADD(QUARTER, n + 2, 0)
-- Q4 start: + 3 quarters → 2024-10-01
DATEADD(QUARTER, n + 3, 0)
```

**End dates** (adding quarters to Dec 31 last year):

```sql
-- Q1 end: + 1 quarter  → 2024-03-31
DATEADD(QUARTER, 1, '2023-12-31')
-- Q2 end: + 2 quarters → 2024-06-30
DATEADD(QUARTER, 2, '2023-12-31')
-- Q3 end: + 3 quarters → 2024-09-30
DATEADD(QUARTER, 3, '2023-12-31')
-- Q4 end: + 4 quarters → 2024-12-31
DATEADD(QUARTER, 4, '2023-12-31')
```

> **Why does adding quarters to Dec 31 always land on the last day of the quarter?** Dec 31 is the last day of Q4. Adding exactly 1 quarter moves to Mar 31 (last day of Q1), adding 2 quarters moves to Jun 30 (last day of Q2), and so on — always landing on a quarter-end date.

---

### Query 1.2 — Generate sequential position numbers using `ROW_NUMBER()`

`ROW_NUMBER() OVER (ORDER BY BusinessEntityID)` generates 20,777 sequential integers from the `BusinessEntity` table. Only the first 4 are needed — controlled by `WHERE Iteration.Position <= 4`.

---

### Query 1.3 — Generate start and end dates for all 20,777 quarters (before filtering)

Without the `WHERE` clause the Cartesian Product between the single-row anchor subquery and the full `BusinessEntity` table generates start and end dates for 20,777 consecutive quarters — stretching all the way to the year 7218.

**Output (truncated — without WHERE clause):**

```
QuarterNum  QtrStartDate  QtrEndDate
1           2024-04-01    2024-03-31
2           2024-07-01    2024-06-30
3           2024-10-01    2024-09-30
4           2025-01-01    2024-12-31
5           2025-04-01    2025-03-31
...
20777       7218-04-01    7218-03-31
(20777 rows affected)
```

Adding `WHERE Iteration.Position <= 4` limits the output to the 4 quarters of the current year.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
