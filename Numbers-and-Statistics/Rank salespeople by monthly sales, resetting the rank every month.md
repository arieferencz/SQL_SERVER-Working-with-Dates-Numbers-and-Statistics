# Rank salespeople by monthly sales, resetting the rank every month

## 🎯 Exercise
Rank all salespeople by their total sales amount for each month — resetting the ranking at the start of every new month so that rank 1 always identifies the lowest-performing salesperson within that specific month.

---

## 💡 Solution

### Approach
We use two levels of queries — an inner subquery and an outer query:

- **Inner query:** Filters to rows where `SalesPersonID IS NOT NULL` (excluding non-salesperson orders), extracts the year and month from `OrderDate`, and uses `SUM(TotalDue)` grouped by salesperson, year, and month to calculate each salesperson's total sales per month.
- **Outer query:** Applies `RANK() OVER (PARTITION BY YearSale, MonthSale ORDER BY MonthlySales)` to rank salespeople within each month — the rank resets at 1 for every new year-month combination.

### T-SQL functions and clauses used

| Function / Clause | Purpose |
|---|---|
| `WHERE SalesPersonID IS NOT NULL` | Excludes orders not associated with a specific salesperson |
| `YEAR(OrderDate)` | Extracts the year from the order date |
| `MONTH(OrderDate)` | Extracts the month number from the order date |
| `SUM(TotalDue)` | Calculates total sales per salesperson per month |
| `GROUP BY SalesPersonID, YEAR(OrderDate), MONTH(OrderDate)` | Groups to one row per salesperson-year-month combination |
| `RANK() OVER (PARTITION BY YearSale, MonthSale ORDER BY MonthlySales)` | Ranks salespeople within each month — lowest sales = rank 1, rank resets for every new month |
| `ROUND(value, 2, 2)` | Rounds to 2 decimal places |
| `FORMAT(value, '#,#.##')` | Formats with comma thousands separators and 2 decimal places |

### Table used

| Schema | Table |
|---|---|
| `Sales` | `SalesOrderHeader` |

---

### T-SQL code

```sql
USE AdventureWorks2022;
GO

SELECT
    X.[SalesPersonID]
  , X.YearSale
  , X.MonthSale
  , RANK() OVER (
        PARTITION BY X.YearSale, X.MonthSale
        ORDER BY X.MonthlySales)                  AS SalesPersonByMonthlySalesRank
  , FORMAT(ROUND(X.MonthlySales, 2, 2), '#,#.##') AS MonthlySales
FROM (
    SELECT
        [SalesPersonID]
      , YEAR([OrderDate])  AS YearSale
      , MONTH([OrderDate]) AS MonthSale
      , SUM([TotalDue])    AS MonthlySales
    FROM [AdventureWorks2022].[Sales].[SalesOrderHeader]
    WHERE [SalesPersonID] IS NOT NULL
    GROUP BY [SalesPersonID], YEAR([OrderDate]), MONTH([OrderDate])
) AS X
```

---

### Output (truncated)

