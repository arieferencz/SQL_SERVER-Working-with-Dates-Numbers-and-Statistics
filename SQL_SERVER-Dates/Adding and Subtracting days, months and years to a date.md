# Adding and subtracting days, months and years to a date

## 🎯 Exercise
Add and subtract 365 days, 12 months, and 1 year to the `HireDate` of each employee — and observe how the three approaches differ in their results around leap years.

---

## 💡 Solution

### Approach
We use `DATEADD()` three times per direction — once with `DAY`, once with `MONTH`, and once with `YEAR` — to demonstrate how each interval unit behaves differently when crossing a leap year boundary.

### T-SQL functions used

| Function | Purpose |
|---|---|
| `DATEADD(interval, number, date)` | Adds or subtracts a specified number of the given interval (DAY, MONTH, YEAR) to a date |

### Table used

| Schema | Table |
|---|---|
| `HumanResources` | `Employee` |

---

### T-SQL code

```sql
SELECT
    BusinessEntityID
  , JobTitle
  , DATEADD(DAY,   -365, HireDate) AS HDMinus365Days
  , DATEADD(MONTH,  -12, HireDate) AS HDMinus12Months
  , DATEADD(YEAR,    -1, HireDate) AS HDMinus1Year
  , DATEADD(DAY,    365, HireDate) AS HDPlus365Days
  , DATEADD(MONTH,   12, HireDate) AS HDPlus12Months
  , DATEADD(YEAR,     1, HireDate) AS HDPlus1Year
FROM [AdventureWorks2022].[HumanResources].[Employee]
```

---

### Output (truncated)

```
BusinessEntityID  JobTitle                        HDMinus365Days  HDMinus12Months  HDMinus1Year  HDPlus365Days  HDPlus12Months  HDPlus1Year
1                 Chief Executive Officer          1/15/2008       1/14/2008        1/14/2008     1/14/2010      1/14/2010       1/14/2010
2                 Vice President of Engineering    1/31/2007       1/31/2007        1/31/2007     1/30/2009      1/31/2009       1/31/2009
3                 Engineering Manager              11/11/2006      11/11/2006       11/11/2006    11/10/2008     11/11/2008      11/11/2008
4                 Senior Tool Designer             12/5/2006       12/5/2006        12/5/2006     12/4/2008      12/5/2008       12/5/2008
5                 Design Engineer                  1/6/2007        1/6/2007         1/6/2007      1/5/2009       1/6/2009        1/6/2009
...
285               Pacific Sales Manager            3/14/2012       3/14/2012        3/14/2012     3/14/2014      3/14/2014       3/14/2014
286               Sales Representative             5/30/2012       5/30/2012        5/30/2012     5/30/2014      5/30/2014       5/30/2014
287               Sales Representative             4/17/2011       4/16/2011        4/16/2011     4/16/2013      4/16/2013       4/16/2013
288               Sales Representative             5/30/2012       5/30/2012        5/30/2012     5/30/2014      5/30/2014       5/30/2014
289               Sales Representative             5/31/2011       5/30/2011        5/30/2011     5/30/2013      5/30/2013       5/30/2013
290               Sales Representative             5/31/2011       5/30/2011        5/30/2011     5/30/2013      5/30/2013       5/30/2013
(290 rows affected)
```

---

## 🔍 Step-by-step explanation

### How the three interval units differ

`DATEADD(DAY, ±365, date)`, `DATEADD(MONTH, ±12, date)`, and `DATEADD(YEAR, ±1, date)` are not always equivalent — they produce different results when the date range crosses a **leap year**.

A regular year has 365 days. A leap year has 366 days. This means:

- `DATEADD(DAY, 365, date)` always moves exactly 365 calendar days — it may land on a different calendar date if a leap day is crossed
- `DATEADD(MONTH, 12, date)` moves exactly 12 months — it always lands on the same day of the month one year later
- `DATEADD(YEAR, 1, date)` moves exactly 1 year — it always lands on the same calendar date one year later

**Visible example from the output — `BusinessEntityID = 3` (HireDate = 2007-11-11):**

```
HDPlus365Days   = 2008-11-10  ← 365 days later lands on Nov 10 (2008 is a leap year — 366 days)
HDPlus12Months  = 2008-11-11  ← 12 months later lands on Nov 11 ✓
HDPlus1Year     = 2008-11-11  ← 1 year later lands on Nov 11 ✓
```

Because 2008 is a leap year, adding 365 days to a date in 2007 does not land on the same calendar date as adding 1 year — it lands one day earlier.

**Another example — `BusinessEntityID = 2` (HireDate = 2008-01-31):**

```
HDPlus365Days   = 2009-01-30  ← 365 days later (2008 is a leap year with 366 days)
HDPlus12Months  = 2009-01-31  ← 12 months later ✓
HDPlus1Year     = 2009-01-31  ← 1 year later ✓
```

### Summary of differences

| Approach | Behaviour | Use when |
|---|---|---|
| `DATEADD(DAY, ±365, date)` | Moves exactly 365 calendar days — may shift the calendar date if a leap year is crossed | You need an exact number of days |
| `DATEADD(MONTH, ±12, date)` | Moves exactly 12 months — always lands on the same day of the month | You need to move by whole months |
| `DATEADD(YEAR, ±1, date)` | Moves exactly 1 year — always lands on the same calendar date | You need to move by whole years |

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
