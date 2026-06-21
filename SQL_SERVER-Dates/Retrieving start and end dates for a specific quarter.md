# Retrieving the start and end dates for a specific quarter

## 🎯 Exercise
Retrieve the start and end dates for all 4 quarters of a specific year — in this case, **2024**.

---

## 💡 Solution

### Approach
We use `UNION ALL` to generate 4 rows — one per quarter — each encoded as a 5-digit integer (`YYYYQ`, e.g. `20241` for Q1 2024). We extract the year using `LEFT()` and the quarter number using the modulus operator `%`. Multiplying the quarter number by `3` gives the last month of that quarter. We then cast this into a date (the first day of that last month), subtract 2 months to get the quarter start, and add 1 month then subtract 1 day to get the quarter end.

### T-SQL functions and operators used

| Function / Operator | Purpose |
|---|---|
| `UNION ALL` | Generates 4 rows — one per quarter |
| `SELECT DISTINCT` | Collapses the 20,777 `BusinessEntity` rows into a single value per `UNION ALL` branch |
| `LEFT(YRQ, 4)` | Extracts the first 4 characters — the year |
| `YRQ % 10` | Modulus — returns the last digit, which is the quarter number (1–4) |
| `YRQ % 10 * 3` | Quarter number × 3 = last month of that quarter (Q1→3, Q2→6, Q3→9, Q4→12) |
| `CAST(year + '-' + month + '-1' AS DATE)` | Constructs the first day of the last month of the quarter |
| `DATEADD(MONTH, -2, FirstDayOfLastMonth)` | Subtracts 2 months → first day of the quarter |
| `DATEADD(DAY, -1, DATEADD(MONTH, 1, FirstDayOfLastMonth))` | Adds 1 month then subtracts 1 day → last day of the quarter |

### Table used

| Schema | Table |
|---|---|
| `Person` | `BusinessEntity` |

---

### T-SQL code — Final query

```sql
SELECT
    DATEADD(MONTH, -2, Z.FirstDayOfLastMonthOfQuarter)                             AS DateFirstDayOfQuarter
  , DATEADD(DAY, -1, DATEADD(MONTH, 1, Z.FirstDayOfLastMonthOfQuarter))            AS DateLastDayOfQuarter
FROM (
    SELECT
        CAST(Y.[YEAR] AS VARCHAR)                                                   AS YearOfLastMonthOfQuarter
      , CAST(Y.[MONTH] AS VARCHAR)                                                  AS MonthOfLastMonthOfQuarter
      , CAST(Y.[YEAR] AS VARCHAR) + '-' + CAST(Y.[MONTH] AS VARCHAR) + '-1'         AS DateOfLastMonthOfQuarterCAST
      , CAST(CAST(Y.[YEAR] AS VARCHAR) + '-' + CAST(Y.[MONTH] AS VARCHAR) + '-1'
            AS DATE)                                                                AS FirstDayOfLastMonthOfQuarter
    FROM (
        SELECT LEFT(X.YRQ, 4) AS [YEAR]
             , X.YRQ % 10     AS [QUARTER]
             , X.YRQ % 10 * 3 AS [MONTH]
        FROM (
            SELECT DISTINCT 20241 AS YRQ FROM [AdventureWorks2022].[Person].[BusinessEntity]
            UNION ALL
            SELECT DISTINCT 20242 AS YRQ FROM [AdventureWorks2022].[Person].[BusinessEntity]
            UNION ALL
            SELECT DISTINCT 20243 AS YRQ FROM [AdventureWorks2022].[Person].[BusinessEntity]
            UNION ALL
            SELECT DISTINCT 20244 AS YRQ FROM [AdventureWorks2022].[Person].[BusinessEntity]
        ) AS X
    ) AS Y
) AS Z
```

---

### Output

