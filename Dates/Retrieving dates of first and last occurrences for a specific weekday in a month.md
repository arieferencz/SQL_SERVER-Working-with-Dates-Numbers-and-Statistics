# Retrieving the dates of first and last occurrences for a specific weekday in a month

## 🎯 Exercise
Retrieve the dates of the **first** and **last** occurrence of a specific weekday in the current month — in this case, all **Tuesdays** in December 2024.

---

## 📝 Note

> The output shown below was captured in **December 2024**. Since `GETDATE()` is used, the query automatically reflects the current month when run.
>
> To retrieve a different weekday, replace `'Tuesday'` in the `CASE` statement and `WHERE` clause with any other weekday name.

---

## 💡 Solution

### Approach
We generate all calendar dates for the current month using `GENERATE_SERIES()`. We then add a `CASE` statement flag that marks Tuesdays as `1` and all other days as `0`. We filter to keep only the flagged Tuesday rows, then use `MIN()` and `MAX()` to return the first and last Tuesday of the month in a single row.

### T-SQL functions, case expressions, and clauses used

| Function | Purpose |
|---|---|
| `DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()), 0)` | Calculates the first day of the current month |
| `DATEADD(MONTH, DATEDIFF(MONTH, -1, GETDATE()), -1)` | Calculates the last day of the current month |
| `GENERATE_SERIES(start, stop, step)` | Generates sequential integers — one per day of the month |
| `DATEADD(DAY, value, start_date)` | Converts each integer into a calendar date |
| `DATENAME(weekday, date)` | Returns the weekday name (e.g. `'Tuesday'`) |
| `CASE WHEN DATENAME(...) = 'Tuesday' THEN 1 ELSE 0 END` | Flags Tuesday rows as `1`, all others as `0` |
| `MIN(date)` | Returns the first Tuesday of the month |
| `MAX(date)` | Returns the last Tuesday of the month |
| `SELECT DISTINCT` | Ensures subqueries return a single date value |

### Table used

| Schema | Table |
|---|---|
| `Person` | `BusinessEntity` |

---

### T-SQL code

```sql
SELECT
    Y.GeneratedWeekDays
  , MIN(Y.GeneratedDates) AS DateFirstTuesdayCurrentMonth
  , MAX(Y.GeneratedDates) AS DateLastTuesdayCurrentMonth
FROM (
    SELECT
        DATENAME(weekday, X.GeneratedDates)  AS GeneratedWeekDays
      , X.GeneratedDates
      , CASE
            WHEN DATENAME(weekday, X.GeneratedDates) = 'Tuesday' THEN 1
            ELSE 0
        END                                  AS IS_Tuesday
    FROM (
        SELECT
            CAST(DATEADD(DAY, value,
                (SELECT DISTINCT DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()), 0)
                 FROM [AdventureWorks2022].[Person].[BusinessEntity])
            ) AS DATE) AS GeneratedDates
        FROM GENERATE_SERIES(
            0,
            DATEDIFF(DAY,
                (SELECT DISTINCT DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()), 0)
                 FROM [AdventureWorks2022].[Person].[BusinessEntity]),
                (SELECT DISTINCT DATEADD(MONTH, DATEDIFF(MONTH, -1, GETDATE()), -1)
                 FROM [AdventureWorks2022].[Person].[BusinessEntity])
            ),
            1
        )
    ) AS X
) AS Y
WHERE Y.IS_Tuesday = 1
GROUP BY Y.GeneratedWeekDays
```

---

### Output — December 2024

```
GeneratedWeekDays  DateFirstTuesdayCurrentMonth  DateLastTuesdayCurrentMonth
Tuesday            2024-12-03                    2024-12-31
(1 row affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — Calculate the first and last day of the current month

**T-SQL code:**
```sql
SELECT CAST(DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()), 0) AS DATE)   AS FirstDayofCurrentMonth
SELECT CAST(DATEADD(MONTH, DATEDIFF(MONTH, -1, GETDATE()), -1) AS DATE) AS LastDayofCurrentMonth
```

**Output:**
```
FirstDayofCurrentMonth  LastDayofCurrentMonth
2024-12-01              2024-12-31
```

---

### Query 1.2 — Generate all calendar dates for the current month

`GENERATE_SERIES(0, 30, 1)` generates integers from `0` to `30` (31 days in December). `DATEADD(DAY, value, '2024-12-01')` converts each into a calendar date.

**T-SQL code:**
```sql
SELECT										-- GenerateDatesInYear_1_Dec_31_DecLevel1
CAST(DATEADD(DAY, value, (SELECT DISTINCT DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()), 0) FROM [AdventureWorks2022].[Person].[BusinessEntity])) AS DATE) AS GeneratedDates
FROM GENERATE_SERIES(0
					, DATEDIFF(DAY
						, (SELECT DISTINCT DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()), 0) FROM [AdventureWorks2022].[Person].[BusinessEntity])
						, (SELECT DISTINCT DATEADD(MONTH,DATEDIFF(MONTH, -1, GETDATE()),-1) FROM [AdventureWorks2022].[Person].[BusinessEntity]))
							, 1)			-- GenerateDatesInYear_1_Dec_31_DecLevel1
