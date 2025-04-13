---
title: "SQL Stock Aging Report for SAP B1"
summary: Basic raw tbble view and pivot stock value view.
categories: ["StockManagement",]
series: ["SQL Reports for SAP-B1"]
series_order: 4
tags: ["sql","reporting", "sap", "BI"]
heroStyle : "basic"
date: 2025-04-13
draft: false
---
### Basic Form directory querying the underlying table 
Three tables under Inventory and Production module
OBTN : Batch Numbers Master Data 
OBTQ : Batch No. Quantities 
OITW : Items - Warehouse

Notice 
OBTN holds batch number, in/out date
OBTQ allocate quantity to warehouses (virtual)
OITW records cost prices by each warehouse


Please note, your company might use different method for stock valuation. The company I worked with use warehouse in SAP to hold stocks of different cost prices.  Also, only batch managed stock will be captured by this query. 

```
SELECT 
T0.ItemCode, 
T0.ItemName, 
T1.DistNumber [BatchNum], -- Batch Number 
T3.WhsCode, 
T1.InDate, 
T1.ExpDate, 
T2.WhsCode, 
T2.Quantity, 
T2.CommitQty, 
T3.AvgPrice, 
GETDATE() [LogDate] 
FROM OITM T0 WITH (NOLOCK) 
JOIN OBTN T1 WITH (NOLOCK) ON T0.ItemCode = T1.ItemCode 
JOIN OBTQ T2 WITH (NOLOCK) ON 
	T2.ItemCode = T1.ItemCode AND T2.SysNumber = T1.SysNumber AND
	T2.Quantity != 0 
JOIN OITW T3 WITH (NOLOCK) ON 
	T3.ItemCode = T1.ItemCode AND T2.WhsCode = T3.WhsCode 
WHERE
T0.OnHand != 0 
```

### Advanced form: Stock Value Pivot view

```
SELECT 
Pvt.ItemCode,
Pvt.[0] [Within 1 Year],
Pvt.[1] [Between 1-2 Years],
Pvt.[2] [Between 2-3 Years],
Pvt.[3] [3 Years and above],
Pvt.[-1] [Already expired]
FROM 
(
	SELECT 
	T0.ItemCode, 
	(
		CASE 
		WHEN DATEDIFF( DAY , GETDATE(), T1.ExpDate ) > 365 *3
		THEN 3
		WHEN DATEDIFF( DAY , GETDATE(), T1.ExpDate ) > 365 *2
		THEN 2
		WHEN DATEDIFF( DAY , GETDATE(), T1.ExpDate ) > 365
		THEN 1
		WHEN DATEDIFF( DAY , GETDATE(), T1.ExpDate ) > 0
		THEN 0
		ELSE -1 
		END
	) [ShelfLife], 
	T2.Quantity * T3.AvgPrice [StockValue]
	FROM OITM T0 WITH (NOLOCK) 
	JOIN OBTN T1 WITH (NOLOCK) ON T0.ItemCode = T1.ItemCode 
	JOIN OBTQ T2 WITH (NOLOCK) ON 
		T2.ItemCode = T1.ItemCode AND T2.SysNumber = T1.SysNumber AND
		T2.Quantity != 0 
	JOIN OITW T3 WITH (NOLOCK) ON 
		T3.ItemCode = T1.ItemCode AND T2.WhsCode = T3.WhsCode 
	WHERE
	T0.OnHand != 0 
) T 
PIVOT
(
	SUM( StockValue ) FOR [ShelfLife] IN ( [0],[1],[2],[3],[-1] )
) AS Pvt

```
Be careful with the results. If an item doesn't have batch management info, this report won't pick up those line. If the query begins with OITM to the furthest left and left join this query, then you will have the full stock aging report for every line item, with each item displaying in one line. 