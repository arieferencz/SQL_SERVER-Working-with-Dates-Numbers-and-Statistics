# Creating vertical histograms

## 🎯 Exercise
Create a vertical text-based histogram using asterisks (`*`) to visually represent employee counts by department — with each department as a column and each row representing one employee, building the bars from top to bottom.

---

## 📝 Note

> A **horizontal histogram** (from the previous exercise) places each bar in a single cell — one row per group. A **vertical histogram** places each asterisk on a separate row — stacking them downward so the tallest bar (Production, 179 employees) determines the number of rows. This produces 179 rows, one per Production employee, with shorter departments showing empty cells in rows beyond their count.

---

## 💡 Solution

### Approach
We build the vertical histogram in 4 steps:
1. Remove duplicate employee records caused by department history changes
2. Use `ROW_NUMBER()` partitioned by department to number each employee within their department — this becomes the row position in the histogram
3. Use `CASE` statements to place `'*'` in the correct department column and `NULL` elsewhere
4. Group by `RN` (row number) and use `MAX(CASE ...)` with `COALESCE()` to collapse each row — replacing `NULL` with `''` for a clean display

### T-SQL functions, case expressions, and clauses used

| Function | Purpose |
|---|---|
| `ROW_NUMBER() OVER (PARTITION BY BusinessEntityID ORDER BY ModifiedDate DESC)` | Removes duplicate employee records — keeps the most recent department per employee |
| `ROW_NUMBER() OVER (PARTITION BY DepartmentName ORDER BY BusinessEntityID)` | Numbers each employee within their department — becomes the vertical row position |
| `CASE WHEN DepartmentName = 'X' THEN '*' ELSE NULL END` | Places `'*'` in the correct department column and `NULL` in all others |
| `MAX(CASE ...)` | Collapses multiple rows per `RN` into one — picks up the `'*'` value |
| `COALESCE(MAX(...), '')` | Replaces any remaining `NULL` values with an empty string for clean output |
| `GROUP BY RN` | Groups all rows with the same vertical position into a single output row |

### Tables used

| Schema | Table |
|---|---|
| `HumanResources` | `EmployeeDepartmentHistory` |
| `HumanResources` | `Department` |
| `HumanResources` | `Employee` |

---

### T-SQL code — Full solution

