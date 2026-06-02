Exercise: Rank salespeople by monthly sales, resetting the rank every month. 

Solution: We used an OUTER query and an INNER query. 
The INNER query calculates the monthly sales for every salespeople each year, while the OUTER query produces a ranking for these amounts using the RANK() function, resetting the rank every month.

Tables used
[AdventureWorks2022].[Sales].[SalesOrderHeader]

Functions used
SUM function
YEAR function
MONTH function
RANK() OVER
ROUND function
FORMAT function

T-SQL code
USE AdventureWorks2022;
GO	

SELECT	X.[SalesPersonID]
	,X.YearSale
	,X.MonthSale
	,RANK() OVER (PARTITION BY X.YearSale, X.MonthSale ORDER BY X.MonthlySales) AS SalesPersonByMonthlySalesRank
	,FORMAT(ROUND(X.MonthlySales, 2, 2), '#,#.##') AS MonthlySales
FROM 
(
SELECT [SalesPersonID]
      ,YEAR([OrderDate]) AS YearSale
      ,MONTH([OrderDate]) AS MonthSale
      ,SUM([TotalDue]) AS MonthlySales
FROM [AdventureWorks2022].[Sales].[SalesOrderHeader]
WHERE [SalesPersonID] IS NOT NULL
GROUP BY [SalesPersonID], YEAR([OrderDate]), MONTH([OrderDate])
) AS X


Output
SalesPersonID	YearSale	MonthSale	SalesPersonByMonthlySalesRank	MonthlySales
276		2011		5		1				6,167.16
278		2011		5		2				10,254.85
280		2011		5		3				27,510.41
277		2011		5		4				52,586.67
281		2011		5		5				67,231.71
275		2011		5		6				71,792.84
283		2011		5		7				78,223.3
279		2011		5		8				117,577.6
282		2011		5		9				119,678.92
274		2011		7		1				23,130.29
281		2011		7		2				38,387.53
283		2011		7		3				73,343.36
275		2011		7		4				122,246.74
280		2011		7		5				172,367.99
278		2011		7		6				176,300.29
282		2011		7		7				180,684.84
277		2011		7		8				300,998.15
276		2011		7		9				305,145.22
279		2011		7		10				340,236.6
...
288		2014		4		1				1,428.61
285		2014		5		1				4,725.95
274		2014		5		2				42,546.92
290		2014		5		3				134,318.85
288		2014		5		4				152,878.6
283		2014		5		5				159,928.17
284		2014		5		6				161,583.98
278		2014		5		7				163,799.47
280		2014		5		8				170,556.7
281		2014		5		9				248,058.19
286		2014		5		10				255,981.92
279		2014		5		11				262,264.66
276		2014		5		12				319,181.39
277		2014		5		13				380,780.5
275		2014		5		14				417,208.47
282		2014		5		15				480,379.23
289		2014		5		16				495,918.61

(423 rows affected)
