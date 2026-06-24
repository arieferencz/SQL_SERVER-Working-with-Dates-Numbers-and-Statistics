# Counting the number of occurrences for every weekday in a year

## 🎯 Exercise
Count how many times each day of the week (Monday, Tuesday, Wednesday, etc.) occurs in the current year.

---

## 📝 Note

> The output shown below was captured in **2024**, which is a leap year with 366 days. Since `2024-01-01` is a Monday, Monday and Tuesday each appear **53 times** while the remaining five days appear **52 times**.

---

## 💡 Solution

### Approach
We generate a complete list of every day in the current year using `GENERATE_SERIES()` — from January 1st to December 31st. We then use `DATENAME(weekday, date)` to assign a weekday name to each date, and `GROUP BY` with `COUNT(*)` to count how many times each weekday name appears.

### T-SQL functions and clauses used

| Function | Purpose |
|---|---|
| `GETDATE()` | Returns the current date and time |
| `DATEDIFF(YEAR, 0, GETDATE())` | Calculates the number of whole years between `1900-01-01` and today |
| `DATEADD(YEAR, n, 0)` | Adds `n` years to `1900-01-01` — landing on Jan 1 of the target year |
| `DATEADD(DAY, -1, Jan1NextYear)` | Subtracts 1 day from Jan 1 next year to get Dec 31 this year |
| `GENERATE_SERIES(start, stop, step)` | Generates a sequential list of integers — one per day of the year |
| `DATEADD(DAY, value, start_date)` | Converts each integer into a calendar date starting from Jan 1 |
| `DATENAME(weekday, date)` | Returns the weekday name (e.g. `'Monday'`, `'Friday'`) for a given date |
| `COUNT(*)` | Counts the number of dates per weekday name |
| `SELECT DISTINCT` | Ensures the subqueries return a single date value |

### Table used

| Schema | Table |
|---|---|
| `Person` | `BusinessEntity` |

---

### T-SQL code — Full solution

```sql
SELECT
    DATENAME(weekday, X.GeneratedDates)  AS GeneratedWeekDays
  , COUNT(*)                             AS CountOccurrencesForWeekDaysInYear
FROM (
    SELECT
        CAST(DATEADD(DAY, value, (SELECT DISTINCT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0) FROM [AdventureWorks2022].[Person].[BusinessEntity])) AS DATE
			) AS GeneratedDates
    FROM GENERATE_SERIES(0,
        				DATEDIFF(DAY,
            				(SELECT DISTINCT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0)FROM [AdventureWorks2022].[Person].[BusinessEntity]),
            				(SELECT DISTINCT DATEADD(DAY, -1,(SELECT DISTINCT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()) + 1, 0)FROM [AdventureWorks2022].[Person].[BusinessEntity]))
             					FROM [AdventureWorks2022].[Person].[BusinessEntity])),
        				1)
	) AS X
GROUP BY DATENAME(weekday, X.GeneratedDates)
```

---

### Output

```
GeneratedWeekDays  CountOccurrencesForWeekDaysInYear
Friday             52
Monday             53
Saturday           52
Sunday             52
Thursday           52
Tuesday            53
Wednesday          52
(7 rows affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — Calculate the first day of this year and next year

**T-SQL code**
```sql
SELECT DISTINCT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0) AS BeginningThisYear
FROM [AdventureWorks2022].[Person].[BusinessEntity];

SELECT DISTINCT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()) + 1, 0) AS BeginningNextYear
FROM [AdventureWorks2022].[Person].[BusinessEntity];
```

**Output:**
```
BeginningThisYear             BeginningNextYear
2024-01-01 00:00:00.000       2025-01-01 00:00:00.000
```

> `SELECT DISTINCT` collapses the 20,777 rows returned by `BusinessEntity` into a single value. `DATEADD(DAY, -1, BeginningNextYear)` gives `2024-12-31` — the last day of the current year — which is used as the upper bound for `GENERATE_SERIES`.

---

### Query 1.2 — Generate all dates from Jan 1 to Dec 31 of the current year

`GENERATE_SERIES(0, 365, 1)` generates integers from `0` to `365` (366 values for the leap year 2024). `DATEADD(DAY, value, '2024-01-01')` converts each integer into a calendar date.

**T-SQL code**
```sql
SELECT											-- GenerateDatesInYear_1_Jan_31_DecLevel1
	CAST(DATEADD(DAY, value, (SELECT DISTINCT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0) FROM [AdventureWorks2022].[Person].[BusinessEntity])) AS DATE) AS GeneratedDates