```sql
SELECT
    COALESCE(MAX(Z.dept_DocControl),  '') AS dept_DocControl
  , COALESCE(MAX(Z.dept_Engin),       '') AS dept_Engin
  , COALESCE(MAX(Z.dept_Exec),        '') AS dept_Exec
  , COALESCE(MAX(Z.dept_FacILMaint),  '') AS dept_FacILMaint
  , COALESCE(MAX(Z.dept_Finance),     '') AS dept_Finance
  , COALESCE(MAX(Z.dept_HR),          '') AS dept_HR
  , COALESCE(MAX(Z.dept_IT),          '') AS dept_IT
  , COALESCE(MAX(Z.dept_Marketing),   '') AS dept_Marketing
  , COALESCE(MAX(Z.dept_Prod),        '') AS dept_Prod
  , COALESCE(MAX(Z.dept_ProdControl), '') AS dept_ProdControl
  , COALESCE(MAX(Z.dept_Purch),       '') AS dept_Purch
  , COALESCE(MAX(Z.dept_QA),          '') AS dept_QA
  , COALESCE(MAX(Z.dept_R_and_D),     '') AS dept_R_and_D
  , COALESCE(MAX(Z.dept_Sales),       '') AS dept_Sales
  , COALESCE(MAX(Z.dept_ShipReceiv),  '') AS dept_ShipReceiv
  , COALESCE(MAX(Z.dept_ToolDesign),  '') AS dept_ToolDesign
FROM (
    SELECT
        ROW_NUMBER() OVER (PARTITION BY Y.DepartmentName
                           ORDER BY Y.BusinessEntityID)                         AS RN
      , CASE WHEN Y.DepartmentName = 'Document Control'           THEN '*' ELSE NULL END AS dept_DocControl
      , CASE WHEN Y.DepartmentName = 'Engineering'                THEN '*' ELSE NULL END AS dept_Engin
      , CASE WHEN Y.DepartmentName = 'Executive'                  THEN '*' ELSE NULL END AS dept_Exec
      , CASE WHEN Y.DepartmentName = 'Facilities and Maintenance' THEN '*' ELSE NULL END AS dept_FacILMaint
      , CASE WHEN Y.DepartmentName = 'Finance'                    THEN '*' ELSE NULL END AS dept_Finance
      , CASE WHEN Y.DepartmentName = 'Human Resources'            THEN '*' ELSE NULL END AS dept_HR
      , CASE WHEN Y.DepartmentName = 'Information Services'       THEN '*' ELSE NULL END AS dept_IT
      , CASE WHEN Y.DepartmentName = 'Marketing'                  THEN '*' ELSE NULL END AS dept_Marketing
      , CASE WHEN Y.DepartmentName = 'Production'                 THEN '*' ELSE NULL END AS dept_Prod
      , CASE WHEN Y.DepartmentName = 'Production Control'         THEN '*' ELSE NULL END AS dept_ProdControl
      , CASE WHEN Y.DepartmentName = 'Purchasing'                 THEN '*' ELSE NULL END AS dept_Purch
      , CASE WHEN Y.DepartmentName = 'Quality Assurance'          THEN '*' ELSE NULL END AS dept_QA
      , CASE WHEN Y.DepartmentName = 'Research and Development'   THEN '*' ELSE NULL END AS dept_R_and_D
      , CASE WHEN Y.DepartmentName = 'Sales'                      THEN '*' ELSE NULL END AS dept_Sales
      , CASE WHEN Y.DepartmentName = 'Shipping and Receiving'     THEN '*' ELSE NULL END AS dept_ShipReceiv
      , CASE WHEN Y.DepartmentName = 'Tool Design'                THEN '*' ELSE NULL END AS dept_ToolDesign
    FROM (
        SELECT
            X.BusinessEntityID
          , X.DepartmentName
        FROM (
            SELECT
                EmployeeDeptHistory.BusinessEntityID
              , EmployeeDeptHistory.DepartmentID
              , EmployeeDeptHistory.ModifiedDate
              , Department.[Name] AS DepartmentName
              , ROW_NUMBER() OVER (PARTITION BY EmployeeDeptHistory.BusinessEntityID
                                   ORDER BY EmployeeDeptHistory.BusinessEntityID ASC,
                                            EmployeeDeptHistory.ModifiedDate DESC) AS RowNumber
            FROM [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS EmployeeDeptHistory
            LEFT JOIN [AdventureWorks2022].[HumanResources].[Department] AS Department
                ON EmployeeDeptHistory.DepartmentID = Department.DepartmentID
            INNER JOIN [AdventureWorks2022].[HumanResources].[Employee] AS Employee
                ON EmployeeDeptHistory.BusinessEntityID = Employee.BusinessEntityID
        ) AS X
        WHERE RowNumber = 1
        GROUP BY X.BusinessEntityID, X.DepartmentName
    ) AS Y
) AS Z
GROUP BY Z.RN

Warning: Null value is eliminated by an aggregate or other SET operation.
```

---

### Output (truncated)

