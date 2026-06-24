# Counting the number of business days between 2 dates

## 🎯 Exercise
Count the number of business days (Monday to Friday, excluding weekends) between the birth date of the oldest employee and the birth date of the youngest employee.

---

## 📝 Note

> **Oldest employee birth date:** `1951-10-17`
> **Youngest employee birth date:** `1991-05-31`
> **Total calendar days between them:** 14,472
> **Total business days between them:** 10,338

---

## 💡 Solution 1 — Using `GENERATE_SERIES()`

### Approach
We use `GENERATE_SERIES()` to generate a sequential list of integers from `0` to the total number of days between the two dates. We then use `DATEADD()` to convert each integer into an actual date, `DATENAME()` to retrieve the weekday name for each date, and a `WHERE` clause to exclude Saturdays and Sundays. Finally, `COUNT(*)` counts the remaining business days.

### T-SQL functions used

| Function | Purpose |
|---|---|
| `MIN(BirthDate)` / `MAX(BirthDate)` | Retrieves the oldest and youngest employee birth dates |
| `DATEDIFF(DAY, start, end)` | Calculates the total number of calendar days between the two dates |
| `GENERATE_SERIES(start, stop, step)` | Generates a sequential list of integers from `start` to `stop` |
| `DATEADD(DAY, value, start_date)` | Converts each integer into a date by adding it to the start date |
| `DATENAME(weekday, date)` | Returns the weekday name (e.g. `'Monday'`, `'Saturday'`) for a given date |
| `COUNT(*)` | Counts the total number of rows remaining after filtering |

### Table used

| Schema | Table |
|---|---|
| `HumanResources` | `Employee` |

---

### T-SQL code — Full solution

```sql
SELECT FORMAT(COUNT(*), '#,#') AS CountBusinessdays
FROM (
    SELECT
        X.GeneratedDates
      , DATENAME(weekday, X.GeneratedDates) AS GeneratedWeekDays
    FROM (
        SELECT
            CAST(DATEADD(DAY, value,
                (SELECT MIN(BirthDate) FROM [AdventureWorks2022].[HumanResources].[Employee])
            ) AS DATE) AS GeneratedDates
        FROM GENERATE_SERIES(
            0,
            DATEDIFF(
				DAY,
                (SELECT MIN(BirthDate) FROM [AdventureWorks2022].[HumanResources].[Employee]),
                (SELECT MAX(BirthDate) FROM [AdventureWorks2022].[HumanResources].[Employee])
				),
            1)
    	) AS X
	WHERE DATENAME(weekday, X.GeneratedDates) NOT IN ('Saturday', 'Sunday')
	) AS Y
```

---

### Output

```
CountBusinessdays
10,338
(1 row affected)
```

---

## 🔍 Step-by-step explanation — Solution 1

### Query 1.1 — Retrieve the oldest and youngest birth dates

**T-SQL code**
```sql
SELECT MIN(BirthDate) AS OldestEmployee FROM [AdventureWorks2022].[HumanResources].[Employee];
SELECT MAX(BirthDate) AS YoungestEmployee FROM [AdventureWorks2022].[HumanResources].[Employee];
```

**Output**
```
OldestEmployee    YoungestEmployee
1951-10-17        1991-05-31
```

---

### Query 1.2 — Generate all dates between the two birth dates

`GENERATE_SERIES(0, 14471, 1)` generates integers from `0` to `14471` (the total number of days between the two dates). `DATEADD(DAY, value, '1951-10-17')` converts each integer into a date starting from the oldest birth date.

**T-SQL code**
```sql
SELECT
    CAST(DATEADD(DAY, value, (SELECT MIN(BirthDate) FROM [AdventureWorks2022].[HumanResources].[Employee])) AS DATE) AS GeneratedDates
FROM GENERATE_SERIES(
    0,
    DATEDIFF(
		DAY,
        (SELECT MIN(BirthDate) FROM [AdventureWorks2022].[HumanResources].[Employee]),
        (SELECT MAX(BirthDate) FROM [AdventureWorks2022].[HumanResources].[Employee])
    	),
    1)
```

**Output (truncated):**

```
GeneratedDates
1951-10-17
1951-10-18
1951-10-19
...
1991-05-29
1991-05-30
1991-05-31
(14472 rows affected)
```

> You can also use hardcoded dates for a quick verification:
> ```sql
> SELECT CAST(DATEADD(DAY, value, '1951-10-17') AS DATE) AS GeneratedDates
> FROM GENERATE_SERIES(0, DATEDIFF(DAY, '1951-10-17', '1991-05-31'), 1);
> ```

---

### Query 1.3 — Add weekday names using `DATENAME()`

