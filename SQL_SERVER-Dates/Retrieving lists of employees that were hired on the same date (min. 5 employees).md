# Retrieving the lists of employees that were hired on the same date (min. 5 employees)

## 🎯 Exercise
Retrieve the list of employees who were hired on the same date as at least 4 other employees — meaning groups of 5 or more employees sharing the same hire date.

---

## 💡 Solution

### Approach
We use a **self-join via Cartesian Product** to pair each employee (table `A`) with every other employee (table `B`) who shares the same hire date. We use `DATENAME()` comparisons across year, month, week, and weekday to match hire dates. The condition `A.BusinessEntityID <= B.BusinessEntityID` prevents duplicate pairs. We then group by employee and use `HAVING COUNT(*) >= 5` to keep only employees who appear in a group of 5 or more.

### T-SQL functions used

| Function | Purpose |
|---|---|
| `DATENAME(YEAR, date)` | Returns the year of a date as a string — used to match hire year |
| `DATENAME(MONTH, date)` | Returns the month name — used to match hire month |
| `DATENAME(WEEK, date)` | Returns the week number — used to match hire week |
| `DATENAME(WEEKDAY, date)` | Returns the weekday name — used to match hire weekday |
| `COUNT(EmpIDA)` | Counts how many employees share the same hire date as each employee |
| `HAVING COUNT(*) >= 5` | Filters to keep only employees in groups of 5 or more |
| `SELECT DISTINCT` | Removes duplicate employee pairs from the Cartesian Product |

### Table used

| Schema | Table |
|---|---|
| `HumanResources` | `Employee` |

---

### T-SQL code — Final query

```sql
SELECT
    X.EmpIDA
  , X.EmpJobTitleA
  , COUNT(X.EmpIDA) AS NumberEmployeesHiredSameDay
FROM (
    SELECT DISTINCT
        A.BusinessEntityID  AS EmpIDA
      , A.JobTitle           AS EmpJobTitleA
      , A.HireDate           AS EmpHireDateA
      , B.BusinessEntityID  AS EmpIDB
      , B.JobTitle           AS EmpJobTitleB
      , B.HireDate           AS EmpHireDateB
    FROM [AdventureWorks2022].[HumanResources].[Employee] AS A
       , [AdventureWorks2022].[HumanResources].[Employee] AS B
    WHERE DATENAME(YEAR,    A.HireDate) = DATENAME(YEAR,    B.HireDate)
      AND DATENAME(MONTH,   A.HireDate) = DATENAME(MONTH,   B.HireDate)
      AND DATENAME(WEEK,    A.HireDate) = DATENAME(WEEK,    B.HireDate)
      AND DATENAME(WEEKDAY, A.HireDate) = DATENAME(WEEKDAY, B.HireDate)
      AND A.BusinessEntityID <= B.BusinessEntityID
) AS X
GROUP BY X.EmpIDA, X.EmpJobTitleA
HAVING COUNT(X.EmpIDA) >= 5
```

---

### Output

```
EmpIDA  EmpJobTitleA                      NumberEmployeesHiredSameDay
7       Research and Development Manager  5
45      Production Technician - WC60      5
109     Production Technician - WC50      5
275     Sales Representative              9
276     Sales Representative              8
277     Sales Representative              7
278     Sales Representative              6
279     Sales Representative              5
(8 rows affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — Pair each employee with all employees sharing the same hire date

We perform a Cartesian Product between the `Employee` table aliased as `A` and the same table aliased as `B`. The `WHERE` clause matches rows where all four date parts are equal — effectively finding all employees who were hired on the exact same date. `A.BusinessEntityID <= B.BusinessEntityID` prevents counting the same pair twice (e.g. pairing employee 275 with 276 and again 276 with 275) by only keeping pairs where A's ID is less than or equal to B's ID. `SELECT DISTINCT` removes any remaining duplicates.

**Output (truncated):** 500 rows — all valid employee pairs sharing a hire date.

```
EmpIDA  EmpJobTitleA                   EmpHireDateA  EmpIDB  EmpJobTitleB                   EmpHireDateB
1       Chief Executive Officer        2009-01-14    1       Chief Executive Officer        2009-01-14
1       Chief Executive Officer        2009-01-14    134     Production Supervisor - WC20   2009-01-14
1       Chief Executive Officer        2009-01-14    148     Production Technician - WC30   2009-01-14
2       Vice President of Engineering  2008-01-31    2       Vice President of Engineering  2008-01-31
3       Engineering Manager            2007-11-11    3       Engineering Manager            2007-11-11
...
275     Sales Representative           2011-05-31    283     Sales Representative           2011-05-31
286     Sales Representative           2013-05-30    288     Sales Representative           2013-05-30
289     Sales Representative           2012-05-30    290     Sales Representative           2012-05-30
(500 rows affected)
```

> **Note:** Every employee always pairs with themselves (`A.BusinessEntityID = B.BusinessEntityID`) — so an employee hired alone on a date still appears with `NumberEmployeesHiredSameDay = 1`.

---

### Query 1.2 — Count employees per hire date group

Adding `GROUP BY X.EmpIDA, X.EmpJobTitleA` and `COUNT(X.EmpIDA)` gives the total number of employees sharing the hire date for each employee — including employees hired alone (count = 1).

**Output (truncated):** 290 rows — one per employee with their group size.

```
EmpIDA  EmpJobTitleA                      NumberEmployeesHiredSameDay
1       Chief Executive Officer           3
2       Vice President of Engineering     1
3       Engineering Manager               1
...
22      Marketing Specialist              4
23      Marketing Specialist              4
...
275     Sales Representative              9
276     Sales Representative              8
277     Sales Representative              7
278     Sales Representative              6
279     Sales Representative              5
280     Sales Representative              4
281     Sales Representative              3
282     Sales Representative              2
283     Sales Representative              1
...
(290 rows affected)
```

> **Why does the count decrease from 9 down to 1 for Sales Representatives?** All 9 Sales Representatives hired on 2011-05-31 are `BusinessEntityID` 275–283. Because of the `A.BusinessEntityID <= B.BusinessEntityID` condition, employee 275 can pair with all 9 (including themselves), employee 276 can pair with 8 (276–283), employee 277 with 7, and so on down to 283 who only pairs with themselves.

### Final Query (Query 1) — Filter to groups of 5 or more using `HAVING`

Adding `HAVING COUNT(X.EmpIDA) >= 5` keeps only the 8 employees who belong to a hire-date group of 5 or more.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