```
SalesPersonID  YearSale  MonthSale  SalesPersonByMonthlySalesRank  MonthlySales
276            2011      5          1                              6,167.16
278            2011      5          2                              10,254.85
280            2011      5          3                              27,510.41
277            2011      5          4                              52,586.67
281            2011      5          5                              67,231.71
275            2011      5          6                              71,792.84
283            2011      5          7                              78,223.3
279            2011      5          8                              117,577.6
282            2011      5          9                              119,678.92
274            2011      7          1                              23,130.29
281            2011      7          2                              38,387.53
283            2011      7          3                              73,343.36
275            2011      7          4                              122,246.74
280            2011      7          5                              172,367.99
278            2011      7          6                              176,300.29
282            2011      7          7                              180,684.84
277            2011      7          8                              300,998.15
276            2011      7          9                              305,145.22
279            2011      7          10                             340,236.6
...
288            2014      4          1                              1,428.61
285            2014      5          1                              4,725.95
274            2014      5          2                              42,546.92
290            2014      5          3                              134,318.85
288            2014      5          4                              152,878.6
283            2014      5          5                              159,928.17
284            2014      5          6                              161,583.98
278            2014      5          7                              163,799.47
280            2014      5          8                              170,556.7
281            2014      5          9                              248,058.19
286            2014      5          10                             255,981.92
279            2014      5          11                             262,264.66
276            2014      5          12                             319,181.39
277            2014      5          13                             380,780.5
275            2014      5          14                             417,208.47
282            2014      5          15                             480,379.23
289            2014      5          16                             495,918.61
(423 rows affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — Calculate monthly sales per salesperson (inner query)

We filter `SalesOrderHeader` to rows where `SalesPersonID IS NOT NULL` — excluding orders placed through non-salesperson channels (e.g. online orders). We then extract the year and month from `OrderDate` and use `SUM(TotalDue)` grouped by salesperson, year, and month to produce one row per salesperson-month combination.

**Output (truncated):** 423 rows — one per salesperson-year-month combination.

```
SalesPersonID  YearSale  MonthSale  MonthlySales
274            2011      5          71792.84
275            2011      5          71792.84
276            2011      5          6167.16
277            2011      5          52586.67
...
289            2014      5          495918.61
290            2014      5          134318.85
(423 rows affected)
```

> **Note:** Not every salesperson appears in every month. The database contains sales data from **2011 to 2014**, with some months having only 1 salesperson and others having up to 17.

---

### Query 1.2 — Apply `RANK()` partitioned by year and month (outer query)

`RANK() OVER (PARTITION BY YearSale, MonthSale ORDER BY MonthlySales)` creates a separate ranking for each unique year-month combination. Within each partition, salespeople are ranked from lowest to highest monthly sales — rank `1` is the lowest performer for that month.

**How `PARTITION BY` resets the rank every month:**

```
YearSale  MonthSale  SalesPersonID  MonthlySales  Rank
2011      5          276            6,167.16      1     ← rank resets for May 2011
2011      5          278            10,254.85     2
2011      5          280            27,510.41     3
...
2011      5          282            119,678.92    9     ← 9 salespeople in May 2011
2011      7          274            23,130.29     1     ← rank resets for July 2011
2011      7          281            38,387.53     2
...
2011      7          279            340,236.6     10    ← 10 salespeople in July 2011
2012      1          ...            ...           1     ← rank resets for Jan 2012
```

Each new year-month pair is treated as an independent partition — so rank `1` always identifies the lowest-performing salesperson **within that specific month**, regardless of how they compare to other months.

---

### Key observations from the output

**Number of salespeople per month varies significantly:**
- **May 2011** had only **9 salespeople** active
- **May 2014** had **17 salespeople** active — the largest group in the dataset
- **April 2014** had only **1 salesperson** (`SalesPersonID = 288`, `$1,428.61`)

**Rank 1 (lowest monthly performer) across selected months:**

| Year | Month | SalesPersonID | MonthlySales |
|---|---|---|---|
| 2011 | 5 | 276 | $6,167.16 |
| 2011 | 7 | 274 | $23,130.29 |
| 2014 | 4 | 288 | $1,428.61 |
| 2014 | 5 | 285 | $4,725.95 |

**Top performer (highest rank) in May 2014:**
`SalesPersonID = 289` ranked 16th (highest) with `$495,918.61` — nearly 100× more than the lowest performer in the same month (`SalesPersonID = 285` with `$4,725.95`).

---

### Why `ORDER BY MonthlySales ASC` (lowest = rank 1)?

The `ORDER BY MonthlySales` in the `RANK()` clause uses **ascending** order by default — so rank `1` goes to the salesperson with the **lowest** monthly sales. To rank by highest sales first (rank 1 = top performer), use `ORDER BY MonthlySales DESC`:

```sql
RANK() OVER (
    PARTITION BY X.YearSale, X.MonthSale
    ORDER BY X.MonthlySales DESC)   -- highest sales = rank 1
```

---

### How `PARTITION BY` with two columns drives the monthly reset

`PARTITION BY YearSale, MonthSale` creates one independent partition for every unique **year-month combination**. This means:

- May 2011 (`2011, 5`) is one partition → rank resets
- July 2011 (`2011, 7`) is a different partition → rank resets
- May 2012 (`2012, 5`) is yet another partition → rank resets

Without `PARTITION BY`, all 423 rows would form a single partition — producing a global ranking from 1 to 423 across all salespeople and all months combined.

---

### Related exercises

This exercise applies the same `RANK()` function used in:
- [Rank the product names based on total sales](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/Numbers-and-Statistics/Rank%20the%20product%20names%20based%20on%20total%20sales.md) — ranking without `PARTITION BY` (global ranking)

The key difference is the addition of `PARTITION BY YearSale, MonthSale` — which transforms a global ranking into a per-month ranking that resets every month.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
