# Mode calculations using DENSE_RANK()

## 🎯 Exercise
Calculate the mode of product sales — identifying which products have been sold the most units, both historically (all years combined) and broken down by year.

---

## 📝 Note

> In statistics, the **mode** is the value that appears most frequently in a dataset. In this context, the "mode" is the product that has been ordered the most units across all sales orders.

---

## 💡 Exercise 1 — Mode: most sold product historically (all years)

### Approach
We join three tables to retrieve each product's total units sold across all orders. We then use `DENSE_RANK()` ordered by total units sold descending to rank products — rank `1` identifies the mode (the most sold product).

### T-SQL functions used

| Function | Purpose |
|---|---|
| `SUM(OrderQty)` | Calculates the total units sold per product across all orders |
| `DENSE_RANK() OVER (ORDER BY SUMOrderQty DESC)` | Assigns a rank to each product by total units sold — rank 1 = most sold |
| `GROUP BY ProductID, ProductName` | Groups sales by product to calculate totals |

### Tables used

| Schema | Table |
|---|---|
| `Sales` | `SalesOrderHeader` |
| `Sales` | `SalesOrderDetail` |
| `Production` | `Product` |

---

### T-SQL code

```sql
SELECT
    Y.ProductID
  , Y.ProductName
  , Y.SUMOrderQty
  , DENSE_RANK() OVER (ORDER BY Y.SUMOrderQty DESC) AS DenseRank
FROM (
    SELECT
        X.ProductID
      , X.ProductName   AS ProductName
      , SUM(X.OrderQty) AS SUMOrderQty
    FROM (
        SELECT
            SalesOrderHeader.SalesOrderID
          , [Product].[Name] AS ProductName
          , SalesOrderDetail.ProductID
          , SalesOrderDetail.OrderQty
        FROM [AdventureWorks2022].[Sales].[SalesOrderHeader] AS SalesOrderHeader
        LEFT JOIN [AdventureWorks2022].[Sales].[SalesOrderDetail] AS SalesOrderDetail
            ON SalesOrderHeader.SalesOrderID = SalesOrderDetail.SalesOrderID
        LEFT JOIN [AdventureWorks2022].[Production].[Product] AS [Product]
            ON SalesOrderDetail.ProductID = [Product].ProductID
    ) AS X
    GROUP BY X.ProductID, X.ProductName
) AS Y
```

---

### Output (truncated)

```
ProductID  ProductName                      SUMOrderQty  DenseRank
712        AWC Logo Cap                     8311         1   ← MODE (most sold product)
870        Water Bottle - 30 oz.            6815         2
711        Sport-100 Helmet, Blue           6743         3
715        Long-Sleeve Logo Jersey, L       6592         4
708        Sport-100 Helmet, Black          6532         5
...
927        LL Mountain Frame - Black, 52    15           241
898        LL Touring Frame - Blue, 62      15           241
911        LL Road Seat/Saddle              10           242
943        LL Mountain Frame - Black, 40    8            243
942        ML Mountain Frame-W - Silver, 38 7            244
897        LL Touring Frame - Blue, 58      4            245
(266 rows affected)
```

---

## 🔍 Step-by-step explanation — Exercise 1

### Query 1.1 — Retrieve raw sales data (`OriginalTablesLevel1`)
We join `SalesOrderHeader`, `SalesOrderDetail`, and `Product` to get each order line with its product name and quantity ordered.

**Output:** 121,317 rows — one per order line.

```
SalesOrderID  ProductName               ProductID  OrderQty
43659         Mountain Bike Socks, M    709        6
43659         Sport-100 Helmet, Black   708        1
43659         Road-650 Red, 44          773        2
...
(121317 rows affected)
```

---

### Query 1.2 — Sum units sold per product (`SumUnitsSoldPerProductIDLevel2`)
`GROUP BY ProductID, ProductName` and `SUM(OrderQty)` collapses 121,317 order lines into 266 rows — one per unique product — with the total units sold.

**Output (truncated):** 266 rows — one per product.

```
ProductID  ProductName                SUMOrderQty
707        Sport-100 Helmet, Red      4384
708        Sport-100 Helmet, Black    6532
709        Mountain Bike Socks, M     3680
712        AWC Logo Cap               8311
...
(266 rows affected)
```

---

### Final Query (Query 1.3) — Rank products using `DENSE_RANK()`
`DENSE_RANK() OVER (ORDER BY SUMOrderQty DESC)` assigns rank `1` to the product with the highest total units sold. Unlike `RANK()`, `DENSE_RANK()` does not skip rank numbers when two products tie — so if two products share rank 241, the next rank is 242 (not 243).

