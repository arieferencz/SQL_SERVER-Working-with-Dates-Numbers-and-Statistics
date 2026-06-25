# Creating horizontal histograms

## 🎯 Exercise
Create horizontal text-based histograms using asterisks (`*`) to visually represent employee counts — first by gender, then by department and gender combined.

---

## 💡 Exercise 1 — Horizontal histogram by gender

### Approach
We use `COUNT(*)` to count employees per gender and `REPLICATE('*', COUNT(*))` to generate a string of asterisks whose length equals the employee count — creating a visual bar for each gender.

### T-SQL functions used

| Function | Purpose |
|---|---|
| `COUNT(*)` | Counts the number of employees per group |
| `REPLICATE('*', n)` | Returns the character `'*'` repeated `n` times — creating the histogram bar |

### Table used

| Schema | Table |
|---|---|
| `HumanResources` | `Employee` |

---

### T-SQL code

```sql
SELECT
    GENDER
  , COUNT(*) AS EmployeeCount
  , REPLICATE('*', COUNT(*)) AS EmployeeCountHistogram
FROM [AdventureWorks2022].[HumanResources].[Employee]
GROUP BY GENDER
```

---

### Output

```
GENDER  EmployeeCount  EmployeeCountHistogram
F       84             ************************************************************************************
M       206            **************************************************************************************************************************************************************************************************************
(2 rows affected)
```

---

## 💡 Exercise 2 — Horizontal histogram by department and gender

### Approach
We join three tables to get each employee's current department and gender, using `ROW_NUMBER()` to remove duplicate employee records caused by department history changes. We then group by department name and gender, and use `REPLICATE('*', COUNT(*))` to build the histogram bar for each combination.

### Tables used

| Schema | Table |
|---|---|
| `HumanResources` | `EmployeeDepartmentHistory` |
| `HumanResources` | `Department` |
| `HumanResources` | `Employee` |

---

### Solution 2.1 — Histogram by department and gender

```sql
SELECT
    X.DepartmentName
  , X.Gender
  , REPLICATE('*', COUNT(*)) AS EmployeeCountHistogram
FROM (
    SELECT
        EmployeeDeptHistory.BusinessEntityID
        , EmployeeDeptHistory.DepartmentID
        , EmployeeDeptHistory.ModifiedDate
        , Department.[Name] AS DepartmentName
        , Employee.Gender
        , ROW_NUMBER() OVER (PARTITION BY EmployeeDeptHistory.BusinessEntityID
                        ORDER BY EmployeeDeptHistory.BusinessEntityID ASC, EmployeeDeptHistory.ModifiedDate DESC) AS RowNumber
    FROM [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS EmployeeDeptHistory
    LEFT JOIN [AdventureWorks2022].[HumanResources].[Department] AS Department
        ON EmployeeDeptHistory.DepartmentID = Department.DepartmentID
    INNER JOIN [AdventureWorks2022].[HumanResources].[Employee] AS Employee
        ON EmployeeDeptHistory.BusinessEntityID = Employee.BusinessEntityID
    ) AS X
WHERE RowNumber = 1
GROUP BY X.Gender, X.DepartmentName
```

---

### Output
```
DepartmentName				Gender  EmployeeCountHistogram
Document Control           F       *
Document Control           M       ****
Engineering                F       ***
Engineering                M       ***
Executive                  F       *
Executive                  M       *
Facilities and Maintenance F       **
Facilities and Maintenance M       *****
Finance                    F       *****
Finance                    M       *****
Human Resources            F       **
Human Resources            M       ****
Information Services       F       ****
Information Services       M       ******
Marketing                  F       ****
Marketing                  M       *****
Production                 F       **********************************************
Production                 M       *************************************************************************************************************************************
Production Control         M       ******
Purchasing                 F       ****
Purchasing                 M       ********
Quality Assurance          M       ******
Research and Development   F       **
Research and Development   M       **
Sales                      F       *******
Sales                      M       ***********
Shipping and Receiving     F       **
Shipping and Receiving     M       ****
Tool Design                F       *
Tool Design                M       ***
```

---

## 🔍 Step-by-step explanation

### Why `ROW_NUMBER()` is needed
Joining `EmployeeDepartmentHistory` and `Department` produces duplicate rows for employees who have changed departments — 5 employees appear more than once:

**T-SQL code**
```sql
SELECT
    EmployeeDeptHistory.BusinessEntityID
  , COUNT(*) AS EmployeeCount
  , REPLICATE('*', COUNT(*)) AS EmployeeCountHistogram
FROM [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS EmployeeDeptHistory
LEFT JOIN [AdventureWorks2022].[HumanResources].[Department] AS Department
    ON EmployeeDeptHistory.DepartmentID = Department.DepartmentID
GROUP BY EmployeeDeptHistory.BusinessEntityID
HAVING COUNT(*) > 1
ORDER BY EmployeeDeptHistory.BusinessEntityID
```

**Output**
```
BusinessEntityID  EmployeeCount  EmployeeCountHistogram
4                 2              **
16                2              **
224               2              **
234               2              **
250               3              ***
```

> `ROW_NUMBER()` partitioned by `BusinessEntityID` and ordered by `ModifiedDate DESC` numbers each employee's records, and `WHERE RowNumber = 1` keeps only the most recent department record per employee — removing all 5 duplicates.

---

### How `REPLICATE()` builds the histogram bar
`REPLICATE('*', COUNT(*))` returns the asterisk character repeated as many times as the employee count. For example:
- `REPLICATE('*', 6)` → `'******'`
- `REPLICATE('*', 179)` → a string of 179 asterisks

The longer the bar, the larger the employee count — making it immediately visible which departments and genders have more employees.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
