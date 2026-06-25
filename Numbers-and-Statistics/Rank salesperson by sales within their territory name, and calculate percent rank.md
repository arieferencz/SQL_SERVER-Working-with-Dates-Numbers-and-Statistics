# Rank salesperson by sales within their territory name, and calculate PERCENT_RANK

## 🎯 Exercise
Rank each salesperson by their total historical sales within their assigned territory — and calculate their percentile rank within that territory using `PERCENT_RANK()`.

---

## 💡 Solution

### Approach
We use two levels of queries — an inner subquery and an outer query:

- **Inner query:** Filters to rows where `SalesPersonID IS NOT NULL`, joins `SalesOrderHeader` to `SalesTerritory` to get the territory name, and uses `SUM(TotalDue)` grouped by territory and salesperson to calculate each salesperson's total sales within their territory.
- **Outer query:** Applies `RANK()` and `PERCENT_RANK()` both partitioned by territory name — producing a rank and a percentile rank for each salesperson within their specific territory.

### T-SQL functions and clauses used

| Function / Clause | Purpose |
|---|---|
| `WHERE SalesPersonID IS NOT NULL` | Excludes orders not associated with a specific salesperson |
| `SUM(TotalDue)` | Calculates total sales per salesperson per territory |
| `GROUP BY TerritoryName, SalesPersonID` | Groups to one row per salesperson-territory combination |
| `RANK() OVER (PARTITION BY TerritoryName ORDER BY Sales)` | Ranks salespeople within each territory — lowest sales = rank 1, resets for every territory |
| `PERCENT_RANK() OVER (PARTITION BY TerritoryName ORDER BY Sales)` | Calculates the relative percentile rank within each territory — returns a value between `0.00` and `1.00` |
| `ROUND(value, 2, 2)` | Rounds to 2 decimal places |
| `FORMAT(value, '#,#.##')` | Formats sales amounts with comma separators |
| `FORMAT(value, 'N2')` | Formats percentile rank as a decimal number with 2 decimal places |

### Tables used

| Schema | Table |
|---|---|
| `Sales` | `SalesOrderHeader` |
| `Sales` | `SalesTerritory` |

---

### T-SQL code

```sql
USE AdventureWorks2022;
GO

SELECT
    X.TerritoryName
	, FORMAT(ROUND(X.Sales, 2, 2), '#,#.##') AS Sales
	, RANK() OVER (PARTITION BY X.TerritoryName ORDER BY X.Sales) AS SalesRank
	, X.SalesPersonID
	, FORMAT(ROUND(PERCENT_RANK() OVER (PARTITION BY X.TerritoryName ORDER BY X.Sales), 2, 2), 'N2') AS SalesPERCENTRANK
FROM (
    SELECT
		SalesTerritory.[Name] AS TerritoryName
		, SalesOrderHeader.[SalesPersonID] AS SalesPersonID
		, SUM(SalesOrderHeader.[TotalDue]) AS Sales
    FROM [AdventureWorks2022].[Sales].[SalesOrderHeader] AS SalesOrderHeader
    INNER JOIN [AdventureWorks2022].[Sales].[SalesTerritory] AS SalesTerritory
        ON SalesOrderHeader.TerritoryID = SalesTerritory.TerritoryID
    WHERE [SalesPersonID] IS NOT NULL
    GROUP BY SalesTerritory.[Name], SalesOrderHeader.[SalesPersonID]
) AS X
```

---