```

**Output (truncated):** 31 rows — every day in December 2024.

```
GeneratedDates
2024-12-01
2024-12-02
2024-12-03
...
2024-12-29
2024-12-30
2024-12-31
(31 rows affected)
```

---

### Query 1.3 — Add weekday names and Tuesday flag using `DATENAME()` and `CASE`

`DATENAME(weekday, date)` assigns the weekday name to each date. The `CASE` statement flags each Tuesday as `1` and all other days as `0`.

**T-SQL code:**
```sql
SELECT DATENAME(weekday, X.GeneratedDates) AS GeneratedWeekDays			-- GenerateWeekDaysLevel2
, X.GeneratedDates
, CASE	WHEN DATENAME(weekday, X.GeneratedDates) = 'Tuesday'
		THEN 1 
		ELSE 0
		END AS IS_Tuesday
FROM (
	SELECT										-- GenerateDatesInYear_1_Dec_31_DecLevel1
	CAST(DATEADD(DAY, value, (SELECT DISTINCT DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()), 0) FROM [AdventureWorks2022].[Person].[BusinessEntity])) AS DATE) AS GeneratedDates
	FROM GENERATE_SERIES(0
						, DATEDIFF(DAY
							, (SELECT DISTINCT DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()), 0) FROM [AdventureWorks2022].[Person].[BusinessEntity])
							, (SELECT DISTINCT DATEADD(MONTH,DATEDIFF(MONTH, -1, GETDATE()),-1) FROM [AdventureWorks2022].[Person].[BusinessEntity]))
								, 1)			-- GenerateDatesInYear_1_Dec_31_DecLevel1
	) AS X									-- GenerateWeekDaysLevel2
```

**Output:**

```
GeneratedWeekDays  GeneratedDates  IS_Tuesday
Sunday             2024-12-01      0
Monday             2024-12-02      0
Tuesday            2024-12-03      1
Wednesday          2024-12-04      0
Thursday           2024-12-05      0
Friday             2024-12-06      0
Saturday           2024-12-07      0
Sunday             2024-12-08      0
Monday             2024-12-09      0
Tuesday            2024-12-10      1
...
Monday             2024-12-23      0
Tuesday            2024-12-24      1
Wednesday          2024-12-25      0
Thursday           2024-12-26      0
Friday             2024-12-27      0
Saturday           2024-12-28      0
Sunday             2024-12-29      0
Monday             2024-12-30      0
Tuesday            2024-12-31      1
(31 rows affected)
```

---

### Query 1.4 — Filter to Tuesdays only using `WHERE IS_Tuesday = 1`

Adding `WHERE Y.IS_Tuesday = 1` keeps only the 5 Tuesday rows.

**T-SQL code:**
```sql
SELECT Y.GeneratedWeekDays				-- KeepOnlyTuesdaysLevel3
, Y.GeneratedDates
FROM (
	SELECT DATENAME(weekday, X.GeneratedDates) AS GeneratedWeekDays			-- GenerateWeekDaysLevel2
	, X.GeneratedDates
	, CASE	WHEN DATENAME(weekday, X.GeneratedDates) = 'Tuesday'
			THEN 1 
			ELSE 0
			END AS IS_Tuesday
	FROM (
		SELECT										-- GenerateDatesInYear_1_Dec_31_DecLevel1
		CAST(DATEADD(DAY, value, (SELECT DISTINCT DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()), 0) FROM [AdventureWorks2022].[Person].[BusinessEntity])) AS DATE) AS GeneratedDates
		FROM GENERATE_SERIES(0
							, DATEDIFF(DAY
								, (SELECT DISTINCT DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()), 0) FROM [AdventureWorks2022].[Person].[BusinessEntity])
								, (SELECT DISTINCT DATEADD(MONTH,DATEDIFF(MONTH, -1, GETDATE()),-1) FROM [AdventureWorks2022].[Person].[BusinessEntity]))
									, 1)			-- GenerateDatesInYear_1_Dec_31_DecLevel1
		) AS X									-- GenerateWeekDaysLevel2
	) AS Y
WHERE Y.IS_Tuesday = 1					-- KeepOnlyTuesdaysLevel3
```

**Output:**

```
GeneratedWeekDays  GeneratedDates
Tuesday            2024-12-03
Tuesday            2024-12-10
Tuesday            2024-12-17
Tuesday            2024-12-24
Tuesday            2024-12-31
(5 rows affected)
```

---

### Final Query (Query 1) — Return first and last Tuesday using `MIN()` and `MAX()`

Adding `MIN(Y.GeneratedDates)` and `MAX(Y.GeneratedDates)` with `GROUP BY Y.GeneratedWeekDays` collapses all 5 Tuesday rows into a single row showing the first and last occurrence.

**Final output:**

```
GeneratedWeekDays  DateFirstTuesdayCurrentMonth  DateLastTuesdayCurrentMonth
Tuesday            2024-12-03                    2024-12-31
```

### Changing the target weekday

To retrieve the first and last occurrence of any other weekday, replace `'Tuesday'` in both the `CASE` statement and the `WHERE` clause:

```sql
-- For Mondays:
CASE WHEN DATENAME(weekday, X.GeneratedDates) = 'Monday' THEN 1 ELSE 0 END AS IS_Monday
...
WHERE Y.IS_Monday = 1

-- For Fridays:
CASE WHEN DATENAME(weekday, X.GeneratedDates) = 'Friday' THEN 1 ELSE 0 END AS IS_Friday
...
WHERE Y.IS_Friday = 1
```

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