**T-SQL code**
```sql
SELECT									-- GenerateWeekDaysLevel2
	X.GeneratedDates
	, DATENAME(weekday, X.GeneratedDates) AS GeneratedWeekDays				
FROM
	(
	SELECT								-- GenerateDatesLevel1
	CAST(DATEADD(DAY, value, (SELECT MIN(BirthDate) FROM [AdventureWorks2022].[HumanResources].[Employee])) AS DATE) AS GeneratedDates
	FROM GENERATE_SERIES(
		0,
		DATEDIFF(
			DAY,
			(SELECT MIN(BirthDate) FROM [AdventureWorks2022].[HumanResources].[Employee]),
			(SELECT MAX(BirthDate) FROM [AdventureWorks2022].[HumanResources].[Employee])
			),
		1)								-- GenerateDatesLevel1
	) AS X								-- GenerateWeekDaysLevel2
```

**Output (truncated):**
```
GeneratedDates  GeneratedWeekDays
1951-10-17      Wednesday
1951-10-18      Thursday
1951-10-19      Friday
1951-10-20      Saturday
1951-10-21      Sunday
1951-10-22      Monday
...
1991-05-29      Wednesday
1991-05-30      Thursday
1991-05-31      Friday
(14472 rows affected)
```

---

### Query 1.4 — Filter out weekends using `WHERE`

Adding `WHERE DATENAME(weekday, ...) NOT IN ('Saturday', 'Sunday')` removes all Saturday and Sunday rows, leaving only business days.

**T-SQL code**
```sql
SELECT																			-- GenerateBusinessDaysLevel3
	X.GeneratedDates
	, DATENAME(weekday, X.GeneratedDates) AS GeneratedWeekDays				
FROM
	(
	SELECT																		-- GeneratedDatesLevel1
		CAST(DATEADD(DAY, value, (SELECT MIN(BirthDate) FROM [AdventureWorks2022].[HumanResources].[Employee])) AS DATE) AS GeneratedDates
	FROM GENERATE_SERIES(
		0,
		DATEDIFF(
			DAY,
			(SELECT MIN(BirthDate) FROM [AdventureWorks2022].[HumanResources].[Employee]),
			(SELECT MAX(BirthDate) FROM [AdventureWorks2022].[HumanResources].[Employee])
			),
		1)																		-- GeneratedDatesLevel1
	) AS X	
WHERE DATENAME(weekday, X.GeneratedDates) NOT IN ('Saturday','Sunday')			-- GenerateBusinessDaysLevel3
```

**Output (truncated):**
```
GeneratedDates  GeneratedWeekDays
1951-10-17      Wednesday
1951-10-18      Thursday
1951-10-19      Friday
1951-10-22      Monday
1951-10-23      Tuesday
...
1991-05-27      Monday
1991-05-28      Tuesday
1991-05-29      Wednesday
1991-05-30      Thursday
1991-05-31      Friday
(10338 rows affected)
```

---
<br>

## 💡 Solution 2 — Using a Cartesian Product and iteration

### Approach
We use a Cartesian Product between a subquery returning the oldest and youngest birth dates and an `Iteration` subquery that generates sequential position numbers using `ROW_NUMBER()`. A `WHERE` clause limits the iteration to the total number of days. A `CASE` statement assigns `1` for business days and `0` for weekends, and `SUM()` totals the business days.

### Tables used

| Schema | Table |
|---|---|
| `HumanResources` | `Employee` |
| `Sales` | `SalesOrderDetail` |

---

### T-SQL code — Full solution

```sql
SELECT
    FORMAT(SUM(CASE
        WHEN DATENAME(WEEKDAY,
            DATEADD(DAY, Iteration.Position - 1, EmployeesBirthDates.OldestEmployee))
            IN ('Saturday', 'Sunday') THEN 0
        ELSE 1
    END), '#,#') AS CountBusinessdays
FROM (
    SELECT
        MIN(BirthDate) AS OldestEmployee
      , MAX(BirthDate) AS YoungestEmployee
    FROM [AdventureWorks2022].[HumanResources].[Employee]
	) AS EmployeesBirthDates,
	(
    SELECT
		ROW_NUMBER() OVER (ORDER BY SalesOrderDetailID) AS Position
    FROM [AdventureWorks2022].[Sales].[SalesOrderDetail]
	) AS Iteration
WHERE Iteration.Position <=
    DATEDIFF(DAY, EmployeesBirthDates.OldestEmployee, EmployeesBirthDates.YoungestEmployee) + 1
```

**Output:** Identical to Solution 1 — `10,338` business days.

---

## 🔍 Step-by-step explanation — Solution 2

### Query 2.1 — Generate dates via Cartesian Product and iteration
The Cartesian Product pairs the single-row `EmployeesBirthDates` subquery with each row from `Iteration`. `DATEADD(DAY, Position - 1, OldestEmployee)` converts each position number into a date. The `WHERE` clause stops the iteration at `14,472` (total calendar days + 1).

