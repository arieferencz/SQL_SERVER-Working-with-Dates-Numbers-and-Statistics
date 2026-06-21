# Running totals calculations

## 🎯 Exercise
Calculate cumulative running totals for sales order amounts at four levels of granularity:
1. By geographical group (Europe, North America, Pacific)
2. By country
3. By state or province (one total per state)
4. By state or province at individual order level

---

## 💡 Exercise 1 — Running total by geographical group

### Approach
We join `SalesOrderHeader` to `SalesTerritory` to get each order's geographical group. We sum amounts per group, then use `SUM() OVER (ORDER BY CountryGroup)` as a window function to accumulate the total alphabetically across groups.

### T-SQL functions used

| Function | Purpose |
|---|---|
| `SUM(Amount) OVER (ORDER BY CountryGroup)` | Calculates the cumulative running total ordered alphabetically by group name |
| `GROUP BY` | Aggregates total sales per geographical group |
| `FORMAT()` / `ROUND()` | Formats the result with comma separators and 2 decimal places |

### Tables used

| Schema | Table |
|---|---|
| `Sales` | `SalesOrderHeader` |
| `Sales` | `SalesTerritory` |

---

### T-SQL code

```sql
SELECT
    Y.CountryGroup
  , FORMAT(ROUND(Y.Amount, 2, 2), '#,#.##')                                     AS TotalSalesAmount
  , FORMAT(ROUND(SUM(Y.Amount) OVER (ORDER BY Y.CountryGroup), 2, 2), '#,#.##') AS RunningTotalSalesAmount
FROM (
    SELECT
        X.CountryGroupRegion AS CountryGroup
      , SUM(X.TotalDue)      AS Amount
    FROM (
        SELECT
            SalesOrderHeader.SalesOrderID
          , SalesOrderHeader.BillToAddressID
          , SalesOrderHeader.TerritoryID
          , SalesOrderHeader.TotalDue
          , SalesTerritory.[Group] AS CountryGroupRegion
        FROM [AdventureWorks2022].[Sales].[SalesOrderHeader] AS SalesOrderHeader
        LEFT JOIN [AdventureWorks2022].[Sales].[SalesTerritory] AS SalesTerritory
            ON SalesOrderHeader.TerritoryID = SalesTerritory.TerritoryID
    ) AS X
    GROUP BY X.CountryGroupRegion
) AS Y
GROUP BY Y.CountryGroup, Y.Amount
```

---

### Output

```
CountryGroup   TotalSalesAmount  RunningTotalSalesAmount
Europe         22,173,617.62     22,173,617.62
North America  89,228,792.39     111,402,410.02
Pacific        11,814,376.09     123,216,786.11
(3 rows affected)
```

---

## 💡 Exercise 2 — Running total by country

### Approach
We join `SalesOrderHeader` to `Address` and `StateProvince` to retrieve each order's country code. We sum amounts per country, then use `SUM() OVER (ORDER BY Country)` to accumulate alphabetically.

### Tables used

| Schema | Table |
|---|---|
| `Sales` | `SalesOrderHeader` |
| `Person` | `Address` |
| `Person` | `StateProvince` |

---

### T-SQL code

```sql
SELECT
    Y.Country
  , FORMAT(ROUND(Y.Amount, 2, 2), '#,#.##')                                AS TotalSalesAmount
  , FORMAT(ROUND(SUM(Y.Amount) OVER (ORDER BY Y.Country), 2, 2), '#,#.##') AS RunningTotalSalesAmount
FROM (
    SELECT
        X.CountryRegionCode AS Country
      , SUM(X.TotalDue)     AS Amount
    FROM (
        SELECT
            SalesOrderHeader.SalesOrderID
          , SalesOrderHeader.BillToAddressID
          , SalesOrderHeader.TerritoryID
          , SalesOrderHeader.TotalDue
          , [Address].AddressID
          , [Address].StateProvinceID
          , [StateProvince].CountryRegionCode
        FROM [AdventureWorks2022].[Sales].[SalesOrderHeader] AS SalesOrderHeader
        LEFT JOIN [AdventureWorks2022].[Person].[Address] AS [Address]
            ON SalesOrderHeader.ShipToAddressID = [Address].AddressID
        LEFT JOIN [AdventureWorks2022].[Person].[StateProvince] AS [StateProvince]
            ON [Address].StateProvinceID = [StateProvince].StateProvinceID
    ) AS X
    GROUP BY X.CountryRegionCode
) AS Y
GROUP BY Y.Country, Y.Amount
```

---

### Output

```
Country  TotalSalesAmount  RunningTotalSalesAmount
AU       11,814,376.09     11,814,376.09
CA       18,398,929.18     30,213,305.28
DE       5,479,819.57      35,693,124.85
FR       8,119,749.34      43,812,874.2
GB       8,574,048.7       52,386,922.91
US       70,829,863.2      123,216,786.11
(6 rows affected)
```

---

## 💡 Exercise 3 — Running total by state or province (one total per state)

### Approach
We join `SalesOrderHeader` to `Address` and `StateProvince` using `BillToAddressID` to get the billing state of each order. We sum amounts per state, then use `SUM() OVER (ORDER BY StateProvince)` to accumulate alphabetically across all 69 states and provinces.

### Tables used

| Schema | Table |
|---|---|
| `Sales` | `SalesOrderHeader` |
| `Person` | `Address` |
| `Person` | `StateProvince` |

---

### T-SQL code