```
DateFirstDayOfQuarter  DateLastDayOfQuarter
2024-01-01             2024-03-31
2024-04-01             2024-06-30
2024-07-01             2024-09-30
2024-10-01             2024-12-31
(4 rows affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — Generate 4 rows, one per quarter, using `UNION ALL`

We hardcode 4 integer values — one per quarter — each combining the year and quarter number into a single 5-digit code (`YYYYQ`). `SELECT DISTINCT` collapses each `BusinessEntity` table into a single row.

```sql
SELECT DISTINCT 20241 AS YRQ FROM [AdventureWorks2022].[Person].[BusinessEntity]
UNION ALL
SELECT DISTINCT 20242 AS YRQ FROM [AdventureWorks2022].[Person].[BusinessEntity]
UNION ALL
SELECT DISTINCT 20243 AS YRQ FROM [AdventureWorks2022].[Person].[BusinessEntity]
UNION ALL
SELECT DISTINCT 20244 AS YRQ FROM [AdventureWorks2022].[Person].[BusinessEntity]
```

**Output:** 4 rows — one integer per quarter.

```
YRQ
20241
20242
20243
20244
(4 rows affected)
```

---

### Query 1.2 — Extract year, quarter number, and last month of each quarter

`LEFT(YRQ, 4)` extracts the year (`2024`). `YRQ % 10` extracts the last digit — the quarter number (`1`, `2`, `3`, `4`). `YRQ % 10 * 3` multiplies the quarter number by 3 — giving the **last month** of each quarter:

| Quarter | `YRQ % 10` | `YRQ % 10 × 3` = Last month |
|---|---|---|
| Q1 | 1 | 3 → March |
| Q2 | 2 | 6 → June |
| Q3 | 3 | 9 → September |
| Q4 | 4 | 12 → December |

**Output:**

```
YEAR  QUARTER  MONTH
2024  1        3
2024  2        6
2024  3        9
2024  4        12
(4 rows affected)
```

---

### Query 1.3 — Construct the first day of the last month of each quarter

We concatenate the year, month, and `-1` into a date string (e.g. `'2024-3-1'`) and cast it as a `DATE`.

**Output:**

```
YearOfLastMonthOfQuarter  MonthOfLastMonthOfQuarter  DateOfLastMonthOfQuarterCAST  FirstDayOfLastMonthOfQuarter
2024                      3                          3/1/2024                      2024-03-01
2024                      6                          6/1/2024                      2024-06-01
2024                      9                          9/1/2024                      2024-09-01
2024                      12                         12/1/2024                     2024-12-01
(4 rows affected)
```

---

### Final Query (Query 1.4) — Calculate start and end dates for each quarter

Starting from `FirstDayOfLastMonthOfQuarter`:
- **Quarter start:** `DATEADD(MONTH, -2, FirstDayOfLastMonthOfQuarter)` — subtracts 2 months to reach the first month of the quarter
- **Quarter end:** `DATEADD(DAY, -1, DATEADD(MONTH, 1, FirstDayOfLastMonthOfQuarter))` — adds 1 month to get the first day of the next quarter, then subtracts 1 day to land on the last day of the current quarter

**How the formula works for Q1 (March 1 as anchor):**

```
FirstDayOfLastMonthOfQuarter = 2024-03-01
DateFirstDayOfQuarter        = 2024-03-01 - 2 months = 2024-01-01
FirstDayNextQuarter          = 2024-03-01 + 1 month  = 2024-04-01
DateLastDayOfQuarter         = 2024-04-01 - 1 day    = 2024-03-31
```

**Final output:**

```
DateFirstDayOfQuarter  DateLastDayOfQuarter
2024-01-01             2024-03-31
2024-04-01             2024-06-30
2024-07-01             2024-09-30
2024-10-01             2024-12-31
(4 rows affected)
```

### Changing the year

To retrieve quarter dates for a different year, simply replace the four `YRQ` values. For example, for 2025:

```sql
SELECT DISTINCT 20251 AS YRQ ...
SELECT DISTINCT 20252 AS YRQ ...
SELECT DISTINCT 20253 AS YRQ ...
SELECT DISTINCT 20254 AS YRQ ...
```

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
