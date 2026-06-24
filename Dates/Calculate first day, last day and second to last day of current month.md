# Calculate first day, last day and second to last day of current month

## 🎯 Exercise
Calculate the first day, last day, and second to last day of the current month — using multiple different approaches for each.

---

## 📝 Note

> All outputs shown below were captured on **2024-12-02**. Since these queries use `GETDATE()`, results will vary depending on the date the query is run.

---

## 💡 Exercise 1 — First day of the current month (4 solutions)

### T-SQL functions used

| Function | Purpose |
|---|---|
| `GETDATE()` | Returns the current date and time |
| `DAY(date)` | Returns the day number of the month (e.g. `2` for Dec 2) |
| `DATEADD(interval, number, date)` | Adds or subtracts a number of the given interval to a date |
| `DATEDIFF(interval, start, end)` | Returns the difference between two dates in the given interval |
| `CAST(value AS DATE)` | Converts a datetime value to a date-only value |

---

### Solution 1.1 — Using `GETDATE()` and `DAY()`

**How it works step by step:**

```sql
SELECT
    CAST(GETDATE() AS DATE)                        AS TodayGetDate   -- 2024-12-02
  , -DAY(GETDATE())                                AS Step1          -- -2
  , -DAY(GETDATE()) + 1                            AS Step2          -- -1
  , CAST(GETDATE() - DAY(GETDATE()) + 1 AS DATE)   AS Step3          -- 2024-12-01
```

**T-SQL code of Solution 1.1**
```sql
SELECT CAST(GETDATE() - DAY(GETDATE()) + 1 AS DATE) AS FirstDayofCurrentMonth
```

**Output of Solution 1.1:**
```
TodayGetDate	Step1	Step2	Step3_FirstDayofCurrentMonth
2024-12-02		-2		-1		2024-12-01
```

`DAY(GETDATE())` returns `2` (the current day number). Subtracting `2` from today and adding `1` moves back to the 1st of the month.

---

### Solution 1.2 — Using `DATEADD()` and `DAY(GETDATE() - 1)`

```sql
SELECT CAST(DATEADD(DAY, -(DAY(GETDATE() - 1)), GETDATE()) AS DATE) AS FirstDayofCurrentMonth
```

**How it works step by step:**

```sql
SELECT
    CAST(GETDATE() AS DATE)                                        AS TodayGetDate  -- 2024-12-02
  , DAY(GETDATE())                                                 AS Step1         -- 2
  , DAY(GETDATE() - 1)                                             AS Step2         -- 1
  , -(DAY(GETDATE() - 1))                                          AS Step3         -- -1
  , CAST(DATEADD(DAY, -(DAY(GETDATE() - 1)), GETDATE()) AS DATE)   AS Step4         -- 2024-12-01
```

```
TodayGetDate  Step1  Step2  Step3  Step4_FirstDayofCurrentMonth
2024-12-02    2      1      -1     2024-12-01
```

`DAY(GETDATE() - 1)` returns the day number of yesterday (`1`). Subtracting that from today lands on the 1st.

---

### Solution 1.3 — Using `DATEADD()` and `1 - DAY()`

```sql
SELECT CAST(DATEADD(DAY, 1 - (DAY(GETDATE())), GETDATE()) AS DATE) AS FirstDayofCurrentMonth
```

**How it works step by step:**

```sql
SELECT
    CAST(GETDATE() AS DATE)                                       AS TodayGetDate  -- 2024-12-02
  , DAY(GETDATE())                                                AS Step1         -- 2
  , 1 - (DAY(GETDATE()))                                          AS Step2         -- -1
  , CAST(DATEADD(DAY, 1 - (DAY(GETDATE())), GETDATE()) AS DATE)   AS Step3         -- 2024-12-01
```

```
TodayGetDate  Step1  Step2  Step3_FirstDayofCurrentMonth
2024-12-02    2      -1     2024-12-01
```

`1 - DAY(GETDATE())` = `1 - 2` = `-1`. Adding `-1` day to today moves back to the 1st.

---

### Solution 1.4 — Using `DATEADD()`, `DATEDIFF()`, and the SQL Server DATETIME default (`1900-01-01`)

