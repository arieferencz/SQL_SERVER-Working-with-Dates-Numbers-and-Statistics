# Median calculations using PERCENTILE_DISC() and PERCENTILE_CONT()

## 🎯 Exercise
Calculate the median employee salary rate for every department using both `PERCENTILE_DISC()` and `PERCENTILE_CONT()` — and compare how the two functions differ.

---

## 💡 Solution

### Approach
We join four tables to retrieve each employee's most recent salary rate in their current department. We use `ROW_NUMBER()` to remove duplicates caused by department history changes and multiple pay rate records. We then apply `PERCENTILE_DISC(0.5)` and `PERCENTILE_CONT(0.5)` as window functions partitioned by department to calculate the median salary rate for each department.

### T-SQL functions and clauses used

| Function | Purpose |
|---|---|
| `ROW_NUMBER() OVER (PARTITION BY BusinessEntityID ORDER BY EDH_ModifiedDate DESC, EPH_ModifiedDate DESC)` | Keeps only the most recent department and pay rate record per employee |
| `PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY SalaryRate) OVER (PARTITION BY DepartmentName)` | Returns the median salary rate per department — always an actual value from the data |
| `PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY SalaryRate) OVER (PARTITION BY DepartmentName)` | Returns the median salary rate per department — may be an interpolated value not in the data |
| `SELECT DISTINCT` | Collapses the window function result to one row per department |

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
    Y.DepartmentName
  , PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY Y.SalaryRate)
        OVER (PARTITION BY Y.DepartmentName) AS MedianPercentileDisc
  , PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY Y.SalaryRate)
        OVER (PARTITION BY Y.DepartmentName) AS MedianPercentileCont
FROM (
    SELECT
        X.BusinessEntityID
      , X.DepartmentName
      , X.Rate AS SalaryRate
    FROM (
        SELECT
            EmployeeDepartmentHistory.BusinessEntityID
          , EmployeeDepartmentHistory.DepartmentID
          , EmployeeDepartmentHistory.ModifiedDate AS EDH_ModifiedDate
          , Department.[Name]                      AS DepartmentName
          , EmployeePayHistory.Rate
          , EmployeePayHistory.ModifiedDate        AS EPH_ModifiedDate
          , ROW_NUMBER()
                OVER (PARTITION BY EmployeeDepartmentHistory.BusinessEntityID
                ORDER BY EmployeeDepartmentHistory.BusinessEntityID ASC, EmployeeDepartmentHistory.ModifiedDate DESC, EmployeePayHistory.ModifiedDate DESC)  AS RowNumber
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

### Output
```
DepartmentName					MedianPercentileDisc		MedianPercentileCont
Document Control				16.8269						16.8269
Engineering						32.6923						34.375
Executive						60.0962						92.7981
Facilities and Maintenance		9.25						9.25
Finance							19							19
Human Resources					16.5865						17.42785
Information Services			32.4519						32.4519
Marketing						14.4231						14.4231
Production						12.45						12.45
Production Control				16							16
Purchasing						18.2692						18.2692
Quality Assurance				10.5769						10.5769
Research and Development		40.8654						41.6731
Sales							23.0769						23.0769
Shipping and Receiving			9							9.25
Tool Design						25							26.9231
(16 rows affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — Remove duplicate records (`OriginalTablesLevel1`)
Joining `EmployeeDepartmentHistory`, `Department`, `Employee`, and `EmployeePayHistory` can produce multiple rows per employee — both from department changes and from pay rate changes over time. We use `ROW_NUMBER()` partitioned by `BusinessEntityID` and ordered by both `EDH_ModifiedDate DESC` and `EPH_ModifiedDate DESC` to keep only the most recent department and pay rate record per employee.

**Output:** 290 rows — one per employee with their current department name and most recent salary rate.

---

### Query 1.2 — Apply `PERCENTILE_DISC()` and `PERCENTILE_CONT()`
Both functions use `WITHIN GROUP (ORDER BY SalaryRate)` to sort salary rates within each department, and `OVER (PARTITION BY DepartmentName)` to calculate the result per department. `SELECT DISTINCT` on `DepartmentName` collapses the 290 employee rows into 16 department rows.

---

## 🔍 Key concept — `PERCENTILE_DISC()` vs `PERCENTILE_CONT()`

Both functions calculate the 50th percentile (median) — the value at the midpoint of the sorted salary distribution. They differ in how they handle cases where the median falls between two values:

### `PERCENTILE_DISC(0.5)` — Discrete distribution
Returns the **actual value** from the dataset whose cumulative distribution (`CUME_DIST`) is greater than or equal to `0.5`. The result is always one of the real salary values in the column.

### `PERCENTILE_CONT(0.5)` — Continuous distribution
Returns an **interpolated value** based on a continuous distribution model. If the median falls between two values, it calculates a weighted average of those two values. The result may not exist in the actual data.

### When do they differ?
They produce the **same result** when the department has an **odd** number of employees (the median is a single middle value). They produce **different results** when the department has an **even** number of employees (the median falls between two values):

| DepartmentName | Employees | MedianPercentileDisc | MedianPercentileCont | Differ? |
|---|---|---|---|---|
| Document Control | 5 (odd) | 16.8269 | 16.8269 | No |
| Engineering | 6 (even) | 32.6923 | 34.375 | **Yes** |
| Executive | 1 | 60.0962 | 92.7981 | **Yes** |
| Facilities and Maintenance | 7 (odd) | 9.25 | 9.25 | No |
| Shipping and Receiving | 6 (even) | 9 | 9.25 | **Yes** |

> **Executive department** shows the largest difference (`60.0962` vs `92.7981`) — with only 1 employee counted after deduplication, the interpolation calculation in `PERCENTILE_CONT` behaves differently from `PERCENTILE_DISC`.

### Which to use?
- Use `PERCENTILE_DISC()` when you need a value that **actually exists** in your data (e.g. reporting an actual salary)
- Use `PERCENTILE_CONT()` when you need a **mathematically precise** midpoint (e.g. statistical analysis)

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
