# Running total number of employees hired by department name and by year

## 🎯 Exercise
Calculate a cumulative running total of the number of employees hired per department **and** per year — restarting from 1 for each new department-year combination.

---

## 📝 Note

> This exercise builds directly on [Running total number of employees hired by department name](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/Numbers-and-Statistics/Running%20total%20number%20of%20employees%20hired%20by%20department%20name.md). The only difference is that `YEAR(HireDate)` is added to `PARTITION BY` — creating a finer partition that restarts the running total for every new department-year combination instead of just every new department.

---

## 💡 Solution

### Approach
We join three tables to retrieve each employee's department name and hire year. We then use `SUM(1) OVER (PARTITION BY DepartmentName, HireYear ORDER BY HireYear ROWS UNBOUNDED PRECEDING)` as a window function. Adding `YEAR(HireDate)` to the `PARTITION BY` clause creates a separate partition for each unique combination of department name and hire year — so the running total restarts at `1` at the beginning of each new year within each department.

### T-SQL functions and clauses used

| Function / Clause | Purpose |
|---|---|
| `SUM(1)` | Acts as a counter — adds `1` for each row processed |
| `OVER (...)` | Defines the window over which `SUM(1)` is calculated |
| `PARTITION BY Departments.[Name], YEAR(Employees.[HireDate])` | Creates one partition per department-year combination — the running total restarts at `1` for each new department-year pair |
| `ORDER BY YEAR(Employees.[HireDate])` | Defines the order within each partition — employees are counted in chronological order of their hire year |
| `ROWS UNBOUNDED PRECEDING` | Includes all rows from the first row of the partition up to and including the current row |
| `YEAR(HireDate)` | Extracts the hire year from the `HireDate` column |
| `INNER JOIN` (Department → EmployeeDepartmentHistory) | Connects departments to their employee history records |
| `INNER JOIN` (EmployeeDepartmentHistory → Employee) | Connects employee history records to hire date information |

### Tables used

| Schema | Table |
|---|---|
| `HumanResources` | `Department` |
| `HumanResources` | `EmployeeDepartmentHistory` |
| `HumanResources` | `Employee` |

---

### T-SQL code

```sql
USE AdventureWorks2022;
GO

SELECT
    Departments.[Name]
  , YEAR(Employees.[HireDate])                                        AS YearHired
  , SUM(1) OVER (
        PARTITION BY Departments.[Name], YEAR(Employees.[HireDate])
        ORDER BY YEAR(Employees.[HireDate])
        ROWS UNBOUNDED PRECEDING)                                     AS RunningTotalHired
FROM [AdventureWorks2022].[HumanResources].[Department] AS Departments
INNER JOIN [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS EmployeesHistorical
    ON Departments.[DepartmentID] = EmployeesHistorical.[DepartmentID]
INNER JOIN [AdventureWorks2022].[HumanResources].[Employee] AS Employees
    ON EmployeesHistorical.[BusinessEntityID] = Employees.[BusinessEntityID]
```

---

### Output (truncated)

```
Name              YearHired  RunningTotalHired
Document Control  2008       1
Document Control  2009       1
Document Control  2009       2
Document Control  2009       3
Document Control  2009       4
Engineering       2007       1
Engineering       2007       2
Engineering       2008       1
Engineering       2008       2
Engineering       2008       3
Engineering       2010       1
Engineering       2011       1
...
Tool Design       2007       1
Tool Design       2007       2
Tool Design       2010       1
Tool Design       2010       2
(296 rows affected)
```

---

## 🔍 Step-by-step explanation

### The key difference from the previous exercise

In the [previous exercise](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/Numbers-and-Statistics/Running%20total%20number%20of%20employees%20hired%20by%20department%20name.md), `PARTITION BY` used only `Departments.[Name]` — creating one partition per department. The running total grew continuously from the first hire to the last within each department.

In this exercise, `PARTITION BY` uses **both** `Departments.[Name]` and `YEAR(Employees.[HireDate])` — creating one partition per **department-year combination**. The running total now restarts at `1` every time the year changes within a department.

**Side-by-side comparison for Engineering:**

```
Previous exercise                      This exercise
(PARTITION BY Name only)               (PARTITION BY Name + Year)

Name         YearHired  Running        Name         YearHired  Running
Engineering  2007       1              Engineering  2007       1
Engineering  2007       2              Engineering  2007       2    ← resets next year
Engineering  2007       3              Engineering  2008       1    ← restarts at 1
Engineering  2008       4              Engineering  2008       2
Engineering  2008       5              Engineering  2008       3    ← resets next year
Engineering  2009       6              Engineering  2010       1    ← restarts at 1
Engineering  2009       7              Engineering  2011       1    ← restarts at 1
```

---

### How `PARTITION BY` with two columns creates finer partitions

Each unique **combination** of department name and hire year becomes its own independent partition. For example, Engineering hired employees across 4 different years — creating 4 separate partitions:

| Partition | Department | Year | Employees in partition |
|---|---|---|---|
| 1 | Engineering | 2007 | 3 |
| 2 | Engineering | 2008 | 3 |
| 3 | Engineering | 2010 | 1 |
| 4 | Engineering | 2011 | 1 |

The running total within each partition goes from `1` up to the number of employees hired in that department-year — independently of all other partitions.

---

### Query 1.1 — Raw data before the window function

The three-table join produces one row per department-employee combination. This is identical to the previous exercise.

**Output (truncated):** 296 rows.

```
DepartmentName    YearHired
Document Control  2008
Document Control  2009
Document Control  2009
Document Control  2009
Document Control  2009
Engineering       2007
Engineering       2007
Engineering       2007
Engineering       2008
Engineering       2008
Engineering       2008
Engineering       2010
Engineering       2011
...
(296 rows affected)
```

---

### Query 1.2 — Apply the window function with two `PARTITION BY` columns

Adding `SUM(1) OVER (PARTITION BY Name, Year ORDER BY Year ROWS UNBOUNDED PRECEDING)` adds the running total column — restarting at `1` for every new department-year pair.

**Full output for Document Control and Engineering:**

```
Name              YearHired  RunningTotalHired
Document Control  2008       1                ← only 1 hire in 2008
Document Control  2009       1                ← restarts at 1 for 2009
Document Control  2009       2
Document Control  2009       3
Document Control  2009       4                ← 4 hires in 2009
Engineering       2007       1                ← restarts at 1 for Engineering 2007
Engineering       2007       2
Engineering       2007       3                ← 3 hires in 2007
Engineering       2008       1                ← restarts at 1 for Engineering 2008
Engineering       2008       2
Engineering       2008       3                ← 3 hires in 2008
Engineering       2010       1                ← restarts at 1 for Engineering 2010
Engineering       2011       1                ← restarts at 1 for Engineering 2011
```

The last row of each department-year partition shows the total hires for that department in that year.

---

### Comparing the two running total exercises

| Feature | By department only | By department and year |
|---|---|---|
| `PARTITION BY` | `Name` | `Name, YEAR(HireDate)` |
| Running total resets | Once per department | Once per department-year combination |
| Last row value per partition | Total employees in the department | Total employees hired in that department-year |
| Use when | Tracking cumulative headcount growth per department | Tracking how many employees were hired each year within each department |

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
