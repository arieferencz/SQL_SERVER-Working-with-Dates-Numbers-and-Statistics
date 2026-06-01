# Counting the number of days in a year

## 🎯 Exercise
Count the total number of days in the current year — correctly identifying whether it is a regular year (365 days) or a leap year (366 days).

---

## 📝 Note

> The output shown below was captured in **2024**, which is a leap year. The result is therefore **366**.

---

## 💡 Solution

### Approach
We calculate the first day of the current year and the first day of next year using `DATEADD()` and `DATEDIFF()` applied to SQL Server's default `DATETIME` value (`1900-01-01`). We then use `DATEDIFF(DAY, ...)` between those two dates to get the total number of days in the current year. Since `DATEDIFF` counts day boundaries crossed, no `+ 1` is needed here — going from Jan 1 of this year to Jan 1 of next year crosses exactly as many boundaries as there are days in the year.

### T-SQL functions used

| Function | Purpose |
|---|---|
| `GETDATE()` | Returns the current date and time |
| `DATEDIFF(YEAR, 0, GETDATE())` | Calculates the number of whole years between `1900-01-01` and today |
| `DATEADD(YEAR, n, 0)` | Adds `n` years to `1900-01-01` — landing on Jan 1 of the target year |
| `DATEDIFF(DAY, start, end)` | Counts calendar days between two dates |
| `DISTINCT` | Ensures the subquery returns only one row regardless of the source table size |

### Table used

| Schema | Table |
|---|---|
| `Person` | `BusinessEntity` |

---

### T-SQL code

```sql
SELECT DATEDIFF(DAY, X.BeginningThisYear, X.BeginningNextYear) AS NumberDaysInCurrentYear
FROM (
    SELECT DISTINCT
        DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()),     0) AS BeginningThisYear
      , DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()) + 1, 0) AS BeginningNextYear
    FROM [AdventureWorks2022].[Person].[BusinessEntity]
) AS X
```

---

### Output

```
NumberDaysInCurrentYear
366
(1 row affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — Calculate the number of years since `1900-01-01`

```sql
SELECT DATEDIFF(YEAR, 0, GETDATE()) AS YearDiffToDefaultValue
```

```
YearDiffToDefaultValue
124
```

> `0` in a date context equals SQL Server's default `DATETIME` value `1900-01-01 00:00:00.000`. `DATEDIFF(YEAR, 0, GETDATE())` returns `124` — the number of whole years between `1900-01-01` and the current date in 2024.

---

### Query 1.2 — Calculate the first day of next year

```sql
SELECT DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()) + 1, 0) AS NextYear
```

```
NextYear
2025-01-01 00:00:00.000
```

Adding `124 + 1 = 125` years to `1900-01-01` lands on `2025-01-01` — the first day of next year.

---

### Query 1.3 — Calculate first day of this year and next year together

```sql
SELECT
    DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()),     0) AS BeginningThisYear
  , DATEADD(YEAR, DATEDIFF(YEAR, 0, GETDATE()) + 1, 0) AS BeginningNextYear
FROM [AdventureWorks2022].[Person].[BusinessEntity]
```

```
BeginningThisYear         BeginningNextYear
2024-01-01 00:00:00.000   2025-01-01 00:00:00.000
```

`DISTINCT` is used in the final query to collapse the 20,777 rows returned by `BusinessEntity` into a single row — since all rows produce the same two dates.

---

### Final Query — Count days between Jan 1 this year and Jan 1 next year

`DATEDIFF(DAY, '2024-01-01', '2025-01-01')` counts the number of day boundaries crossed between those two dates — which equals exactly the number of days in the year:

| Year | Days | Reason |
|---|---|---|
| 2024 | 366 | Leap year (Feb 29 exists) |
| 2023 | 365 | Regular year |
| 2000 | 366 | Leap year |
| 1900 | 365 | Not a leap year (divisible by 100 but not 400) |

> **Why no `+ 1` here?** Unlike counting days in a range (where you include both endpoints), here we are measuring the **length** of the year — the distance from the start of this year to the start of next year. That distance is exactly the number of days in the year, without needing to add 1.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
