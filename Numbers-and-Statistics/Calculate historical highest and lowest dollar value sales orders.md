# Highest and lowest dollar value sales orders calculations

## 🎯 Exercise
Calculate the historical highest and lowest dollar value sales orders at four levels of granularity:
1. Overall (all geographies combined)
2. By geographical group (Europe, North America, Pacific)
3. By country
4. By state or province

---

## 💡 Exercise 1 — Highest and lowest sales for all geographies

### Approach
We use `MAX()` and `MIN()` directly on `TotalDue` from `SalesOrderHeader` to retrieve the single highest and lowest order amounts across all records.

### T-SQL functions used

| Function | Purpose |
|---|---|
| `MAX(TotalDue)` | Returns the highest sales order amount |
| `MIN(TotalDue)` | Returns the lowest sales order amount |
| `ROUND(value, decimals, function)` | Rounds to 2 decimal places |
| `FORMAT(value, format)` | Formats with comma separators and decimal places |

### Table used

| Schema | Table |
|---|---|
| `Sales` | `SalesOrderHeader` |

---

### T-SQL code

```sql
SELECT
    FORMAT(ROUND(MAX(SalesOrderHeader.[TotalDue]), 2, 2), '#,#.##') AS HighestDollarValueSales
  , FORMAT(ROUND(MIN(SalesOrderHeader.[TotalDue]), 2, 2), '#,#.##') AS LowestDollarValueSales
FROM [AdventureWorks2022].[Sales].[SalesOrderHeader] AS SalesOrderHeader
```

### Output

```
HighestDollarValueSales  LowestDollarValueSales
187,487.82               1.51
(1 row affected)
```

---

## 💡 Exercise 2 — Highest and lowest sales by geographical group

### Tables used

| Schema | Table |
|---|---|
| `Sales` | `SalesOrderHeader` |
| `Sales` | `SalesTerritory` |

---

### T-SQL code

```sql
SELECT
    SalesTerritory.[Group]
  , FORMAT(ROUND(MAX(SalesOrderHeader.[TotalDue]), 2, 2), '#,#.##') AS HighestDollarValueSales
  , FORMAT(ROUND(MIN(SalesOrderHeader.[TotalDue]), 2, 2), '#,#.##') AS LowestDollarValueSales
FROM [AdventureWorks2022].[Sales].[SalesOrderHeader] AS SalesOrderHeader
LEFT JOIN [AdventureWorks2022].[Sales].[SalesTerritory] AS SalesTerritory
    ON SalesOrderHeader.TerritoryID = SalesTerritory.TerritoryID
GROUP BY SalesTerritory.[Group]
```

### Output

```
Group          HighestDollarValueSales  LowestDollarValueSales
North America  187,487.82               1.51
Pacific        71,729.85                2.53
Europe         166,537.08               3.03
(3 rows affected)
```

---

## 💡 Exercise 3 — Highest and lowest sales by country

### Tables used

| Schema | Table |
|---|---|
| `Sales` | `SalesOrderHeader` |
| `Sales` | `SalesTerritory` |
| `Person` | `CountryRegion` |
| `Person` | `StateProvince` |

---

### T-SQL code

```sql
SELECT
    SalesTerritory.[Group]
  , CountryRegion.[Name]                                            AS CountryName
  , FORMAT(ROUND(MAX(SalesOrderHeader.[TotalDue]), 2, 2), '#,#.##') AS HighestDollarValueSales
  , FORMAT(ROUND(MIN(SalesOrderHeader.[TotalDue]), 2, 2), '#,#.##') AS LowestDollarValueSales
FROM [AdventureWorks2022].[Sales].[SalesOrderHeader] AS SalesOrderHeader
LEFT JOIN [AdventureWorks2022].[Sales].[SalesTerritory] AS SalesTerritory
    ON SalesOrderHeader.TerritoryID = SalesTerritory.TerritoryID
LEFT JOIN [AdventureWorks2022].[Person].[CountryRegion] AS CountryRegion
    ON SalesTerritory.CountryRegionCode = CountryRegion.CountryRegionCode
LEFT JOIN [AdventureWorks2022].[Person].[StateProvince] AS StateProvince
    ON SalesOrderHeader.TerritoryID = StateProvince.TerritoryID
GROUP BY SalesTerritory.[Group], CountryRegion.[Name]
ORDER BY SalesTerritory.[Group], CountryRegion.[Name]
```

### Output

```
Group          CountryName     HighestDollarValueSales  LowestDollarValueSales
Europe         France          166,537.08               4.4
Europe         Germany         117,506.11               4.4
Europe         United Kingdom  130,249.25               3.03
North America  Canada          170,512.66               2.53
North America  United States   187,487.82               1.51
Pacific        Australia       71,729.85                2.53
(6 rows affected)
```

---

## 💡 Exercise 4 — Highest and lowest sales by state or province

### Approach
We use a subquery to join `SalesOrderHeader` to `Address` via `BillToAddressID`, then join `StateProvince` to retrieve the state or province name. The outer query groups by state name and applies `MAX()` and `MIN()`.

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
    X.StateProvinceName
  , FORMAT(ROUND(MAX(X.[TotalDue]), 2, 2), '#,#.##') AS HighestDollarValueSales
  , FORMAT(ROUND(MIN(X.[TotalDue]), 2, 2), '#,#.##') AS LowestDollarValueSales
FROM (
    SELECT
        SalesOrderHeader.[SalesOrderID]
      , SalesOrderHeader.[BillToAddressID]
      , SalesOrderHeader.[TerritoryID]
      , SalesOrderHeader.[TotalDue]
      , [Address].[AddressID]
      , [Address].[StateProvinceID]
      , [StateProvince].[Name] AS StateProvinceName
    FROM [AdventureWorks2022].[Sales].[SalesOrderHeader] AS SalesOrderHeader
    LEFT JOIN [AdventureWorks2022].[Person].[Address] AS [Address]
        ON SalesOrderHeader.BillToAddressID = [Address].AddressID
    LEFT JOIN [AdventureWorks2022].[Person].[StateProvince] AS [StateProvince]
        ON [Address].StateProvinceID = [StateProvince].StateProvinceID
) AS X
GROUP BY StateProvinceName
```

### Output (truncated)

```
StateProvinceName     HighestDollarValueSales  LowestDollarValueSales
Moselle               3,953.98                 5.51
Garonne (Haute)       140,042.12               16.55
Illinois              122,500.66               8.04
Wisconsin             54,611.53                76.16
Seine et Marne        13,855.97                5.51
Brandenburg           38,066.51                5.51
Maine                 76,861.57                11,086.32
...
Nevada                109,948.74               23.17
Victoria              33,550.67                2.53
Bayern                77,823.99                4.4
Kentucky              35,527.84                42.07
New Mexico            66,861.40                12,074.28
(69 rows affected)
```

---

## 🔍 Key observations

- The **overall highest** order (`$187,487.82`) comes from **North America** specifically the **United States** — confirmed across all three drill-down levels.
- The **overall lowest** order (`$1.51`) also comes from **North America** — United States.
- **Exercise 4** uses `BillToAddressID` rather than `TerritoryID` to join to `StateProvince` — this gives the billing state of each order rather than the sales territory, providing a more granular geographic breakdown.
- Some states show a notably high minimum (e.g. **Maine**: `$11,086.32`, **New Mexico**: `$12,074.28`) — suggesting those states have fewer, higher-value orders.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Dates-Numbers-and-Statistics](https://github.com/arieferencz/SQL_SERVER-Working-with-Dates-Numbers-and-Statistics)