```sql
SELECT
    Y.StateProvince
  , FORMAT(ROUND(Y.Amount, 2, 2), '#,#.##')                                      AS TotalSalesAmount
  , FORMAT(ROUND(SUM(Y.Amount) OVER (ORDER BY Y.StateProvince), 2, 2), '#,#.##') AS RunningTotalSalesAmount
FROM (
    SELECT
        X.StateProvinceName AS StateProvince
      , SUM(X.TotalDue)     AS Amount
    FROM (
        SELECT
            SalesOrderHeader.SalesOrderID
          , SalesOrderHeader.BillToAddressID
          , SalesOrderHeader.TerritoryID
          , SalesOrderHeader.TotalDue
          , [Address].AddressID
          , [Address].StateProvinceID
          , [StateProvince].[Name] AS StateProvinceName
        FROM [AdventureWorks2022].[Sales].[SalesOrderHeader] AS SalesOrderHeader
        LEFT JOIN [AdventureWorks2022].[Person].[Address] AS [Address]
            ON SalesOrderHeader.BillToAddressID = [Address].AddressID
        LEFT JOIN [AdventureWorks2022].[Person].[StateProvince] AS [StateProvince]
            ON [Address].StateProvinceID = [StateProvince].StateProvinceID
    ) AS X
    GROUP BY X.StateProvinceName
) AS Y
GROUP BY Y.StateProvince, Y.Amount
```

---

### Output (truncated)

```
Row  StateProvince    TotalSalesAmount  RunningTotalSalesAmount
1    Alabama          51,198.85         51,198.85
2    Alberta          1,597,568.47      1,648,767.32
3    Arizona          1,619,092.02      3,267,859.35
4    Bayern           673,132.43        3,940,991.79
...
30   Minnesota        1,073,914.61      55,737,824.03
31   Mississippi      589,365.46        56,327,189.49
32   Missouri         2,034,067.67      58,361,257.16
...
49   Rhode Island     15,379.54         88,705,740.34
50   Saarland         1,721,063.51      90,426,803.85
51   Seine (Paris)    2,233,808.19      92,660,612.05
...
67   Wisconsin        551,114.39        121,952,740.88
68   Wyoming          967,170.62        122,919,911.51
69   Yveline          296,874.60        123,216,786.11
(69 rows affected)
```

---

## 💡 Exercise 4 — Running total by state or province at individual order level

### Approach
Instead of summing to one total per state, we keep each individual order amount and use `SUM() OVER (PARTITION BY StateProvinceName ORDER BY TotalDue)` to accumulate the running total **within each state** — restarting the counter for each new state. This gives a granular view of how sales accumulate order by order within each state.

---

### T-SQL code

```sql
SELECT
    X.StateProvinceName
  , FORMAT(ROUND(X.TotalDue, 2, 2), '#,#.##')                       AS SalesOrderAmount
  , FORMAT(ROUND(SUM(X.TotalDue) OVER (PARTITION BY X.StateProvinceName
        ORDER BY X.StateProvinceName, X.TotalDue), 2, 2), '#,#.##') AS RunningTotalSalesAmount
FROM (
    SELECT
        SalesOrderHeader.SalesOrderID
      , SalesOrderHeader.BillToAddressID
      , SalesOrderHeader.TerritoryID
      , SalesOrderHeader.TotalDue
      , [Address].AddressID
      , [Address].StateProvinceID
      , [StateProvince].[Name] AS StateProvinceName
    FROM [AdventureWorks2022].[Sales].[SalesOrderHeader] AS SalesOrderHeader
    LEFT JOIN [AdventureWorks2022].[Person].[Address] AS [Address]
        ON SalesOrderHeader.BillToAddressID = [Address].AddressID
    LEFT JOIN [AdventureWorks2022].[Person].[StateProvince] AS [StateProvince]
        ON [Address].StateProvinceID = [StateProvince].StateProvinceID
) AS X
GROUP BY X.StateProvinceName, X.TotalDue
ORDER BY X.StateProvinceName
```

---

### Output (truncated)

```
Row    StateProvince  SalesOrderAmount  RunningTotalSalesAmount
1      Alabama        2.53              2.53
2      Alabama        27.44             29.97
3      Alabama        38.67             68.64
4      Alabama        82.33             150.97
...
19     Alabama        5,683.61          34,351.91
20     Alabama        7,294.29          41,646.20
21     Alabama        9,186.17          50,832.38    ← Alabama total resets for next state
22     Alberta        8.04              8.04         ← Alberta starts fresh
23     Alberta        18.37             26.42
...
108    Alberta        104,478.56        1,482,178.73
109    Alberta        111,207.22        1,593,385.95
110    Arizona        12.69             12.69        ← Arizona starts fresh
111    Arizona        13.71             26.41
...
10906  Yveline        2,832.50          143,449.80
10907  Yveline        3,953.98          147,403.79
(10,907 rows affected)
```

---

## 🔍 Key concept — how the running total window function works

`SUM(Amount) OVER (ORDER BY column)` is a **window function** that calculates a cumulative sum. Unlike a regular `SUM()` aggregate which collapses all rows into one, a window function keeps every row and adds a new column showing the accumulated total up to and including each row.

### Exercise 3 vs Exercise 4 — the role of `PARTITION BY`

| | Exercise 3 | Exercise 4 |
|---|---|---|
| Granularity | One total per state | One row per order |
| `PARTITION BY` | None — accumulates across all states | `StateProvinceName` — resets per state |
| `ORDER BY` | `StateProvince` alphabetically | `StateProvinceName, TotalDue` |
| Running total resets? | Never — grows to $123M by the last state | Yes — resets to `0` for each new state |
| Use when | Showing cumulative share of total sales | Showing how sales build up within each state |

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
