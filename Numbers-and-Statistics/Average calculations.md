# Average calculations

## 🎯 Exercise
Calculate the historic average sales order amount at three levels of granularity:
1. Overall (all geographies combined)
2. By geographical group (Europe, North America, Pacific)
3. By country

---

## 💡 Exercise 1 — Historic average for all geographies

### T-SQL functions used

| Function | Purpose |
|---|---|
| `AVG(value)` | Calculates the arithmetic mean directly |
| `SUM(value) / COUNT(value)` | Alternative calculation — total divided by count |
| `ROUND(value, decimals, function)` | Rounds to the specified number of decimal places |
| `FORMAT(value, format)` | Formats the result with comma separators and decimal places |

### Table used

| Schema | Table |
|---|---|
| `Sales` | `SalesOrderHeader` |

---

### Solution 1.1 — Using `AVG()`

```sql
SELECT ROUND(AVG(TotalDue), 2, 2) AS HistoricAverageSalesAmount
FROM [AdventureWorks2022].[Sales].[SalesOrderHeader]
```

```
HistoricAverageSalesAmount
3915.99
(1 row affected)
```

---

### Solution 1.2 — Using `SUM()` / `COUNT()`

```sql
SELECT
    FORMAT(ROUND(SUM(TotalDue) / COUNT(TotalDue), 2, 2), '#,#.##') AS HistoricAverageSalesAmount
  , FORMAT(SUM(TotalDue), '#,#.##')                                AS SumTotalSalesAmount
  , FORMAT(COUNT(TotalDue), '#,#')                                 AS CountTotalSales
FROM [AdventureWorks2022].[Sales].[SalesOrderHeader]
```

```
HistoricAverageSalesAmount  SumTotalSalesAmount  CountTotalSales
3,915.99                    123,216,786.12       31,465
(1 row affected)
```

---

## 💡 Exercise 2 — Historic average by geographical group

### Tables used

| Schema | Table |
|---|---|
| `Sales` | `SalesOrderHeader` |
| `Sales` | `SalesTerritory` |

---

### Solution 2.1 — Using `AVG()` as a window function with `PARTITION BY`

```sql
SELECT DISTINCT
    SalesTerritory.[Group]
  , FORMAT(ROUND(AVG(TotalDue) OVER (PARTITION BY SalesTerritory.[Group]), 2), '#,#.##')
        AS HistoricAverageSalesAmount
FROM [AdventureWorks2022].[Sales].[SalesOrderHeader] AS SalesOrderHeader
LEFT JOIN [AdventureWorks2022].[Sales].[SalesTerritory] AS SalesTerritory
    ON SalesOrderHeader.TerritoryID = SalesTerritory.TerritoryID
```

```
Group          HistoricAverageSalesAmount
Europe         2,604.37
North America  5,539.41
Pacific        1,726.49
(3 rows affected)
```

---

### Solution 2.2 — Using `SUM()` / `COUNT()` with `GROUP BY`

```sql
SELECT DISTINCT
    SalesTerritory.[Group]
  , FORMAT(ROUND(SUM(TotalDue) / COUNT(TotalDue), 2, 2), '#,#.##') AS HistoricAverageSalesAmount
FROM [AdventureWorks2022].[Sales].[SalesOrderHeader] AS SalesOrderHeader
LEFT JOIN [AdventureWorks2022].[Sales].[SalesTerritory] AS SalesTerritory
    ON SalesOrderHeader.TerritoryID = SalesTerritory.TerritoryID
GROUP BY SalesTerritory.[Group]
ORDER BY FORMAT(ROUND(SUM(TotalDue) / COUNT(TotalDue), 2, 2), '#,#.##')
```

```
Group          HistoricAverageSalesAmount
Pacific        1,726.49
Europe         2,604.37
North America  5,539.4
(3 rows affected)
```

---

### Solution 2.3 — Using `GROUP BY ... WITH ROLLUP` (includes grand total row)

```sql
SELECT
    COALESCE(SalesTerritory.[Group], 'Total')                      AS CountryCode
  , FORMAT(SUM(SalesOrderHeader.TotalDue), '#,#.##')               AS TotalSalesAmount
  , FORMAT(COUNT(TotalDue), '#,#')                                 AS CountTotalSales
  , FORMAT(ROUND(SUM(TotalDue) / COUNT(TotalDue), 2, 2), '#,#.##') AS AverageSalesAmount
FROM [AdventureWorks2022].[Sales].[SalesOrderHeader] AS SalesOrderHeader
LEFT JOIN [AdventureWorks2022].[Sales].[SalesTerritory] AS SalesTerritory
    ON SalesOrderHeader.TerritoryID = SalesTerritory.TerritoryID
GROUP BY SalesTerritory.[Group] WITH ROLLUP
```