**The mode is `ProductID = 712` (AWC Logo Cap) with 8,311 total units sold.**

---

## 💡 Exercise 2 — Mode: most sold product per year

### Approach
We add `DATEPART(YEAR, OrderDate)` to the base query to extract the sales year for each order. We group by product and year, then use `DENSE_RANK()` ordered by year and total units sold descending to rank products within each year.

---

### T-SQL code

```sql
SELECT
    Y.[Year]
  , Y.ProductID
  , Y.ProductName
  , Y.SUMOrderQty
  , DENSE_RANK() OVER (ORDER BY Y.[Year], Y.SUMOrderQty DESC) AS DenseRank
FROM (
    SELECT
        X.ProductID
      , X.ProductName   AS ProductName
      , SUM(X.OrderQty) AS SUMOrderQty
      , X.SalesYear     AS [Year]
    FROM (
        SELECT
            SalesOrderHeader.SalesOrderID
          , DATEPART(YEAR, SalesOrderHeader.OrderDate) AS SalesYear
          , [Product].[Name]                           AS ProductName
          , SalesOrderDetail.ProductID
          , SalesOrderDetail.OrderQty
        FROM [AdventureWorks2022].[Sales].[SalesOrderHeader] AS SalesOrderHeader
        LEFT JOIN [AdventureWorks2022].[Sales].[SalesOrderDetail] AS SalesOrderDetail
            ON SalesOrderHeader.SalesOrderID = SalesOrderDetail.SalesOrderID
        LEFT JOIN [AdventureWorks2022].[Production].[Product] AS [Product]
            ON SalesOrderDetail.ProductID = [Product].ProductID
    ) AS X
    GROUP BY X.ProductID, X.ProductName, X.SalesYear
) AS Y
```

---

### Output (truncated)

```
Year  ProductID  ProductName                       SUMOrderQty  DenseRank
2011  709        Mountain Bike Socks, M             608          1   ← 2011 MODE
2011  712        AWC Logo Cap                       545          2
2011  715        Long-Sleeve Logo Jersey, L         544          3
2011  770        Road-650 Black, 52                 415          4
...
2011  723        LL Road Frame - Black, 60          1            52
2012  863        Full-Finger Gloves, L              2380         53  ← 2012 MODE
2012  715        Long-Sleeve Logo Jersey, L         2113         54
2012  712        AWC Logo Cap                       2048         55
...
2012  744        HL Mountain Frame - Black, 44      8            177
2013  870        Water Bottle - 30 oz.              3913         178 ← 2013 MODE
2013  712        AWC Logo Cap                       3768         179
2013  708        Sport-100 Helmet, Black            3088         180
...
2013  828        HL Road Rear Wheel                 2            385
2014  870        Water Bottle - 30 oz.              2902         386 ← 2014 MODE
2014  712        AWC Logo Cap                       1950         387
2014  711        Sport-100 Helmet, Blue             1776         388
...
2014  763        Road-650 Red, 48                   1            531
2014  765        Road-650 Black, 58                 1            531
2014  726        LL Road Frame - Red, 48            1            531
2014  730        LL Road Frame - Red, 62            1            531
(610 rows affected)
```

---

## 🔍 Key observations

| Year | Mode (most sold product) | Units sold |
|---|---|---|
| 2011 | Mountain Bike Socks, M (ProductID 709) | 608 |
| 2012 | Full-Finger Gloves, L (ProductID 863) | 2,380 |
| 2013 | Water Bottle - 30 oz. (ProductID 870) | 3,913 |
| 2014 | Water Bottle - 30 oz. (ProductID 870) | 2,902 |
| All years | AWC Logo Cap (ProductID 712) | 8,311 |

**Key insight:** The overall historical mode (AWC Logo Cap) is different from any single year's mode. This is because the AWC Logo Cap consistently ranks near the top every year but never leads in any individual year — its cumulative total across all years surpasses all other products.

### `DENSE_RANK()` vs `RANK()`

| Function | Behaviour on ties | Example |
|---|---|---|
| `DENSE_RANK()` | No gaps — tied rows share a rank and the next rank continues sequentially | 1, 2, 2, 3, 4 |
| `RANK()` | Gaps — tied rows share a rank but the next rank skips the tied positions | 1, 2, 2, 4, 5 |

`DENSE_RANK()` is preferred here because it preserves a continuous ranking sequence — making it easier to filter to a specific rank (e.g. `WHERE DenseRank = 1` to get only the mode).

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