**T-SQL code**
```sql
SELECT																						-- IterationLevelandGeneratedDated2
	Iteration.Position
	, EmployeesBirthDates.OldestEmployee
	, EmployeesBirthDates.YoungestEmployee
	, DATEDIFF(DAY, EmployeesBirthDates.OldestEmployee, EmployeesBirthDates.YoungestEmployee) + 1 AS DiffDaysEmployeeDates
	, (DATEDIFF(DAY, EmployeesBirthDates.OldestEmployee, EmployeesBirthDates.YoungestEmployee) + 1) - Iteration.Position AS IterationOnWhereClause
	, DATEADD(DAY, Iteration.Position - 1, EmployeesBirthDates.OldestEmployee) AS GeneratedDates
FROM (
	SELECT																	-- DatesOldestYoungestOriginalTablesLevel1
		MIN(BirthDate) AS OldestEmployee
		, MAX(BirthDate) AS YoungestEmployee
	FROM [AdventureWorks2022].[HumanResources].[Employee]					-- DatesOldestYoungestOriginalTablesLevel1
	) AS EmployeesBirthDates
	,(
	SELECT
		ROW_NUMBER() OVER (ORDER BY SalesOrderDetailID) AS Position				-- IterationLevel1
	FROM [AdventureWorks2022].[Sales].[SalesOrderDetail]						-- IterationLevel1
	) AS Iteration
WHERE Iteration.Position <= DATEDIFF(DAY, EmployeesBirthDates.OldestEmployee, EmployeesBirthDates.YoungestEmployee) + 1
GROUP BY Iteration.Position, EmployeesBirthDates.OldestEmployee, EmployeesBirthDates.YoungestEmployee
ORDER BY DATEADD(DAY, Iteration.Position, EmployeesBirthDates.OldestEmployee)				-- IterationLevelandGeneratedDated2
```


**Output (truncated):**
```
Position  OldestEmployee  YoungestEmployee  DiffDays  GeneratedDates
1         1951-10-17      1991-05-31        14472     1951-10-17
2         1951-10-17      1991-05-31        14472     1951-10-18
3         1951-10-17      1991-05-31        14472     1951-10-19
...
14472     1951-10-17      1991-05-31        14472     1991-05-31
(14472 rows affected)
```

---

### Query 2.2 — Assign `1` or `0` per day using `CASE`
`DATENAME(WEEKDAY, ...)` returns the weekday name. The `CASE` statement assigns `0` for Saturday and Sunday, `1` for all other days. `SUM()` in the final query totals all the `1`s.

**T-SQL code**
```sql
SELECT																						-- GeneratedBusinessDatesLevel2
	DATEADD(DAY, Iteration.Position - 1, EmployeesBirthDates.OldestEmployee) AS GeneratedDates
	, DATENAME(WEEKDAY, DATEADD(DAY, Iteration.Position, EmployeesBirthDates.OldestEmployee)) AS GeneratedWeekdates
	, CASE
		WHEN DATENAME(WEEKDAY, DATEADD(DAY, Iteration.Position, EmployeesBirthDates.OldestEmployee)) IN ('Saturday','Sunday') THEN 0
		ELSE 1
		END AS CountBusinessdays
FROM (
	SELECT																	-- DatesOldestYoungestOriginalTablesLevel1
		MIN(BirthDate) AS OldestEmployee
		, MAX(BirthDate) AS YoungestEmployee
	FROM [AdventureWorks2022].[HumanResources].[Employee]					-- DatesOldestYoungestOriginalTablesLevel1
	) AS EmployeesBirthDates
	,(
	SELECT																	-- IterationLevel1
		ROW_NUMBER() OVER (ORDER BY SalesOrderDetailID) AS Position
	FROM [AdventureWorks2022].[Sales].[SalesOrderDetail]					-- IterationLevel1
	) AS Iteration
WHERE Iteration.Position <= DATEDIFF(DAY, EmployeesBirthDates.OldestEmployee, EmployeesBirthDates.YoungestEmployee) + 1
GROUP BY Iteration.Position, EmployeesBirthDates.OldestEmployee
ORDER BY DATEADD(DAY, Iteration.Position, EmployeesBirthDates.OldestEmployee)				-- GeneratedBusinessDatesLevel2
```


**Output (truncated):**
```
GeneratedDates  GeneratedWeekdates  CountBusinessdays
1951-10-17      Thursday            1
1951-10-18      Friday              1
1951-10-19      Saturday            0
1951-10-20      Sunday              0
1951-10-21      Monday              1
1951-10-22      Tuesday             1
...
1991-05-30      Friday              1
1991-05-31      Saturday            0
(14472 rows affected)
```

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
