# Rank the product names based on total sales

## 🎯 Exercise
Rank all products by their total historical sales amount — from highest to lowest — with ties handled correctly using `RANK()`.

---

## 💡 Solution

### Approach
We use two levels of queries — an inner subquery and an outer query:

- **Inner query:** Joins three tables and uses `SUM(LineTotal)` grouped by product name and product ID to calculate the total sales amount per product.
- **Outer query:** Applies `RANK() OVER (ORDER BY SalesByProductName DESC)` to assign a rank to each product based on its total sales — highest sales gets rank 1. `FORMAT()` and `ROUND()` format the amounts for readability.

### T-SQL functions and clauses used

| Function / Clause | Purpose |
|---|---|
| `SUM(LineTotal)` | Calculates the total sales amount per product across all order lines |
| `GROUP BY Product.[Name], Product.[ProductID]` | Groups results to one row per product |
| `RANK() OVER (ORDER BY SalesByProductName DESC)` | Assigns a rank based on total sales — highest first. Tied products share a rank and the next rank skips the tied positions |
| `ROUND(value, 2, 2)` | Rounds to 2 decimal places |
| `FORMAT(value, '#,#.##')` | Formats with comma thousands separators and 2 decimal places |

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
	X.ProductID
	, X.ProductName
	, FORMAT(ROUND(X.SalesByProductName, 2, 2), '#,#.##') AS SalesByProductName
	, RANK() OVER (ORDER BY X.SalesByProductName DESC) AS SalesByProductNameRANK
FROM (
    SELECT
        Product.[Name] AS ProductName
      , Product.[ProductID] AS ProductID
      , SUM(SalesOrderDetail.[LineTotal]) AS SalesByProductName
    FROM [AdventureWorks2022].[Sales].[SalesOrderHeader] AS SalesOrderHeader
    INNER JOIN [AdventureWorks2022].[Sales].[SalesOrderDetail] AS SalesOrderDetail
        ON SalesOrderHeader.[SalesOrderID] = SalesOrderDetail.[SalesOrderID]
    INNER JOIN [AdventureWorks2022].[Production].[Product] AS Product
        ON SalesOrderDetail.[ProductID] = Product.[ProductID]
    GROUP BY Product.[Name], Product.[ProductID]
) AS X
```

---

### Output (truncated)

```
ProductID  ProductName                          SalesByProductName  SalesByProductNameRANK
782        Road-150 Red, 44                     4,400,592.8         1
783        Road-150 Red, 48                     4,009,494.76        2
779        Road-150 Red, 62                     3,693,678.02        3
780        Road-150 Red, 44                     3,438,478.86        4
781        Road-150 Red, 48                     3,434,256.94        5
784        Road-150 Red, 52                     3,309,673.21        6
793        Road-250 Red, 44                     2,516,857.31        7
794        Road-250 Red, 48                     2,347,655.95        8
795        Road-250 Red, 52                     2,012,447.77        9
753        Mountain-100 Silver, 38              1,847,818.62        10
...
914        LL Mountain Rear Wheel               1,480.75            262
943        LL Mountain Frame - Black, 40        1,198.99            263
897        LL Touring Frame - Blue, 58          800.2               264
710        Mountain Bike Socks, S               513                 265
911        LL Road Seat/Saddle                  162.72              266
(266 rows affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — Calculate total sales per product (inner query)

We join three tables to connect each order line to its product name. `GROUP BY Product.[Name], Product.[ProductID]` and `SUM(LineTotal)` collapse 121,317 order lines into 266 rows — one per product with its total revenue. This is the same aggregation used in the [Calculate total sales by product name](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/Numbers-and-Statistics/Calculate%20total%20sales%20by%20product%20name.md) exercise.

**Output (truncated):** 266 rows — one per product with total sales amount.

```
ProductName             	ProductID	SalesByProductName
Road-150 Red, 44        	782			4400592.799...
Road-150 Red, 48        	783			4009494.762...
Mountain-100 Silver, 38 	753			1847818.621...
AWC Logo Cap            	712			51229.44...
...
LL Road Seat/Saddle     	911			162.72
(266 rows affected)
```

---

### Query 1.2 — Apply `RANK()` to order by total sales (outer query)

`RANK() OVER (ORDER BY SalesByProductName DESC)` assigns a rank to each product — rank `1` goes to the product with the highest total sales. The results are ordered from highest to lowest revenue.

**How `RANK()` handles ties:**
If two products share the same total sales amount, they receive the same rank — and the next rank skips the number of tied positions. For example, if two products tie at rank 3, the next product receives rank 5 (not rank 4).

In this dataset there are no tied total sales amounts — each product has a unique total — so all 266 products receive a consecutive rank from 1 to 266.

---

### Key observations from the output

**Top 10 products by total sales:**

| Rank | ProductID | ProductName | Total Sales |
|---|---|---|---|
| 1 | 782 | Road-150 Red, 44 | $4,400,592.80 |
| 2 | 783 | Road-150 Red, 48 | $4,009,494.76 |
| 3 | 779 | Road-150 Red, 62 | $3,693,678.02 |
| 4 | 780 | Road-150 Red, 44 | $3,438,478.86 |
| 5 | 781 | Road-150 Red, 48 | $3,434,256.94 |
| 6 | 784 | Road-150 Red, 52 | $3,309,673.21 |
| 7 | 793 | Road-250 Red, 44 | $2,516,857.31 |
| 8 | 794 | Road-250 Red, 48 | $2,347,655.95 |
| 9 | 795 | Road-250 Red, 52 | $2,012,447.77 |
| 10 | 753 | Mountain-100 Silver, 38 | $1,847,818.62 |

**Key insight:** The top 6 ranked products are all variants of the **Road-150 Red** — different sizes of the same high-end road bike model. This suggests the Road-150 Red is the company's best-selling product line by revenue, with its various sizes each generating over $3 million in total sales.

**Bottom products by total sales:**
The lowest-ranked products are mostly accessories and components — `LL Road Seat/Saddle` at rank 266 generated only `$162.72` in total sales across all orders.

---

### `RANK()` vs `DENSE_RANK()` vs `ROW_NUMBER()`

| Function | Tie handling | Gap after ties? | Example (tie at 3rd) |
|---|---|---|---|
| `RANK()` | Tied rows share a rank | Yes — next rank skips | 1, 2, 3, 3, 5 |
| `DENSE_RANK()` | Tied rows share a rank | No — next rank continues | 1, 2, 3, 3, 4 |
| `ROW_NUMBER()` | No ties — every row gets a unique number | N/A | 1, 2, 3, 4, 5 |

`RANK()` is used here because it correctly reflects the relative standing when ties exist — a product tied at rank 3 should both be ranked 3rd, and the next product should be ranked 5th (skipping 4th) to accurately represent the gap.

---

### Related exercise

This exercise extends the total sales calculation from:
- [Calculate total sales by product name](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics/blob/main/Numbers-and-Statistics/Calculate%20total%20sales%20by%20product%20name.md) — the same aggregation without ranking

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
