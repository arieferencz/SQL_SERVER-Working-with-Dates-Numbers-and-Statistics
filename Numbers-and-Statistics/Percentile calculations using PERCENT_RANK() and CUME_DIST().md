# Percentile calculations using PERCENT_RANK() and CUME_DIST()

## 🎯 Exercise
Calculate the salary percentile rank for each employee — both across the entire company and within specific departments — using `PERCENT_RANK()` and `CUME_DIST()`.

---

## 💡 Exercise 1 — Salary percentiles across the entire company

### Approach
We join four tables to retrieve each employee's most recent salary rate, using `ROW_NUMBER()` to remove duplicates. We then apply `PERCENT_RANK()` and `CUME_DIST()` as window functions ordered by salary rate to calculate each employee's relative position within the company's salary distribution.

### T-SQL functions used

| Function | Purpose |
|---|---|
| `ROW_NUMBER() OVER (PARTITION BY BusinessEntityID ORDER BY ModifiedDate DESC)` | Keeps only the most recent department and pay rate record per employee |
| `PERCENT_RANK() OVER (ORDER BY SalaryRate)` | Returns the relative rank of each salary as a value between 0 and 1 |
| `CUME_DIST() OVER (ORDER BY SalaryRate)` | Returns the cumulative distribution — the proportion of salaries ≤ this value |
| `ROUND(value, 2)` | Rounds to 2 decimal places |
| `SELECT DISTINCT` | Collapses duplicate rows — one row per unique salary rate |

### Tables used

| Schema | Table |
|---|---|
| `HumanResources` | `EmployeeDepartmentHistory` |
| `HumanResources` | `Department` |
| `HumanResources` | `Employee` |
| `HumanResources` | `EmployeePayHistory` |

---

### T-SQL code

```sql
SELECT DISTINCT
    Y.SalaryRate
  , ROUND(PERCENT_RANK() OVER (ORDER BY Y.SalaryRate), 2) AS SalaryRatePercentRank
  , ROUND(CUME_DIST()    OVER (ORDER BY Y.SalaryRate), 2) AS SalaryRateCummulativeDistribution
FROM (
    SELECT
        X.BusinessEntityID
      , X.Rate AS SalaryRate
    FROM (
        SELECT
            EmployeeDepartmentHistory.BusinessEntityID
          , EmployeeDepartmentHistory.DepartmentID
          , EmployeeDepartmentHistory.ModifiedDate  AS EDH_ModifiedDate
          , Department.[Name]                       AS DepartmentName
          , EmployeePayHistory.Rate
          , EmployeePayHistory.ModifiedDate         AS EPH_ModifiedDate
          , ROW_NUMBER() OVER (
                PARTITION BY EmployeeDepartmentHistory.BusinessEntityID
                ORDER BY EmployeeDepartmentHistory.BusinessEntityID ASC,
                         EmployeeDepartmentHistory.ModifiedDate DESC,
                         EmployeePayHistory.ModifiedDate DESC)  AS RowNumber
        FROM [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS EmployeeDepartmentHistory
        LEFT JOIN [AdventureWorks2022].[HumanResources].[Department] AS Department
            ON EmployeeDepartmentHistory.DepartmentID = Department.DepartmentID
        INNER JOIN [AdventureWorks2022].[HumanResources].[Employee] AS Employee
            ON EmployeeDepartmentHistory.BusinessEntityID = Employee.BusinessEntityID
        INNER JOIN [AdventureWorks2022].[HumanResources].[EmployeePayHistory] AS EmployeePayHistory
            ON EmployeeDepartmentHistory.BusinessEntityID = EmployeePayHistory.BusinessEntityID
    ) AS X
    WHERE RowNumber = 1
) AS Y
```

---

### Output (truncated)

```
SalaryRate  SalaryRatePercentRank  SalaryRateCummulativeDistribution
9           0                      0.01
9.25        0.01                   0.02
9.5         0.02                   0.12
9.75        0.12                   0.12
10          0.12                   0.17
10.25       0.17                   0.18
...
13.45       0.38                   0.44
13.4615     0.44                   0.45
13.9423     0.45                   0.46
14          0.46                   0.53
14.4231     0.54                   0.55
15          0.55                   0.64
16          0.64                   0.66
16.5865     0.66                   0.66
...
24.0385     0.79                   0.79
24.5192     0.79                   0.79
25          0.8                    0.87
26.4423     0.88                   0.88
27.1394     0.88                   0.88
...
50.4808     0.98                   0.98
60.0962     0.99                   0.99
63.4615     0.99                   0.99
72.1154     0.99                   0.99
84.1346     1                      1
125.5       1                      1
(53 rows affected)
```

---

## 🔍 Key concept — `PERCENT_RANK()` vs `CUME_DIST()`

Both functions return a value between `0` and `1` representing a salary's position within the distribution. They differ in what exactly that position measures:

### `PERCENT_RANK()` — Relative rank
Calculates the **relative rank** of a salary value within the sorted distribution. It answers: *"What percentage of salaries are strictly lower than this one?"*

**Formula:** `(Rank - 1) / (Total rows - 1)`

- The **lowest** salary always returns `0` (no salaries are below it)
- The **highest** salary always returns `1` (all other salaries are below it)
- Ties share the same `PERCENT_RANK` value

**Example from output:**
- `SalaryRate = 9` → `PERCENT_RANK = 0` → no salary is lower
- `SalaryRate = 125.5` → `PERCENT_RANK = 1` → all other salaries are lower

---

### `CUME_DIST()` — Cumulative distribution
Calculates the **cumulative distribution** of a salary — the proportion of all salaries that are **less than or equal** to this value.

