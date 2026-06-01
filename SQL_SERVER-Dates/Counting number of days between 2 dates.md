# Counting the number of days between 2 dates

## 🎯 Exercise
Calculate the total number of calendar days between the birth date of the oldest employee and the birth date of the youngest employee.

---

## 💡 Solution

### Approach
We use a subquery to retrieve the oldest (`MIN`) and youngest (`MAX`) birth dates from the `Employee` table, then apply `DATEDIFF(DAY, ...)` to calculate the number of days between them. We add `+ 1` to include both the start and end date in the count.

### T-SQL functions used

| Function | Purpose |
|---|---|
| `MIN(BirthDate)` | Returns the earliest birth date — the oldest employee |
| `MAX(BirthDate)` | Returns the latest birth date — the youngest employee |
| `DATEDIFF(DAY, start, end)` | Calculates the number of calendar days between two dates |
| `FORMAT(value, format)` | Formats the result with a comma thousands separator |

### Table used

| Schema | Table |
|---|---|
| `HumanResources` | `Employee` |

---

### T-SQL code

```sql
SELECT FORMAT(DATEDIFF(DAY, OldestEmployee, YoungestEmployee) + 1, '#,#') AS NumberDaysDifference
FROM (
    SELECT
        MAX(BirthDate) AS YoungestEmployee
      , MIN(BirthDate) AS OldestEmployee
    FROM [AdventureWorks2022].[HumanResources].[Employee]
) AS X
```

---

### Output

```
NumberDaysDifference
14,472
(1 row affected)
```

---

## 🔍 Step-by-step explanation

### The subquery — retrieve oldest and youngest birth dates
The inner subquery uses `MIN(BirthDate)` and `MAX(BirthDate)` to return the two boundary dates in a single row:

```
OldestEmployee  YoungestEmployee
1951-10-17      1991-05-31
```

### `DATEDIFF(DAY, start, end)` — count calendar days
`DATEDIFF(DAY, '1951-10-17', '1991-05-31')` returns `14,471` — the number of day boundaries crossed between the two dates. Adding `+ 1` includes both the start and end dates themselves in the count, giving `14,472`.

**Why `+ 1`?**

`DATEDIFF()` counts the number of times a day boundary is crossed — so the difference between `2024-01-01` and `2024-01-03` is `2`, not `3`. If you want to count all days from Jan 1 to Jan 3 inclusive (Jan 1, Jan 2, Jan 3), you need to add `1`.

| Without `+ 1` | With `+ 1` |
|---|---|
| `DATEDIFF` counts boundaries crossed: 14,471 | Includes both endpoints: 14,472 |
| Use when measuring elapsed time | Use when counting all days in a range |

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