```
CountryCode    TotalSalesAmount    CountTotalSales  AverageSalesAmount
Europe         22,173,617.63       8,514            2,604.37
North America  89,228,792.39       16,108           5,539.4
Pacific        11,814,376.1        6,843            1,726.49
Total          123,216,786.12      31,465           3,915.99
(4 rows affected)
```

---

## 💡 Exercise 3 — Historic average by country

---

### Solution 3.1 — Using `AVG()` as a window function with `PARTITION BY`

```sql
SELECT DISTINCT
    SalesTerritory.CountryRegionCode
  , FORMAT(ROUND(AVG(TotalDue) OVER (PARTITION BY SalesTerritory.CountryRegionCode), 2), '#,#')
        AS HistoricAverageSalesAmount
FROM [AdventureWorks2022].[Sales].[SalesOrderHeader] AS SalesOrderHeader
LEFT JOIN [AdventureWorks2022].[Sales].[SalesTerritory] AS SalesTerritory
    ON SalesOrderHeader.TerritoryID = SalesTerritory.TerritoryID
```

```
CountryRegionCode  HistoricAverageSalesAmount
CA                 4,524
US                 5,882
FR                 3,039
GB                 2,664
AU                 1,726
DE                 2,089
(6 rows affected)
```

---

### Solution 3.2 — Using `SUM()` / `COUNT()` with `GROUP BY`

```sql
SELECT DISTINCT
    SalesTerritory.CountryRegionCode
  , FORMAT(ROUND(SUM(TotalDue) / COUNT(TotalDue), 2, 2), '#,#.##') AS HistoricAverageSalesAmount
FROM [AdventureWorks2022].[Sales].[SalesOrderHeader] AS SalesOrderHeader
LEFT JOIN [AdventureWorks2022].[Sales].[SalesTerritory] AS SalesTerritory
    ON SalesOrderHeader.TerritoryID = SalesTerritory.TerritoryID
GROUP BY SalesTerritory.CountryRegionCode
```

```
CountryRegionCode  HistoricAverageSalesAmount
DE                 2,089.14
GB                 2,663.57
AU                 1,726.49
CA                 4,523.95
FR                 3,038.82
US                 5,882.39
(6 rows affected)
```

---

### Solution 3.3 — Using `GROUP BY ... WITH ROLLUP` (includes grand total row)

```sql
SELECT
    COALESCE(SalesTerritory.CountryRegionCode, 'Total')            AS CountryCode
  , FORMAT(SUM(SalesOrderHeader.TotalDue), '#,#.##')               AS TotalSalesAmount
  , FORMAT(COUNT(TotalDue), '#,#')                                 AS CountTotalSales
  , FORMAT(ROUND(SUM(TotalDue) / COUNT(TotalDue), 2, 2), '#,#.##') AS AverageSalesAmount
FROM [AdventureWorks2022].[Sales].[SalesOrderHeader] AS SalesOrderHeader
LEFT JOIN [AdventureWorks2022].[Sales].[SalesTerritory] AS SalesTerritory
    ON SalesOrderHeader.TerritoryID = SalesTerritory.TerritoryID
GROUP BY SalesTerritory.CountryRegionCode WITH ROLLUP
```

```
CountryCode  TotalSalesAmount    CountTotalSales  AverageSalesAmount
AU           11,814,376.1        6,843            1,726.49
CA           18,398,929.19       4,067            4,523.95
DE           5,479,819.58        2,623            2,089.14
FR           8,119,749.35        2,672            3,038.82
GB           8,574,048.71        3,219            2,663.57
US           70,829,863.2        12,041           5,882.39
Total        123,216,786.12      31,465           3,915.99
(7 rows affected)
```

---

## 🔍 Key differences between the approaches

| Approach | When to use |
|---|---|
| `AVG()` aggregate | Simple single-value average — cleanest syntax |
| `SUM() / COUNT()` | When you also need the total and count alongside the average |
| `AVG() OVER (PARTITION BY ...)` | Window function — returns the group average on every row without collapsing the result set |
| `GROUP BY ... WITH ROLLUP` | When you need both group-level averages and a grand total in one query |

> **`AVG()` vs `SUM() / COUNT()`:** These two approaches can produce slightly different results due to rounding behaviour. `AVG()` performs the division internally before rounding, while `SUM() / COUNT()` allows you to control the precision explicitly using `ROUND()`. In this dataset both return `3,915.99` overall.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
