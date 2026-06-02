# Calculate the difference between current row and previous row's sales amount by product

## 🎯 Exercise
For each product, calculate the difference between its total sales amount and the total sales amount of the previous product — when all products are ordered from lowest to highest total sales.

---

## 💡 Solution

### Approach
We use two levels of queries — an inner subquery and an outer query:

- **Inner query:** Joins three tables and uses `SUM(LineTotal)` grouped by product name and product ID to calculate the total sales amount per product.
- **Outer query:** Applies `LAG(SalesByProductName) OVER (ORDER BY SalesByProductName)` to retrieve the previous row's sales amount, then subtracts it from the current row's amount to produce the difference column `SalesDiff`.

### T-SQL functions and clauses used

| Function / Clause | Purpose |
|---|---|
| `SUM(LineTotal)` | Calculates the total sales amount per product across all order lines |
| `GROUP BY Product.[Name], Product.[ProductID]` | Groups results to one row per product |
| `LAG(SalesByProductName) OVER (ORDER BY SalesByProductName)` | Returns the sales amount of the previous row in the ascending sales order — `NULL` for the first row |
| `SalesByProductName - LAG(...)` | Subtracts the previous row's amount from the current row's amount — producing the difference |
| `ROUND(value, 2, 2)` | Rounds to 2 decimal places |
| `FORMAT(value, '#,#.##')` | Formats with comma thousands separators and 2 decimal places |
| `ORDER BY SalesByProductName ASC` | Orders products from lowest to highest total sales |

### Tables used

| Schema | Table |
|---|---|
| `Sales` | `SalesOrderHeader` |
| `Sales` | `SalesOrderDetail` |
| `Production` | `Product` |

---

### T-SQL code

```sql
USE AdventureWorks2022;
GO

SELECT
    X.ProductName
  , FORMAT(ROUND(X.SalesByProductName, 2, 2), '#,#.##')                AS SalesByProductName
  , FORMAT(ROUND(X.SalesByProductName
        - LAG(X.SalesByProductName) OVER (ORDER BY X.SalesByProductName),
        2, 2), '#,#.##')                                                AS SalesDiff
FROM (
    SELECT
        Product.[Name]       AS ProductName
      , Product.[ProductID]  AS ProductID
      , SUM(SalesOrderDetail.[LineTotal]) AS SalesByProductName
    FROM [AdventureWorks2022].[Sales].[SalesOrderHeader] AS SalesOrderHeader
    INNER JOIN [AdventureWorks2022].[Sales].[SalesOrderDetail] AS SalesOrderDetail
        ON SalesOrderHeader.[SalesOrderID] = SalesOrderDetail.[SalesOrderID]
    INNER JOIN [AdventureWorks2022].[Production].[Product] AS Product
        ON SalesOrderDetail.[ProductID] = Product.[ProductID]
    GROUP BY Product.[Name], Product.[ProductID]
) AS X
ORDER BY X.SalesByProductName ASC
```

---

### Output (truncated)