**Formula:** `Number of rows with value ≤ current value / Total rows`

- The **lowest** salary returns a small positive value (not `0`) — because at least one salary (itself) is ≤ it
- The **highest** salary always returns `1`
- Ties share the same `CUME_DIST` value

**Example from output:**
- `SalaryRate = 9` → `CUME_DIST = 0.01` → 1% of salaries are ≤ $9
- `SalaryRate = 14` → `CUME_DIST = 0.53` → 53% of salaries are ≤ $14 (roughly the median)
- `SalaryRate = 125.5` → `CUME_DIST = 1` → 100% of salaries are ≤ $125.50

---

### Key differences at a glance

| Feature | `PERCENT_RANK()` | `CUME_DIST()` |
|---|---|---|
| Lowest value | Always `0` | Small positive value (e.g. `0.01`) |
| Highest value | Always `1` | Always `1` |
| Measures | % of rows **strictly below** | % of rows **≤ current value** |
| Formula | `(Rank − 1) / (N − 1)` | `Rows ≤ value / N` |
| Use when | Comparing relative standing | Finding what % earns at most X |

---

## 💡 Exercise 2 — Salary percentiles by department (Sales and Production)

### Approach
We add `PARTITION BY DepartmentName` to both window functions so that percentiles are calculated **within each department** rather than across the whole company. A `WHERE` clause filters to departments `'Sales'` and `'Production'` only.

---

### T-SQL code

```sql
SELECT
    Y.DepartmentName
  , Y.BusinessEntityID
  , Y.SalaryRate
  , ROUND(PERCENT_RANK() OVER (PARTITION BY Y.DepartmentName ORDER BY Y.SalaryRate), 2)
        AS SalaryRatePercentRank
  , ROUND(CUME_DIST()    OVER (PARTITION BY Y.DepartmentName ORDER BY Y.SalaryRate), 2)
        AS SalaryRateCummulativeDistribution
FROM (
    SELECT
        X.DepartmentName
      , X.BusinessEntityID
      , X.Rate AS SalaryRate
    FROM (
        SELECT
            EmployeeDepartmentHistory.BusinessEntityID
          , EmployeeDepartmentHistory.DepartmentID
          , EmployeeDepartmentHistory.ModifiedDate  AS EDH_ModifiedDate
          , Department.[Name]                       AS DepartmentName
          , EmployeePayHistory.Rate
          , EmployeePayHistory.ModifiedDate         AS EPH_ModifiedDate
          , ROW_NUMBER() OVER (
                PARTITION BY EmployeeDepartmentHistory.BusinessEntityID
                ORDER BY EmployeeDepartmentHistory.BusinessEntityID ASC,
                         EmployeeDepartmentHistory.ModifiedDate DESC,
                         EmployeePayHistory.ModifiedDate DESC)  AS RowNumber
        FROM [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS EmployeeDepartmentHistory
        LEFT JOIN [AdventureWorks2022].[HumanResources].[Department] AS Department
            ON EmployeeDepartmentHistory.DepartmentID = Department.DepartmentID
        INNER JOIN [AdventureWorks2022].[HumanResources].[Employee] AS Employee
            ON EmployeeDepartmentHistory.BusinessEntityID = Employee.BusinessEntityID
        INNER JOIN [AdventureWorks2022].[HumanResources].[EmployeePayHistory] AS EmployeePayHistory
            ON EmployeeDepartmentHistory.BusinessEntityID = EmployeePayHistory.BusinessEntityID
    ) AS X
    WHERE RowNumber = 1
) AS Y
WHERE Y.DepartmentName IN ('Sales', 'Production')
ORDER BY Y.DepartmentName, Y.SalaryRate DESC
```

---

### Output (truncated)

```
DepartmentName  BusinessEntityID  SalaryRate  SalaryRatePercentRank  SalaryRateCummulativeDistribution
Sales           263               50.4808     1                      1
Sales           264               39.6635     0.89                   0.9
Sales           270               38.4615     0.67                   0.8
Sales           271               38.4615     0.67                   0.8
Sales           265               32.4519     0.44                   0.6
Sales           266               32.4519     0.44                   0.6
Sales           267               27.4038     0                      0.4
Sales           268               27.4038     0                      0.4
Sales           269               27.4038     0                      0.4
Sales           272               27.4038     0                      0.4
Production      25                84.1346     1                      1
Production      87                25          0.88                   0.99
Production      93                25          0.88                   0.99
Production      78                25          0.88                   0.99
...
Production      85                15          0.74                   0.88
Production      86                15          0.74                   0.88
Production      135               14          0.61                   0.73
Production      136               14          0.61                   0.73
...
Production      48                13.45       0.52                   0.61
...
Production      41                12.45       0.37                   0.51
Production      42                12.45       0.37                   0.51
...
Production      75                9.5         0                      0.14
Production      76                9.5         0                      0.14
Production      77                9.5         0                      0.14
(189 rows affected)
```

---

### Key observations from Exercise 2

**Sales department (10 employees):**
- `BusinessEntityID = 263` earns the highest salary (`$50.48/hr`) — `PERCENT_RANK = 1`, `CUME_DIST = 1`
- Employees 267–272 all earn `$27.40/hr` — they share `PERCENT_RANK = 0` (no one earns less within this department) and `CUME_DIST = 0.4` (40% of employees earn ≤ $27.40)

**Production department (179 employees):**
- `BusinessEntityID = 25` earns the highest salary (`$84.13/hr`) — significantly above all other Production employees
- The majority of Production employees earn between `$9.50` and `$25.00/hr`
- Employees earning `$9.50/hr` have `PERCENT_RANK = 0` — the lowest earners in the department

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
