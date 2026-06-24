# Counting the number of employees hired every month of every year

## 🎯 Exercise
Count the number of employees hired in each month of each year — including months where no employees were hired (showing `0` instead of omitting the row entirely).

---

## 💡 Solution

### Approach
We generate a complete list of every calendar day between the first day of the earliest hire month and the first day of the latest hire month using `GENERATE_SERIES()`. We then `LEFT JOIN` this list to a subquery that converts each employee's hire date to the first day of their hire month. A `CASE` flag marks matching rows as `1` and non-matching rows as `0`. Filtering to the 1st of each month and summing the flags produces the final monthly count — including months with zero hires.

### T-SQL functions, joins, and case expressions used

| Function | Purpose |
|---|---|
| `MIN(HireDate)` / `MAX(HireDate)` | Retrieves the earliest and latest hire dates |
| `DATEDIFF(MONTH, 0, date)` | Calculates the number of months between `1900-01-01` and the given date |
| `DATEADD(MONTH, n, 0)` | Adds `n` months to `1900-01-01` — landing on the 1st of the target month |
| `GENERATE_SERIES(start, stop, step)` | Generates a sequential list of integers used to create a date range |
| `DATEADD(DAY, value, start_date)` | Converts each integer into a calendar date |
| `DATEADD(DAY, -(DAY(HireDate) - 1), HireDate)` | Calculates the first day of the month for each hire date |
| `DATEPART(DAY, date)` | Returns the day number — used to filter to the 1st of each month only |
| `LEFT JOIN` | Keeps all generated dates, including months with no hires |
| `SUM(CASE WHEN ... THEN 1 ELSE 0 END)` | Counts matching employee rows per month |

### Table used

| Schema | Table |
|---|---|
| `HumanResources` | `Employee` |

---

### T-SQL code — Final solution
```sql
SELECT
    W.GeneratedDates
  , SUM(CASE WHEN Employee.FirstDayOfMonthOfHireDate IS NOT NULL THEN 1 ELSE 0 END) AS CountHireDates
FROM (
    SELECT CAST(DATEADD(DAY, value, (SELECT DATEADD(MONTH, DATEDIFF(MONTH, 0, (SELECT MIN(HireDate) FROM [AdventureWorks2022].[HumanResources].[Employee])), 0))) AS DATE) AS GeneratedDates
    FROM GENERATE_SERIES(0,
                        DATEDIFF(DAY,
                            (SELECT DATEADD(MONTH, DATEDIFF(MONTH, 0, (SELECT MIN(HireDate) FROM [AdventureWorks2022].[HumanResources].[Employee])), 0)),
                            (SELECT DATEADD(MONTH, DATEDIFF(MONTH, 0, (SELECT MAX(HireDate) FROM [AdventureWorks2022].[HumanResources].[Employee])), 0))),
                        1)
    ) AS W
LEFT JOIN (SELECT DATEADD(DAY, -(DAY(HireDate) - 1), HireDate) AS FirstDayOfMonthOfHireDate
            FROM [AdventureWorks2022].[HumanResources].[Employee]) AS Employee
    ON W.GeneratedDates = Employee.FirstDayOfMonthOfHireDate
WHERE DATEPART(DAY, W.GeneratedDates) = 1
GROUP BY W.GeneratedDates
ORDER BY W.GeneratedDates
```

---

