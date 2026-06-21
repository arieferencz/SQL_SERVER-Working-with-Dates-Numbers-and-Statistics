# Counting the number of days between hiring dates on same column

## 🎯 Exercise
For each employee in department 11, calculate the number of days between their start date and the next employee's start date — based on seniority order.

---

## 💡 Solution

### Approach
We use a CTE to remove duplicate employee records caused by department history changes, then apply the `LEAD()` window function to retrieve the next employee's start date in the same ordered result set. `DATEDIFF(DAY, current, next)` then calculates the gap in days between consecutive start dates.

### T-SQL functions used

| Function | Purpose |
|---|---|
| `ROW_NUMBER() OVER (PARTITION BY ... ORDER BY ...)` | Removes duplicate employee records by numbering them per employee |
| `LEAD(StartDate) OVER (ORDER BY StartDate)` | Returns the start date of the next row in the ordered result set |
| `DATEDIFF(DAY, start, end)` | Calculates the number of calendar days between two dates |

### Tables used

| Schema | Table |
|---|---|
| `HumanResources` | `Employee` |
| `HumanResources` | `EmployeeDepartmentHistory` |
| `HumanResources` | `Department` |

---

### T-SQL code

```sql
WITH
OriginalTablesLevel1 AS
(
    SELECT
        ROW_NUMBER() OVER (PARTITION BY Employee.BusinessEntityID
                           ORDER BY Employee.BusinessEntityID ASC,
                                    EmployeeDepartmentHistory.StartDate DESC) AS RowNumberRemovingDuplicates
      , Employee.BusinessEntityID
      , Department.GroupName
      , GroupNameCode = CASE Department.GroupName
            WHEN 'Executive General and Administration' THEN 1
            WHEN 'Inventory Management'                 THEN 2
            WHEN 'Manufacturing'                        THEN 3
            WHEN 'Quality Assurance'                    THEN 4
            WHEN 'Research and Development'             THEN 5
            WHEN 'Sales and Marketing'                  THEN 6
            ELSE 'Error'
        END
      , EmployeeDepartmentHistory.DepartmentID
      , Department.[Name] AS DeparmentName
      , Employee.JobTitle
      , Employee.OrganizationLevel
      , EmployeeDepartmentHistory.StartDate
    FROM [AdventureWorks2022].[HumanResources].[Employee] AS Employee
    LEFT JOIN [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS EmployeeDepartmentHistory
        ON Employee.BusinessEntityID = EmployeeDepartmentHistory.BusinessEntityID
    LEFT JOIN [AdventureWorks2022].[HumanResources].[Department] AS Department
        ON EmployeeDepartmentHistory.DepartmentID = Department.DepartmentID
    WHERE Employee.BusinessEntityID <> 234
),
RemovingDuplicatesLevel2 AS
(
    SELECT
        OriginalTablesLevel1.BusinessEntityID
      , OriginalTablesLevel1.GroupNameCode
      , OriginalTablesLevel1.GroupName
      , OriginalTablesLevel1.DepartmentID
      , OriginalTablesLevel1.DeparmentName
      , OriginalTablesLevel1.JobTitle
      , OriginalTablesLevel1.OrganizationLevel
      , OriginalTablesLevel1.StartDate
    FROM OriginalTablesLevel1
    WHERE OriginalTablesLevel1.RowNumberRemovingDuplicates = 1
)
SELECT
    RemovingDuplicatesLevel2.BusinessEntityID
  , RemovingDuplicatesLevel2.DepartmentID
  , RemovingDuplicatesLevel2.StartDate
  , LEAD(RemovingDuplicatesLevel2.StartDate)
        OVER (ORDER BY RemovingDuplicatesLevel2.StartDate) AS NextHireDate
  , DATEDIFF(DAY,
        RemovingDuplicatesLevel2.StartDate,
        LEAD(RemovingDuplicatesLevel2.StartDate)
            OVER (ORDER BY RemovingDuplicatesLevel2.StartDate)) AS DateDiffHireDates
FROM RemovingDuplicatesLevel2
WHERE RemovingDuplicatesLevel2.DepartmentID = 11
ORDER BY RemovingDuplicatesLevel2.OrganizationLevel, RemovingDuplicatesLevel2.StartDate
```

---

### Output

```
BusinessEntityID  DepartmentID  StartDate    NextHireDate  DateDiffHireDates
263               11            12/11/2008   12/23/2008    12
272               11            12/23/2008   1/11/2009     19
269               11            1/11/2009    1/17/2009     6
270               11            1/17/2009    1/22/2009     5
271               11            1/22/2009    2/3/2009      12
268               11            2/3/2009     2/4/2009      1
264               11            2/4/2009     2/16/2009     12
267               11            2/16/2009    2/23/2009     7
265               11            12/4/2008    12/11/2008    7
266               11            2/23/2009    NULL          NULL
(10 rows affected)
```

---

## 🔍 Step-by-step explanation

### CTE 1 — `OriginalTablesLevel1`
We join the three tables to retrieve each employee's department and start date. A `ROW_NUMBER()` is added, partitioned by `BusinessEntityID` and ordered by `StartDate DESC`, to prepare for deduplication. `BusinessEntityID = 234` is excluded as it has no valid department record.

### CTE 2 — `RemovingDuplicatesLevel2`
We filter `RowNumberRemovingDuplicates = 1` to keep only the most recent department record per employee — removing duplicates created by employees who changed departments over time.

### Final SELECT — Apply `LEAD()` and `DATEDIFF()`
We filter to `DepartmentID = 11` to focus on that department's employees only. `LEAD(StartDate) OVER (ORDER BY StartDate)` retrieves the start date of the **next** employee in the ordered result set. `DATEDIFF(DAY, StartDate, NextHireDate)` then calculates the number of days between the two consecutive start dates.

**Key observations in the output:**

- The last employee (`BusinessEntityID = 266`, start date `2/23/2009`) has `NULL` for both `NextHireDate` and `DateDiffHireDates` — because there is no next row after the last employee, `LEAD()` returns `NULL`.
- `LEAD()` looks **forward** in the ordered result set — it returns the value from the next row. Its counterpart `LAG()` looks backward and returns the value from the previous row.

### `LEAD()` vs `LAG()`

| Function | Direction | Returns |
|---|---|---|
| `LEAD(column) OVER (ORDER BY ...)` | Forward | Value from the **next** row |
| `LAG(column) OVER (ORDER BY ...)` | Backward | Value from the **previous** row |

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
