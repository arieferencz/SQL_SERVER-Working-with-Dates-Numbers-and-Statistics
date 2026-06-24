# Retrieving the dates of a specific weekday in a year

## 🎯 Exercise
Retrieve all dates during the current year that fall on a specific weekday — in this case, all **Sundays**.

---

## 📝 Note

> The output shown below was captured in **2024**. Since `GETDATE()` is used, the query automatically returns all Sundays for whatever the current year is when it is run.
>
> To retrieve a different weekday, simply change `'Sunday'` in the `WHERE` clause to any other weekday name: `'Monday'`, `'Tuesday'`, `'Wednesday'`, `'Thursday'`, `'Friday'`, or `'Saturday'`.

---

## 💡 Solution

### Approach
We generate a complete list of every calendar day in the current year using `GENERATE_SERIES()`. We then use `DATENAME(weekday, date)` to assign a weekday name to each date, and filter with `WHERE DATENAME(weekday, ...) IN ('Sunday')` to keep only the dates that fall on Sundays.

### T-SQL functions and clauses used

| Function | Purpose |
|---|---|
| `DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0)` | Calculates January 1st of the current year |
| `DATEADD(DAY, -1, Jan1NextYear)` | Calculates December 31st of the current year |
| `GENERATE_SERIES(start, stop, step)` | Generates sequential integers — one per day of the year |
| `DATEADD(DAY, value, start_date)` | Converts each integer into a calendar date |
| `DATENAME(weekday, date)` | Returns the weekday name (e.g. `'Sunday'`) for a given date |
| `SELECT DISTINCT` | Ensures subqueries return a single date value |

### Table used

| Schema | Table |
|---|---|
| `Person` | `BusinessEntity` |

---

### T-SQL code — Full solution

```sql
SELECT
    DATENAME(weekday, X.GeneratedDates) AS GeneratedWeekDays
  , X.GeneratedDates
FROM (
    SELECT CAST(DATEADD(DAY, value, (SELECT DISTINCT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0) FROM [AdventureWorks2022].[Person].[BusinessEntity])) AS DATE
        ) AS GeneratedDates
    FROM GENERATE_SERIES(0,
                        DATEDIFF(DAY, 
                                (SELECT DISTINCT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0) FROM [AdventureWorks2022].[Person].[BusinessEntity]),
                                (SELECT DISTINCT DATEADD(DAY, -1, (SELECT DISTINCT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()) + 1, 0) FROM [AdventureWorks2022].[Person].[BusinessEntity])) FROM [AdventureWorks2022].[Person].[BusinessEntity])),
                        1)
    ) AS X
WHERE DATENAME(weekday, X.GeneratedDates) IN ('Sunday')
```

---

### Output (truncated)
```
GeneratedWeekDays  GeneratedDates
Sunday             2024-01-07
Sunday             2024-01-14
Sunday             2024-01-21
Sunday             2024-01-28
Sunday             2024-02-04
Sunday             2024-02-11
Sunday             2024-02-18
Sunday             2024-02-25
...
Sunday             2024-11-17
Sunday             2024-11-24
Sunday             2024-12-01
Sunday             2024-12-08
Sunday             2024-12-15
Sunday             2024-12-22
Sunday             2024-12-29
(52 rows affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — Generate all dates from Jan 1 to Dec 31 of the current year

This is the same date generation logic used in the **"Counting the number of occurrences for every weekday in a year"** exercise. `GENERATE_SERIES(0, 365, 1)` generates 366 integers (2024 is a leap year). `DATEADD(DAY, value, '2024-01-01')` converts each into a calendar date.

**T-SQL code**
```sql
SELECT                                    -- GenerateDatesInYear_1_Jan_31_DecLevel1
    CAST(DATEADD(DAY, value, (SELECT DISTINCT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0) FROM [AdventureWorks2022].[Person].[BusinessEntity])) AS DATE) AS GeneratedDates
FROM GENERATE_SERIES(0,
					DATEDIFF(DAY,
					(SELECT DISTINCT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0) FROM [AdventureWorks2022].[Person].[BusinessEntity]),
					(SELECT DISTINCT DATEADD(DAY, -1 , (SELECT DISTINCT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()) + 1, 0) FROM [AdventureWorks2022].[Person].[BusinessEntity])))),
					1)			        -- GenerateDatesInYear_1_Jan_31_DecLevel1
```


**Output (truncated):** 366 rows — every day of 2024.
```
GeneratedDates
2024-01-01
2024-01-02
2024-01-03
2024-01-04
2024-01-05
2024-01-06
...
2024-12-29
2024-12-30
2024-12-31
(366 rows affected)
```

---

### Query 1.2 — Add weekday names using `DATENAME()`

`DATENAME(weekday, date)` assigns the full weekday name to each generated date.

**T-SQL code**
```sql
SELECT                                        	-- GenerateWeekDaysLevel2
    DATENAME(weekday, X.GeneratedDates) AS GeneratedWeekDays
    , X.GeneratedDates
FROM (
    SELECT										-- GenerateDatesInYear_1_Jan_31_DecLevel1
        CAST(DATEADD(DAY, value, (SELECT DISTINCT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0) FROM [AdventureWorks2022].[Person].[BusinessEntity])) AS DATE) AS GeneratedDates
    FROM GENERATE_SERIES(0, 
					    DATEDIFF(DAY,
						(SELECT DISTINCT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()), 0) FROM [AdventureWorks2022].[Person].[BusinessEntity]),
						(SELECT DISTINCT DATEADD(DAY, -1 , (SELECT DISTINCT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()) + 1, 0) FROM [AdventureWorks2022].[Person].[BusinessEntity])))),
						1)        			-- GenerateDatesInYear_1_Jan_31_DecLevel1
	) AS X									-- GenerateWeekDaysLevel2
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

### Final Query (Query 1) — Filter to Sundays only

Adding `WHERE DATENAME(weekday, X.GeneratedDates) IN ('Sunday')` keeps only the 52 rows where the weekday name is `'Sunday'` — returning all Sunday dates for the year.

> **Why 52 Sundays in 2024?** 2024 has 366 days. 366 ÷ 7 = 52 weeks remainder 2 days. The first day of 2024 is a Monday, so the two extra days are Monday (Jan 1) and Tuesday (Dec 31). Since neither is a Sunday, 2024 has exactly 52 Sundays.

### Changing the target weekday

To retrieve all dates for any other weekday, simply replace `'Sunday'` in the `WHERE` clause:

```sql
WHERE DATENAME(weekday, X.GeneratedDates) IN ('Monday')    -- 53 Mondays in 2024
WHERE DATENAME(weekday, X.GeneratedDates) IN ('Tuesday')   -- 53 Tuesdays in 2024
WHERE DATENAME(weekday, X.GeneratedDates) IN ('Saturday')  -- 52 Saturdays in 2024
```

You can also retrieve multiple weekdays at once:

```sql
WHERE DATENAME(weekday, X.GeneratedDates) IN ('Saturday', 'Sunday')  -- all weekend dates
```

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