### Output (truncated)
```
GeneratedDates  CountHireDates
2006-06-01      1
2006-07-01      0
2006-08-01      0
...
2007-10-01      0
2007-11-01      1
2007-12-01      4
2008-01-01      5
2008-02-01      4
2008-03-01      3
2008-04-01      0
...
2008-12-01      62
2009-01-01      61
2009-02-01      58
2009-03-01      16
...
2009-12-01      12
2010-01-01      15
2010-02-01      12
2010-03-01      6
...
2013-04-01      0
2013-05-01      2
(84 rows affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — Retrieve the first and last hire dates

**T-SQL code**
```sql
SELECT MIN(HireDate) AS FirstHireDate FROM [AdventureWorks2022].[HumanResources].[Employee];
SELECT MAX(HireDate) AS LastHireDate  FROM [AdventureWorks2022].[HumanResources].[Employee];
```

**Output**
```
FirstHireDate   LastHireDate
2006-06-30      2013-05-30
```

---

### Query 1.2 — Generate all calendar dates between the first and last hire months

We convert both boundary dates to the **first day of their respective months** before generating the series — this ensures the generated range starts and ends cleanly on month boundaries.

`DATEADD(MONTH, DATEDIFF(MONTH, 0, MIN(HireDate)), 0)` converts `2006-06-30` → `2006-06-01`.
`DATEADD(MONTH, DATEDIFF(MONTH, 0, MAX(HireDate)), 0)` converts `2013-05-30` → `2013-05-01`.

`GENERATE_SERIES(0, 2527, 1)` generates integers from 0 to 2,527 (the number of days between `2006-06-01` and `2013-05-01`). `DATEADD(DAY, value, '2006-06-01')` converts each integer to a calendar date.

**Output (truncated):** 2,527 rows — every calendar day from `2006-06-01` to `2013-05-01`.

```
GeneratedDates
2006-06-01
2006-06-02
2006-06-03
...
2013-04-30
2013-05-01
(2527 rows affected)
```

---

### Query 1.3 — Calculate the first day of the month for each hire date

**How the formula works — step by step for a sample hire date of `2006-06-30`:**
```
HireDate   DayOfHireDate  DayMinus1  NegativeDayMinus1  FirstDayOfMonth
2006-06-30 30             29         -29                 2006-06-01
2007-01-26 26             25         -25                 2007-01-01
2007-11-11 11             10         -10                 2007-11-01
2007-12-05 5              4          -4                  2007-12-01
```
> `DAY(HireDate)` returns the day number of the hire date. Subtracting `DAY(HireDate) - 1` days moves back to the 1st of that month.


**T-SQL code**
```sql
SELECT DATEADD(DAY, -(DAY(HireDate) - 1), HireDate) AS FirstDayOfMonthOfHireDate
FROM [AdventureWorks2022].[HumanResources].[Employee]
```


**Output:** 290 rows — one first-of-month date per employee.

---

### Query 1.4 — LEFT JOIN generated dates to employee hire months

We `LEFT JOIN` the generated date list (Query 1.2) to the first-of-month hire dates (Query 1.3) on `GeneratedDates = FirstDayOfMonthOfHireDate`. The `LEFT JOIN` preserves all generated dates — months with no hires get `NULL` in the employee column, which the `CASE` statement converts to `0`.

**Output (truncated):** 2,792 rows — the 2,527 generated dates expanded by employees hired in matching months.

```
GeneratedDates  FirstDayOfMonthOfHireDate  CountHireDates
2006-06-01      2006-06-01                 1
2006-06-02      NULL                       0
2006-06-03      NULL                       0
...
2007-12-01      2007-12-01                 1
2007-12-01      2007-12-01                 1
2007-12-01      2007-12-01                 1
2007-12-01      2007-12-01                 1
2007-12-02      NULL                       0
...
(2792 rows affected)
```

---

### Query 1.5 — Filter to the 1st of each month only

Adding `WHERE DATEPART(DAY, W.GeneratedDates) = 1` keeps only rows where the generated date is the 1st of the month — removing all the non-1st dates from the joined result. This gives one row per month per employee hired in that month, plus one row per month with no hires.

**Output (truncated):** 349 rows — one per month-employee combination, showing the flag.

```
GeneratedDates  FirstDayOfMonthOfHireDate  CountHireDates
2006-06-01      2006-06-01                 1
2006-07-01      NULL                       0
2006-08-01      NULL                       0
...
2007-12-01      2007-12-01                 1
2007-12-01      2007-12-01                 1
2007-12-01      2007-12-01                 1
2007-12-01      2007-12-01                 1
...
(349 rows affected)
```

### Final Query — Sum per month and collapse to one row per month

Adding `SUM(CASE WHEN ... THEN 1 ELSE 0 END)` and `GROUP BY W.GeneratedDates` collapses all rows for the same month into a single row with a total count. Months with no hires produce a single row with `CountHireDates = 0` — which is why `LEFT JOIN` was essential: an `INNER JOIN` would have dropped those months entirely.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