### Output
```
TerritoryName    Sales           SalesRank  SalesPersonID  SalesPERCENTRANK
Australia        195,528.78      1          285            0.00
Australia        1,606,441.44    2          286            1.00
Canada           204,459.6       1          274            0.00
Canada           2,354,403.76    2          282            0.33
Canada           4,069,422.21    3          278            0.66
Canada           9,585,124.94    4          289            1.00
Central          39,933.18       1          274            0.00
Central          582,372.21      2          281            0.25
Central          2,077,158.62    3          276            0.50
Central          2,899,017.45    4          275            0.75
Central          3,311,501.85    5          277            1.00
France           92,023.67       1          287            0.00
France           1,474,523.77    2          282            1.00
Germany          122,748.27      1          287            0.00
Germany          1,095,576.37    2          284            1.00
Northeast        89,680.88       1          283            0.00
Northeast        1,113,126.02    2          276            0.33
Northeast        2,161,095.23    3          280            0.66
Northeast        3,951,851.35    4          278            1.00
Northwest        303,793.47      1          278            0.00
Northwest        1,036,621.48    2          286            0.14
Northwest        1,041,261.02    3          275            0.28
Northwest        1,169,819.74    4          276            0.42
Northwest        1,229,060.02    5          280            0.57
Northwest        1,523,767.68    6          281            0.71
Northwest        6,199,819.33    7          277            1.00
Pacific          182,783.29      1          283            0.00
Pacific          3,141,875.93    2          279            1.00
Southeast        88,698.08       1          283            0.00
Southeast        1,453,553.68    2          279            1.00
Southwest        475,761.68      1          282            0.00
Southwest        680,791.46      2          283            0.14
Southwest        727,840.49      3          279            0.28
Southwest        1,017,419.08    4          281            0.42
Southwest        1,085,617.71    5          280            0.57
Southwest        1,212,036.05    6          275            0.71
Southwest        7,386,640.17    7          274            1.00
United Kingdom   497,073.73      1          287            0.00
United Kingdom   4,329,132.89    2          282            1.00
(39 rows affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — Calculate total sales per salesperson per territory (inner query)

We join `SalesOrderHeader` to `SalesTerritory` on `TerritoryID` to get the territory name for each order. We filter with `WHERE SalesPersonID IS NOT NULL` to exclude non-salesperson orders. `GROUP BY TerritoryName, SalesPersonID` and `SUM(TotalDue)` produce one row per salesperson-territory combination.

**Output (truncated):** 39 rows — one per unique salesperson-territory pair.

```
TerritoryName  SalesPersonID  Sales
Australia      285            195,528.78
Australia      286            1,606,441.44
Canada         274            204,459.60
Canada         278            4,069,422.21
Canada         282            2,354,403.76
Canada         289            9,585,124.94
...
(39 rows affected)
```

---

### Query 1.2 — Apply `RANK()` and `PERCENT_RANK()` partitioned by territory

Both window functions use `PARTITION BY TerritoryName` — creating one independent partition per territory. Within each partition rows are ordered by `Sales` ascending (lowest sales = rank 1).

**How the ranking resets for each territory:**

```
TerritoryName  Sales           RANK  PERCENT_RANK
Australia      195,528.78      1     0.00   ← rank resets for Australia
Australia      1,606,441.44    2     1.00
Canada         204,459.60      1     0.00   ← rank resets for Canada
Canada         2,354,403.76    2     0.33
Canada         4,069,422.21    3     0.66
Canada         9,585,124.94    4     1.00
Central        39,933.18       1     0.00   ← rank resets for Central
...
```

---

## 📝 Understanding `PERCENT_RANK()`

`PERCENT_RANK()` is a window function that calculates the **relative percentile rank** of each row within its partition. It returns a value between `0.00` and `1.00` inclusive — representing where the row stands relative to all other rows in the same partition.

### Formula

```
               Rank − 1
PERCENT_RANK = ─────────────
               Total Rows − 1