```
ProductName                       SalesByProductName  SalesDiff
LL Road Seat/Saddle               162.72              NULL
Mountain Bike Socks, L            513                 350.28
LL Touring Frame - Blue, 58       800.2               287.2
LL Mountain Frame - Black, 40     1,198.99            398.78
LL Touring Seat/Saddle            1,480.75            281.76
ML Mountain Frame-W - Silver, 38  1,529.17            48.42
LL Touring Handlebars             1,548.62            19.44
LL Headset                        1,949.4             400.77
...
Mountain-200 Black, 46            3,309,673.21        792,815.9
Mountain-200 Silver, 46           3,434,256.94        124,583.72
Mountain-200 Silver, 42           3,438,478.86        4,221.91
Mountain-200 Silver, 38           3,693,678.02        255,199.16
Mountain-200 Black, 42            4,009,494.76        315,816.73
Mountain-200 Black, 38            4,400,592.8         391,098.03
(266 rows affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — Calculate total sales per product (inner query)

The inner query is identical to the one used in [Calculate total sales by product name](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/Numbers-and-Statistics/Calculate%20total%20sales%20by%20product%20name.md) and [Rank the product names based on total sales](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/Numbers-and-Statistics/Rank%20the%20product%20names%20based%20on%20total%20sales.md). It joins three tables and uses `SUM(LineTotal)` grouped by product to produce 266 rows — one per product with its total revenue.

**Output (truncated):** 266 rows ordered by `SalesByProductName ASC`.

```
ProductName                   SalesByProductName
LL Road Seat/Saddle           162.72
Mountain Bike Socks, L        513
LL Touring Frame - Blue, 58   800.2
...
Mountain-200 Black, 38        4,400,592.80
(266 rows affected)
```

---

### Query 1.2 — Apply `LAG()` to retrieve the previous row's sales amount

`LAG(SalesByProductName) OVER (ORDER BY SalesByProductName)` retrieves the `SalesByProductName` value from the **previous row** in the ascending sales order. For the first row there is no previous row — so `LAG()` returns `NULL`, which is why `SalesDiff` is `NULL` for `LL Road Seat/Saddle`.

**How `LAG()` works row by row:**

```
Row  ProductName                   SalesByProductName  LAG value    SalesDiff
1    LL Road Seat/Saddle            162.72              NULL         NULL
2    Mountain Bike Socks, L         513.00              162.72       350.28
3    LL Touring Frame - Blue, 58    800.20              513.00       287.20
4    LL Mountain Frame - Black, 40  1,198.99            800.20       398.78
5    LL Touring Seat/Saddle         1,480.75            1,198.99     281.76
...
265  Mountain-200 Black, 42         4,009,494.76        3,693,678.02 315,816.73
266  Mountain-200 Black, 38         4,400,592.80        4,009,494.76 391,098.03
```

The `SalesDiff` column shows how much larger each product's revenue is compared to the next lower-performing product — providing a measure of the revenue gap between consecutive products in the ranking.

---

### Key observations from the output

**Smallest gaps (products with very similar sales):**

| ProductName | SalesByProductName | SalesDiff |
|---|---|---|
| ML Mountain Frame-W - Silver, 38 | 1,529.17 | 48.42 |
| LL Touring Handlebars | 1,548.62 | 19.44 |
| Mountain-200 Silver, 42 | 3,438,478.86 | 4,221.91 |

**Largest gaps (biggest jumps between consecutive products):**

| ProductName | SalesByProductName | SalesDiff |
|---|---|---|
| Mountain-200 Black, 46 | 3,309,673.21 | 792,815.90 |
| Mountain-200 Silver, 38 | 3,693,678.02 | 255,199.16 |
| Mountain-200 Black, 38 | 4,400,592.80 | 391,098.03 |

The largest single gap (`$792,815.90`) occurs between `Mountain-200 Black, 46` and the product ranked just below it — indicating a significant revenue tier separation at the top of the product range.

---

### `LAG()` vs `LEAD()`

| Function | Direction | Returns |
|---|---|---|
| `LAG(column) OVER (ORDER BY ...)` | Backward | Value from the **previous** row |
| `LEAD(column) OVER (ORDER BY ...)` | Forward | Value from the **next** row |

To calculate the difference between the current row and the **next** row (instead of the previous), replace `LAG` with `LEAD`:

```sql
FORMAT(ROUND(LEAD(X.SalesByProductName) OVER (ORDER BY X.SalesByProductName)
    - X.SalesByProductName, 2, 2), '#,#.##') AS SalesDiffToNext
```

This would show how much more the next product earns compared to the current one — the last row would return `NULL` since there is no next row.

---

### Related exercises

This exercise builds on the total sales calculation from:
- [Calculate total sales by product name](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/Numbers-and-Statistics/Calculate%20total%20sales%20by%20product%20name.md)
- [Rank the product names based on total sales](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/Numbers-and-Statistics/Rank%20the%20product%20names%20based%20on%20total%20sales.md)

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
