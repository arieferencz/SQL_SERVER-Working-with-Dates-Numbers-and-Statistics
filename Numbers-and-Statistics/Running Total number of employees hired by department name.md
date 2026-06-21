# Running total number of employees hired by department name

## 🎯 Exercise
Calculate a cumulative running total of the number of employees hired per department — ordered by hire year and restarting from 1 for each new department.

---

## 💡 Solution

### Approach
We join three tables to retrieve each employee's department name and hire year. We then use `SUM(1) OVER (PARTITION BY ... ORDER BY ... ROWS UNBOUNDED PRECEDING)` as a window function that acts as a row counter — adding `1` for each employee processed within each department partition, ordered by hire year. This produces a cumulative count that starts at `1` for the first employee in each department and increments with each subsequent hire.

### T-SQL functions and clauses used

| Function / Clause | Purpose |
|---|---|
| `SUM(1)` | Acts as a counter — adds `1` for each row processed |
| `OVER (...)` | Defines the window over which `SUM(1)` is calculated |
| `PARTITION BY Departments.[Name]` | Divides the result into separate partitions per department — the running total restarts at `1` for each new department |
| `ORDER BY YEAR(Employees.[HireDate])` | Defines the order within each partition — employees are counted in chronological order of their hire year |
| `ROWS UNBOUNDED PRECEDING` | Defines the window frame — includes all rows from the first row of the partition up to and including the current row |
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
  , YEAR(Employees.[HireDate]) AS HireYear
  , SUM(1) OVER (
        PARTITION BY Departments.[Name]
        ORDER BY YEAR(Employees.[HireDate])
        ROWS UNBOUNDED PRECEDING) AS RunningTotalHired
FROM [AdventureWorks2022].[HumanResources].[Department] AS Departments
INNER JOIN [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS EmployeesHistorical
    ON Departments.[DepartmentID] = EmployeesHistorical.[DepartmentID]
INNER JOIN [AdventureWorks2022].[HumanResources].[Employee] AS Employees
    ON EmployeesHistorical.[BusinessEntityID] = Employees.[BusinessEntityID]
```

---

### Output (truncated)

```
Name              HireYear  RunningTotalHired
Document Control  2007      1
Document Control  2009      2
Document Control  2009      3
Document Control  2011      4
Document Control  2011      5
Engineering       2007      1
Engineering       2007      2
Engineering       2007      3
Engineering       2008      4
Engineering       2008      5
Engineering       2009      6
Engineering       2009      7
Executive         2009      1
Executive         2011      2
...
Tool Design       2007      1
Tool Design       2008      2
Tool Design       2008      3
Tool Design       2009      4
(296 rows affected)
```

---

## 🔍 Step-by-step explanation

### How the `OVER` clause works

The `SUM(1) OVER (...)` expression is a **window function** — it calculates a value across a defined set of rows (the "window") without collapsing the result into a single row the way a regular `GROUP BY` aggregate would. Every row in the result keeps its own individual row, and the window function adds a new column showing the accumulated value up to that point.

The three arguments inside `OVER (...)` define exactly how the window is built:

---

#### `PARTITION BY Departments.[Name]`
Divides the entire result set into separate independent partitions — one per department name. The window function is applied to each partition separately, and the running total **restarts from 1** at the beginning of each new partition.

Without `PARTITION BY`, all 296 rows would form a single partition — producing a single running total from 1 to 296 across all departments combined.

---

#### `ORDER BY YEAR(Employees.[HireDate])`
Within each department partition, rows are processed in ascending order of hire year. This means employees hired earlier contribute to the running total before those hired later — producing a chronological cumulative count.

---

#### `ROWS UNBOUNDED PRECEDING`
Defines the **window frame** — the subset of rows within the partition that contributes to the calculation for each row.

`ROWS UNBOUNDED PRECEDING` means: *"include all rows from the very first row of this partition up to and including the current row."*

**How the window frame moves row by row — example for Engineering:**

```
Row  HireYear  Window frame included        SUM(1)
1    2007      rows 1–1                     1
2    2007      rows 1–2                     2
3    2007      rows 1–3                     3
4    2008      rows 1–4                     4
5    2008      rows 1–5                     5
6    2009      rows 1–6                     6
7    2009      rows 1–7                     7
```

> **Note:** `ROWS UNBOUNDED PRECEDING` is equivalent to `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` — both produce identical results. The shorter form is simply a convenient alias for the full syntax.

---

### Query 1.1 — Raw data before the window function

Before applying the window function, the three-table join produces one row per department-employee combination with the hire year.

**Output (truncated):** 296 rows — one per department-employee pair (slightly more than 290 unique employees because some employees have records in multiple departments).

```
DepartmentName    HireYear
Document Control  2007
Document Control  2009
Document Control  2009
Document Control  2011
Document Control  2011
Engineering       2007
Engineering       2007
Engineering       2007
Engineering       2008
...
Tool Design       2007
Tool Design       2008
Tool Design       2008
Tool Design       2009
(296 rows affected)
```

---

### Query 1.2 — Apply the window function

Adding `SUM(1) OVER (PARTITION BY Name ORDER BY HireYear ROWS UNBOUNDED PRECEDING)` adds the running total column — without changing the number of rows or collapsing any data.

**Full output for Document Control and Engineering:**

```
Name              HireYear  RunningTotalHired
Document Control  2007      1
Document Control  2009      2
Document Control  2009      3
Document Control  2011      4
Document Control  2011      5            ← 5 employees total in Document Control
Engineering       2007      1            ← restarts at 1 for new department
Engineering       2007      2
Engineering       2007      3
Engineering       2008      4
Engineering       2008      5
Engineering       2009      6
Engineering       2009      7            ← 7 employees total in Engineering
```

The last row of each department shows the department's total employee count — matching the results from the [List all departments and their employee counts](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Working-with-data-sets/List%20all%20departments%20and%20their%20employee%20counts%2C%20including%20departments%20with%20zero%20employees.md) exercise.

---

### Window function vs `GROUP BY` — key difference

| Approach | Rows returned | Running total visible? |
|---|---|---|
| `GROUP BY` + `COUNT()` | One row per department | No — only final total |
| `SUM(1) OVER (...)` | One row per employee | Yes — cumulative count per row |

The window function is the correct choice here because we want to see the **progression** of hires over time — not just the final total per department.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