```
dept_DocControl  dept_Engin  dept_Exec  dept_FaciLMaint  dept_Finance  dept_HR  dept_IT  dept_Marketing  dept_Prod  dept_ProdControl  dept_Purch  dept_QA  dept_R_and_D  dept_Sales  dept_ShipReceiv  dept_ToolDesign
*                *           *          *                *             *        *        *               *          *                 *           *        *             *           *                *
*                *           *          *                *             *        *        *               *          *                 *           *        *             *           *                *
*                *                      *                *             *        *        *               *          *                 *           *        *             *           *                *
*                *                      *                *             *        *        *               *          *                 *           *        *             *           *                *
*                *                      *                *             *        *        *               *          *                 *           *        *                         *                *
                 *                      *                *             *        *        *               *          *                 *           *                      *            *
                                        *                *                      *        *               *                            *                                 *
                                                         *                      *        *               *                            *                                 *
                                                         *                      *        *               *                            *                                 *
                                                         *                      *                        *                            *                                 *
                                                                                *                        *                            *                                 *
                                                                                *                        *                            *                                 *
                                                                                *                        *                                                              *
                                                                                *                        *                                                              *
                                                                                *                        *                                                              *
                                                                                *                        *                                                              *
                                                                                *                        *                                                              *
                                                                                *                        *                                                              *
                                                                                *
...
(rows continue down to row 179 — Production's last employee)
(179 rows affected)
```

> The warning *"Null value is eliminated by an aggregate or other SET operation"* is expected — `MAX()` aggregates columns that contain `NULL` values (the empty histogram cells). This does not affect the correctness of the result.

---

## 🔍 Step-by-step explanation

### Query 1.1 — Remove duplicate records (`OriginalTablesLevel1` → `RemoveDuplicatesLevel2`)
Same deduplication logic as the horizontal histogram exercise — `ROW_NUMBER()` partitioned by `BusinessEntityID` ordered by `ModifiedDate DESC` keeps only the most recent department per employee. `GROUP BY BusinessEntityID, DepartmentName` ensures one clean row per employee.

**Output:** 290 rows — one per employee with their current department name.

---

### Query 1.2 — Assign a vertical row position per department (`AsterisksNULLsLevel3`)
`ROW_NUMBER() OVER (PARTITION BY DepartmentName ORDER BY BusinessEntityID)` numbers each employee within their department from `1` upward. This row number (`RN`) becomes the vertical position in the histogram — row 1 is the top of all bars, row 179 is the bottom of the Production bar.

Simultaneously, 16 `CASE` statements place `'*'` in the matching department column and `NULL` in all other columns for each employee row.

**Output (truncated):** 290 rows — each employee's asterisk in one column, `NULL` in the other 15.

```
RN		dept_DocControl		dept_Engin		dept_Prod		...
1		*					NULL			NULL			...
2		*					NULL			NULL			...
3		*					NULL			NULL			...
4		*					NULL			NULL			...
5		*					NULL			NULL			...
1		NULL				*				NULL			...
2		NULL				*				NULL			...
...
1		NULL				NULL			*				...
2		NULL				NULL			*				...
...
179		NULL				NULL			*				...
(290 rows affected)
```

---

### Final Query (`VerticalHistogramLevel4`) — Collapse by `RN` using `GROUP BY` and `MAX()`

`GROUP BY Z.RN` groups all rows sharing the same vertical position. `MAX(CASE ...)` picks up the one `'*'` value per column per row number — collapsing 290 rows into 179 rows (one per Production employee, the tallest bar). `COALESCE(..., '')` replaces all `NULL` values with empty strings for clean display.

**How the collapsing works for `RN = 1`:**
```
Before GROUP BY (16 rows at RN=1, one per department):
RN		dept_DocControl		dept_Engin		dept_Exec		...		dept_Prod		...
1		*					NULL			NULL			...		NULL			...
1		NULL				*				NULL			...		NULL			...
1		NULL				NULL			*				...		NULL			...
...
1		NULL				NULL			NULL			...		*				...


After GROUP BY RN=1 with MAX():
dept_DocControl		dept_Engin		dept_Exec		...		dept_Prod		...
*					*				*				...		*				...
```

**Why Production determines the number of rows:**
Production has 179 employees — the highest count of any department. `RN` goes up to 179 for Production, so the result has 179 rows. Departments with fewer employees have `NULL` (displayed as `''`) in rows beyond their employee count — creating the visual effect of shorter bars.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