```

Where `Rank` is the standard rank of the row (identical to `RANK()`) and `Total Rows` is the number of rows in the partition.

### How the formula works — example for Canada (4 salespeople)

| SalesPersonID | Sales | Rank | Formula | PERCENT_RANK |
|---|---|---|---|---|
| 274 | 204,459.60 | 1 | (1−1) / (4−1) = 0/3 | 0.00 |
| 282 | 2,354,403.76 | 2 | (2−1) / (4−1) = 1/3 | 0.33 |
| 278 | 4,069,422.21 | 3 | (3−1) / (4−1) = 2/3 | 0.66 |
| 289 | 9,585,124.94 | 4 | (4−1) / (4−1) = 3/3 | 1.00 |

- The **first row** always returns `0.00` — no salespeople perform lower within the territory
- The **last row** always returns `1.00` — this salesperson outperforms everyone else in the territory
- Intermediate rows receive fractional values reflecting their relative position

### How the formula works — example for Central (5 salespeople)

| SalesPersonID | Sales | Rank | Formula | PERCENT_RANK |
|---|---|---|---|---|
| 274 | 39,933.18 | 1 | (1−1) / (5−1) = 0/4 | 0.00 |
| 281 | 582,372.21 | 2 | (2−1) / (5−1) = 1/4 | 0.25 |
| 276 | 2,077,158.62 | 3 | (3−1) / (5−1) = 2/4 | 0.50 |
| 275 | 2,899,017.45 | 4 | (4−1) / (5−1) = 3/4 | 0.75 |
| 277 | 3,311,501.85 | 5 | (5−1) / (5−1) = 4/4 | 1.00 |

---

### Key technical limitations of `PERCENT_RANK()`

| Limitation | Description |
|---|---|
| No frame clauses | Cannot be combined with `ROWS BETWEEN` or `RANGE BETWEEN` window frame filters |
| `NULL` handling | SQL Server treats `NULL` values as the lowest possible rank — filter them upstream with `WHERE` if they affect accuracy |
| Nondeterministic with ties | If two rows share the same sales amount (tied rank), consecutive runs may yield different ordering if no unique tie-breaker column exists in the `ORDER BY` |

---

### Key observations from the output

**Territories with only 2 salespeople** (Australia, France, Germany, Pacific, Southeast, United Kingdom) always produce `PERCENT_RANK` values of exactly `0.00` and `1.00` — since with only 2 rows the formula yields `0/1` and `1/1`.

**Northwest** has the most salespeople of any territory (**7**) — producing the most granular `PERCENT_RANK` distribution (`0.00`, `0.14`, `0.28`, `0.42`, `0.57`, `0.71`, `1.00`).

**Top performer per territory (PERCENT_RANK = 1.00):**

| Territory | SalesPersonID | Total Sales |
|---|---|---|
| Australia | 286 | $1,606,441.44 |
| Canada | 289 | $9,585,124.94 |
| Central | 277 | $3,311,501.85 |
| Northwest | 277 | $6,199,819.33 |
| Southwest | 274 | $7,386,640.17 |

---

### `RANK()` vs `PERCENT_RANK()` — when to use each

| Function | Output | Use when |
|---|---|---|
| `RANK()` | Integer (1, 2, 3...) | You need an absolute position within the group |
| `PERCENT_RANK()` | Decimal between 0.00 and 1.00 | You need a relative position — useful for comparing across groups of different sizes |

`PERCENT_RANK()` is particularly useful when comparing salespeople across territories of **different sizes** — a rank of `3` means something different in a territory with 4 salespeople vs one with 7. A `PERCENT_RANK` of `0.66` means the same thing regardless of territory size: the salesperson outperforms 66% of their territory peers.

---

### Related exercises

This exercise combines ranking techniques from:
- [Rank the product names based on total sales](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/Numbers-and-Statistics/Rank%20the%20product%20names%20based%20on%20total%20sales.md) — global `RANK()` without `PARTITION BY`
- [Rank salespeople by monthly sales, resetting the rank every month](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/Numbers-and-Statistics/Rank%20salespeople%20by%20monthly%20sales%2C%20resetting%20the%20rank%20every%20month.md) — `RANK()` with `PARTITION BY`
- [Percentile calculations using PERCENT_RANK() and CUME_DIST()](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/Numbers-and-Statistics/Percentile%20calculations%20using%20PERCENT_RANK()%20and%20CUME_DIST().md) — deep dive into `PERCENT_RANK()` and `CUME_DIST()`

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