```sql
SELECT CAST(DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()), 0) AS DATE) AS FirstDayofCurrentMonth
```

**How it works step by step:**

```sql
SELECT
    CAST(GETDATE() AS DATE)                                        AS TodayGetDate  -- 2024-12-02
  , DATEDIFF(MONTH, 0, GETDATE())                                  AS Step1         -- 1499
  , CAST(DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()), 0) AS DATE) AS Step2         -- 2024-12-01
```

```
TodayGetDate  Step1  Step2_FirstDayofCurrentMonth
2024-12-02    1499   2024-12-01
```

> **Key concept:** In SQL Server, the integer `0` in a date context equals the default `DATETIME` value `1900-01-01 00:00:00.000`. `DATEDIFF(MONTH, 0, GETDATE())` calculates the number of whole months between `1900-01-01` and today — in this case 1,499. Adding those 1,499 months back to `1900-01-01` always lands on the **1st of the current month**, regardless of which day of the month today is.

---

## 💡 Exercise 2 — Last day of the current month (4 solutions)

### T-SQL functions used (additional)

| Function | Purpose |
|---|---|
| `EOMONTH(date)` | Returns the last day of the month for the given date |

---

### Solution 2.1 — Using `EOMONTH()` *(simplest)*

```sql
SELECT EOMONTH(GETDATE()) AS LastDayofCurrentMonth
```

```
LastDayofCurrentMonth
2024-12-31
```

`EOMONTH()` directly returns the last day of the month. This is the most concise approach.

---

### Solution 2.2 — Using `DATEADD()`, `DATEDIFF()`, and the DATETIME default minus one month

```sql
SELECT CAST(DATEADD(MONTH, DATEDIFF(MONTH, -1, GETDATE()), -1) AS DATE) AS LastDayofCurrentMonth
```

**How it works step by step:**

```sql
SELECT
    CAST(GETDATE() AS DATE)                                           AS TodayGetDate  -- 2024-12-02
  , DATEDIFF(MONTH, -1, GETDATE())                                    AS Step1         -- 1500
  , CAST(DATEADD(MONTH, 0, -1) AS DATE)                               AS Step2         -- 1899-12-31
  , CAST(DATEADD(MONTH, DATEDIFF(MONTH, -1, GETDATE()), -1) AS DATE)  AS Step3         -- 2024-12-31
```

```
TodayGetDate  Step1  Step2       Step3_LastDayofCurrentMonth
2024-12-02    1500   1899-12-31  2024-12-31
```

> `-1` in a date context equals `1899-12-31` (one day before `1900-01-01`). `DATEDIFF(MONTH, -1, GETDATE())` returns 1,500 months between `1899-12-01` and today. Adding 1,500 months to `1899-12-31` lands on `2024-12-31`.

---

### Solution 2.3 — Using `DATEADD()`, `DATEDIFF()`, and first-of-next-month minus 1 day

```sql
SELECT CAST(DATEADD(DAY, -1, DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()) + 1, 0)) AS DATE) AS LastDayofCurrentMonth
```

**How it works step by step:**

```sql
SELECT
    CAST(GETDATE() AS DATE)                                                               AS TodayGetDate  -- 2024-12-02
  , DATEDIFF(MONTH, 0, GETDATE())                                                         AS Step1         -- 1499
  , DATEDIFF(MONTH, 0, GETDATE()) + 1                                                     AS Step2         -- 1500
  , CAST(DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()) + 1, 0) AS DATE)                    AS Step3         -- 2025-01-01
  , CAST(DATEADD(DAY, -1, DATEADD(MONTH, DATEDIFF(MONTH, 0, GETDATE()) + 1, 0)) AS DATE)  AS Step4         -- 2024-12-31
```

```
TodayGetDate  Step1  Step2  Step3       Step4_LastDayofCurrentMonth
2024-12-02    1499   1500   2025-01-01  2024-12-31
```

Adding 1,500 months to `1900-01-01` gives the first of next month (`2025-01-01`). Subtracting 1 day gives the last day of the current month.

---

