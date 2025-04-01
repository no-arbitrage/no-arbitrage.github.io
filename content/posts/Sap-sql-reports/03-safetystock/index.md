---
title: "Calculating Safety Stock"
summary: "Safety Stock Management"
categories: ["StockManagement",]
series: ["SQL Reports for SAP-B1"]
series_order: 3
tags: ["sql","reporting", "sap", "BI"]
heroStyle : "basic"
date: 2025-04-01
draft: false
---
## Background
The company has a nearly 24,000 product lines in the system. Due to the nature of the products, the demand had been relatively stable therefore required no change in safety stock quantity. However, Covid disrupted the this stability, leaving large number of product lines experiencing demand shifts. In order to automate the safety stock level based on the shifting demand and maintain a good service level, I created the following query

## Query Highlights
+ 18 months of data, however we only look at the previous 12 weeks to determine what's the optimal stock level 
+ If you are not familiar with SafetyStock, I recommend downlaod the [MIT's Safety Stock leafpage](https://web.mit.edu/2.810/www/files/readings/King_SafetyStock.pdf) to get started. 
+  Notice where COALESCE is used, this is to fill the gaps where SQL doesn't have the data, and if not careful, the data will be very much biased due to lacking periods of information. 
+ You can analyse the final result by plotting on a line chart, and observe how the stafety stock (recommended level) changed in relation to the shift in demand quantity.  

```sql

WITH Calendar AS 
	(
		SELECT 
		-1 [PeriodNum], DATEADD( DAY, 1-DATEPART(dw, GETDATE()), GETDATE()) [WeekEnd]
		UNION ALL
		SELECT
		[PeriodNum]-1, DATEADD( DAY, -7, [WeekEnd] ) 
		FROM Calendar
		WHERE  [WeekEnd] >= EOMONTH(GETDATE(), -18)
	) 

, FullSalesSeries AS (
    SELECT DISTINCT 
		Y.PeriodNum,
        CAST(Y.WeekEnd AS DATE) [WeekEnd],
        I.ItemCode 
    FROM Calendar Y 
    CROSS JOIN ( 
		SELECT DISTINCT 
		T.ItemCode 
		FROM dbo.OITM T
		WHERE EXISTS ( 
				SELECT * 
				FROM RDR1 T98 WITH (NOLOCK)
				WHERE T98.DocDate >= EOMONTH ( GETDATE(), -18 ) 
				AND T98.ItemCode = T.ItemCode
			) ) I
)
, TotalDemand AS (
	SELECT 
		T0.ItemCode, 
		DATEADD(DAY, 8-DATEPART( dw, T0.DocDate), T0.DocDate ) [WeekEnd],
	
		SUM(T0.Qty_Exploded )			[ProdQuantity], 
		SUM(T0.Sale_Qty )				[SalesQuantity],
		SUM(T0.Qty_Exploded ) + SUM(T0.Sale_Qty ) [Demand_total]
	FROM 
		(
			SELECT 
			Main.DocNum, 
			Main.CardCode, 
			Lines.DocDate, 
			Lines.ItemCode AS 'Item_father', 
			Lines.Quantity AS 'Sales_Qty_Indirect', -- Lines.Price AS 'Price', Lines.GrssProfit, Lines.LineTotal,
			Lines.GrssProfSC AS 'Profit_Indirect', 
			Lines.TotalSumSy AS 'Sales_Indirect',
			0 AS 'Sale_Qty',
			0 AS 'Profit',
			0 AS 'Sales',
			Parts.Code AS 'ItemCode', 
			Parts.Quantity AS 'BaseQty',  
			Lines.Quantity * Parts.Quantity AS 'Qty_Exploded', -- BOM Explosion
			0 AS 'ConsumpType'
			FROM ORDR Main WITH (NOLOCK)
			INNER JOIN RDR1 Lines WITH (NOLOCK) ON Main.DocEntry = Lines.DocEntry
			INNER JOIN OCRD Cards WITH (NOLOCK) ON Cards.CardCode = Main.CardCode
			INNER JOIN ITT1 Parts WITH (NOLOCK)ON Parts.Father = Lines.ItemCode
			INNER JOIN OITM Items WITH (NOLOCK) ON Items.ItemCode = Parts.Father
		WHERE  Items.CardCode = 'S001558' 
			AND Main.DocDate >= EOMONTH ( GETDATE(), -18) 
			
		UNION ALL

		SELECT 
			Main.DocNum, 
			Cards.CardCode,
			DATEADD( DAY, 14, Lines.DocDate) ,  -- Assuming 14 days assemble time which bring the date of actual ahead by 14 days
			Lines.ItemCode,
			0, 
			0, 
			0, 
			Lines.Quantity, 
			Lines.GrssProfSC ,
			Lines.TotalSumSy ,
			Lines.ItemCode, 
			1, 
			0, 
			1
		FROM ORDR Main WITH (NOLOCK) 
			INNER JOIN RDR1 Lines WITH (NOLOCK) ON Main.DocEntry = Lines.DocEntry
			INNER JOIN OCRD Cards WITH (NOLOCK) ON Cards.CardCode = Main.CardCode
			INNER JOIN OITM Items WITH (NOLOCK) ON Items.ItemCode = Lines.ItemCode 

			WHERE  Main.DocDate >= EOMONTH ( GETDATE(), -18) 
		) T0
		GROUP BY T0.ItemCode, 
		DATEADD(DAY, 8-DATEPART( dw, T0.DocDate), T0.DocDate )
		
) 
, SalesWithNoGap AS (
	SELECT 
	F.ItemCode, 
	F.WeekEnd, 
	F.PeriodNum,
	COALESCE ( S.ProdQuantity, 0) [ProdQuantity],
	COALESCE ( S.SalesQuantity,0) [SalesQuantity]
    FROM FullSalesSeries F
    LEFT JOIN TotalDemand S ON F.WeekEnd = S.WeekEnd AND F.ItemCode = S.ItemCode
	
)

, MovingStats AS (
	SELECT 
	T0.WeekEnd, 
	T0.PeriodNum, 
	T0.ItemCode, 
	T0.ProdQuantity , T0.SalesQuantity,
	T0.ProdQuantity + T0.SalesQuantity [TotalQuantity], 
	AVG( T0.ProdQuantity + T0.SalesQuantity ) OVER ( 
		PARTITION BY T0.ItemCode
		ORDER BY T0.PeriodNum ASC 
		ROWS BETWEEN 12 PRECEDING AND 1 PRECEDING
	) [MovingAvg], 

	STDEV(T0.ProdQuantity + T0.SalesQuantity ) OVER ( 
		PARTITION BY T0.ItemCode
		ORDER BY T0.PeriodNum ASC 
		ROWS BETWEEN 12 PRECEDING AND 1 PRECEDING
	) [MovingStdev]


	FROM SalesWithNoGap T0
)
-- , SeasonalityIndex AS (
-- 	SELECT 
-- 	T.ItemCode, 
-- 	DATEPART(WEEK, T.WeekEnd) [WeekNum],
--     AVG(T.TotalQuantity / T.MovingAvg) [Seasonality]
-- 	FROM
--     MovingAverage T
-- 	GROUP BY 
-- 	T.ItemCode, 
-- 	DATEPART(WEEK, T.WeekEnd)
-- )


, STOCK_INFO AS (
SELECT
T1.ItemCode,
T1.WeekEnd, 
T1.PeriodNum, 
T1.ProdQuantity , T1.SalesQuantity, T1.TotalQuantity,

LeadTimeDemand =  T1.[MovingAvg] * T0.[LeadTime] / 7,
SafetyStock = 
(											--- THIS SECTION NEEDS TO BE ADAPTED TO YOUR COMPANY NEEDS
	CASE 
	WHEN T0.QryGroup61 = 'Y'				--- WE HAD A TIERED CORE STOCK SYSTEM WITH SUPERCORE COVERING 95% SERVICE LEVEL TARGET
	THEN 1.88 -- 96% SL 'Supercore'
	ELSE 1.64
	END
) * SQRT( T0.[LeadTime] / 7) * T1.[MovingStdev]

FROM 
OITM T0 JOIN
MovingStats T1 ON T1.ItemCode = T0.ItemCode

)
SELECT * FROM STOCK_INFO

GO
```