FROM GENERATE_SERIES(0,
					DATEDIFF(DAY,
						(SELECT DISTINCT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0) FROM [AdventureWorks2022].[Person].[BusinessEntity]),
						(SELECT DISTINCT DATEADD(DAY, -1 , (SELECT DISTINCT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()) + 1, 0) FROM [AdventureWorks2022].[Person].[BusinessEntity])))),
					1)							-- GenerateDatesInYear_1_Jan_31_DecLevel1
```

**Output (truncated):** 366 rows — every day of the year.

```
GeneratedDates
2024-01-01
2024-01-02
2024-01-03
...
2024-12-29
2024-12-30
2024-12-31
(366 rows affected)
```

---

### Query 1.3 — Add weekday names using `DATENAME()`

`DATENAME(weekday, date)` returns the full weekday name for each generated date.

**T-SQL code**
```sql
SELECT																	-- GenerateWeekDaysLevel2
	DATENAME(weekday, X.GeneratedDates) AS GeneratedWeekDays
	, X.GeneratedDates
FROM (
	SELECT																-- GenerateDatesInYear_1_Jan_31_DecLevel1
		CAST(DATEADD(DAY, value, (SELECT DISTINCT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0) FROM [AdventureWorks2022].[Person].[BusinessEntity])) AS DATE) AS GeneratedDates
	FROM GENERATE_SERIES(0,
						DATEDIFF(DAY,
						(SELECT DISTINCT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0) FROM [AdventureWorks2022].[Person].[BusinessEntity]),
						(SELECT DISTINCT DATEADD(DAY, -1 , (SELECT DISTINCT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()) + 1, 0) FROM [AdventureWorks2022].[Person].[BusinessEntity])))),
						1)												-- GenerateDatesInYear_1_Jan_31_DecLevel1
	) AS X																-- GenerateWeekDaysLevel2
```


**Output (truncated):**
```
GeneratedWeekDays  GeneratedDates
Monday             2024-01-01
Tuesday            2024-01-02
Wednesday          2024-01-03
Thursday           2024-01-04
Friday             2024-01-05
Saturday           2024-01-06
...
Friday             2024-12-27
Saturday           2024-12-28
Sunday             2024-12-29
Monday             2024-12-30
Tuesday            2024-12-31
(366 rows affected)
```

---

### Query 1.4 — Count occurrences including the individual date (intermediate step)

Adding `GROUP BY DATENAME(weekday, ...), X.GeneratedDates` and `COUNT(*)` gives one row per day with a count of `1` — since each date appears exactly once. This is an intermediate step used to verify the data before collapsing by weekday name only.

**T-SQL code**
```sql
SELECT																	-- CountOccurrencesLevel3
	DATENAME(weekday, X.GeneratedDates) AS GeneratedWeekDays
	, X.GeneratedDates
	, COUNT(*) AS CountOccurrencesForWeekDaysInYear
FROM (
	SELECT																-- GenerateDatesInYear_1_Jan_31_DecLevel1
		CAST(DATEADD(DAY, value, (SELECT DISTINCT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0) FROM [AdventureWorks2022].[Person].[BusinessEntity])) AS DATE) AS GeneratedDates
	FROM GENERATE_SERIES(0,
						DATEDIFF(DAY,
							(SELECT DISTINCT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0) FROM [AdventureWorks2022].[Person].[BusinessEntity]),
							(SELECT DISTINCT DATEADD(DAY, -1 , (SELECT DISTINCT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()) + 1, 0) FROM [AdventureWorks2022].[Person].[BusinessEntity])))),
						1)												-- GenerateDatesInYear_1_Jan_31_DecLevel1
	) AS X		
GROUP BY DATENAME(weekday, X.GeneratedDates), X.GeneratedDates			-- CountOccurrencesLevel3
```


**Output (truncated):**
```
GeneratedWeekDays  GeneratedDates  CountOccurrencesForWeekDaysInYear
Monday             2024-01-01      1
Tuesday            2024-01-02      1
Wednesday          2024-01-03      1
...
Monday             2024-12-30      1
Tuesday            2024-12-31      1
(366 rows affected)
```

---

### Final Query (Query 1) — Remove the date column and group by weekday name only

Removing `X.GeneratedDates` from the `SELECT` and `GROUP BY` collapses all 366 rows into 7 rows — one per weekday — with `COUNT(*)` summing all occurrences of each weekday name across the year.

**Why Monday and Tuesday appear 53 times in 2024:**
366 days ÷ 7 days = 52 weeks remainder 2 days. Since `2024-01-01` is a Monday, the two extra days are Monday (Jan 1) and Tuesday (Dec 31) — making those weekdays appear 53 times while the other five appear 52 times.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