### Solution 2.4 — Using `DATEADD()` and `DAY()` (first-of-month then +1 month then -1 day)

```sql
SELECT CAST(DATEADD(DAY, -1, DATEADD(MONTH, 1, DATEADD(DAY, 1 - (DAY(GETDATE())), GETDATE()))) AS DATE) AS LastDayofCurrentMonth
```

**How it works step by step:**

```sql
SELECT
    CAST(GETDATE() AS DATE)                                                                          AS TodayGetDate  -- 2024-12-02
  , DAY(GETDATE())                                                                                   AS Step1         -- 2
  , 1 - (DAY(GETDATE()))                                                                             AS Step2         -- -1
  , CAST(DATEADD(DAY, 1 - (DAY(GETDATE())), GETDATE()) AS DATE)                                      AS Step3         -- 2024-12-01
  , CAST(DATEADD(MONTH, 1, DATEADD(DAY, 1 - (DAY(GETDATE())), GETDATE())) AS DATE)                   AS Step4         -- 2025-01-01
  , CAST(DATEADD(DAY, -1, DATEADD(MONTH, 1, DATEADD(DAY, 1 - (DAY(GETDATE())), GETDATE()))) AS DATE) AS Step5         -- 2024-12-31
```

```
TodayGetDate  Step1  Step2  Step3       Step4       Step5_LastDayofCurrentMonth
2024-12-02    2      -1     2024-12-01  2025-01-01  2024-12-31
```

Step 3 finds the first of the current month (from Solution 1.3). Step 4 adds 1 month to get the first of next month. Step 5 subtracts 1 day.

---

## 💡 Exercise 3 — Second to last day of the current month (2 solutions)

### Solution 3.1 — Using `EOMONTH()` and `DATEADD()` *(simplest)*

```sql
SELECT DATEADD(DAY, -1, EOMONTH(GETDATE())) AS SecondtoLastDayofCurrentMonth
```

**How it works:**

```sql
SELECT
    CAST(GETDATE() AS DATE)              AS TodayGetDate                   -- 2024-12-02
  , EOMONTH(GETDATE())                   AS Step1                          -- 2024-12-31
  , DATEADD(DAY, -1, EOMONTH(GETDATE())) AS SecondtoLastDayofCurrentMonth  -- 2024-12-30
```

```
TodayGetDate  Step1       SecondtoLastDayofCurrentMonth
2024-12-02    2024-12-31  2024-12-30
```

`EOMONTH()` returns the last day, then `DATEADD(DAY, -1, ...)` subtracts 1 day.

---

### Solution 3.2 — Using `DATEADD()` and `DAY()` (first-of-month then +1 month then -2 days)

```sql
SELECT CAST(DATEADD(DAY, -2, DATEADD(MONTH, 1, DATEADD(DAY, 1 - (DAY(GETDATE())), GETDATE()))) AS DATE) AS SecondtoLastDayofCurrentMonth
```

**How it works step by step:**

```sql
SELECT
    CAST(GETDATE() AS DATE)                                                                          AS TodayGetDate  -- 2024-12-02
  , DAY(GETDATE())                                                                                   AS Step1         -- 2
  , 1 - (DAY(GETDATE()))                                                                             AS Step2         -- -1
  , CAST(DATEADD(DAY, 1 - (DAY(GETDATE())), GETDATE()) AS DATE)                                      AS Step3         -- 2024-12-01
  , CAST(DATEADD(MONTH, 1, DATEADD(DAY, 1 - (DAY(GETDATE())), GETDATE())) AS DATE)                   AS Step4         -- 2025-01-01
  , CAST(DATEADD(DAY, -2, DATEADD(MONTH, 1, DATEADD(DAY, 1 - (DAY(GETDATE())), GETDATE()))) AS DATE) AS Step5         -- 2024-12-30
```

```
TodayGetDate  Step1  Step2  Step3       Step4       Step5_SecondtoLastDayofCurrentMonth
2024-12-02    2      -1     2024-12-01  2025-01-01  2024-12-30
```

Same logic as Solution 2.4 — but subtracting `2` days instead of `1` from the first of next month.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
