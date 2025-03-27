
---
title: "Ultimate Sales Analysis Report for SAP B1 (SQL)"
summary: "This is the first in the SQL series"
categories: ["Finance",]
series: ["SQL Reports for SAP-B1"]
series_order: 1
tags: ["sql","reporting", "sap", "BI"]
heroStyle : "basic"
date: 2025-03-27
draft: false
---

Please note
1. Provided for educational purpose. And it is provided as it is, with no liabilities or responsibilities assumed. 
2. Please consult a qualified SAP consultant before you execute any unverified scripts in a production environment. 
3. Sales Correction, as part of the SAP stock report, is not included in this report. They don't normally happen anyway. 




```sql
SELECT 

T0.DocNum, 

T0.SlpCode [SLpCode],
T0.SlpName [SlpName],

T0.CardCode [Customercode],
T0.CardName [CustomerName],

T0.GroupCode,
T0.ProjectCod [ProjectCod],

T0.DocDate,
T0.DocDueDate,

T0.ItemCode,
T0.ItemName,

T0.ItmsGrpCod ,
T0.ItmsGrpNam,

T0.LineNum,
T0.Quantity,

T0.GrossCalc [Gross_sales],
T0.GrossCalcSys [Gross_sales_in_base_currency],

T0.Sales [Net_sales] , -- after discount
T0.SalesSys [Sales_in_base_currency],

T0.Price [Price],
T0.PriceSys [Price_in_base_currency],

T0.GrossProfit [GrossProfit],
T0.GrossProfitSys [GrossProfit_in_base_currency]

FROM
(
/*
------------------------------------------------------------------------------------------------------------
Part 1: Invoice Sales 
------------------------------------------------------------------------------------------------------------
*/

	SELECT Main.DocEntry,
		Main.DocNum,
		Slp.SlpCode,
		Slp.SlpName,
		Cards.CardCode,
		Cards.CardName,
		Cards.ProjectCod,
		Main.DocDate,
		Main.DocDueDate,
		Items.ItemCode,
		ItmGrp.ItmsGrpNam,
		Items.ItemName,
		Quantity = CASE
			Lines.NoInvtryMv
			WHEN 'Y' THEN 0
			ELSE CASE
				Lines.usebaseun
				WHEN 'Y' THEN Lines.Quantity * 1
				ELSE (Lines.Quantity * Lines.NumPerMsr) * 1
			END
		END,
		Price = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE Lines.Price
		END,
		Sales = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE (
				(
					(
						(
							CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
						) / Lines.NumPerMsr
					) * (Lines.Quantity * Lines.NumPerMsr)
				) - (
					(
						(
							(
								CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
							) / Lines.NumPerMsr
						) * (Lines.Quantity * Lines.NumPerMsr)
					) * (
						(
							CASE
								WHEN (
									SUM(DISTINCT(Main.DocTotal)) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
								) = 0 THEN 0
								ELSE (
									CASE
										(
											SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
										)
										WHEN 0 THEN 0
										ELSE (
											SUM(DISTINCT(Main.DiscSum)) / (
												SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
											)
										)
									END
								)
							END
						) * 100
					) / 100
				)
			) * 1
		END,
		GrossProfit = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN (0 - (Lines.GrossBuyPr * Lines.Quantity)) * 1
			ELSE (
				(
					(
						(
							CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
						) / Lines.NumPerMsr
					) * (Lines.Quantity * Lines.NumPerMsr)
				) - (
					(
						(
							(
								CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
							) / Lines.NumPerMsr
						) * (Lines.Quantity * Lines.NumPerMsr)
					) * (
						(
							CASE
								WHEN (
									SUM(DISTINCT(Main.DocTotal)) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
								) = 0 THEN 0
								ELSE (
									CASE
										(
											SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
										)
										WHEN 0 THEN 0
										ELSE (
											SUM(DISTINCT(Main.DiscSum)) / (
												SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
											)
										)
									END
								)
							END
						) * 100
					) / 100
				) - (Lines.GrossBuyPr * Lines.Quantity)
			) * 1
		END,
		-1 AS GrossPrcnt,
		PriceSys = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE (Lines.Price * Main.SysRate)
		END,
		SalesSys = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE (
				(
					(
						(
							CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
						) / Lines.NumPerMsr
					) * (Lines.Quantity * Lines.NumPerMsr)
				) - (
					(
						(
							(
								CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
							) / Lines.NumPerMsr
						) * (Lines.Quantity * Lines.NumPerMsr)
					) * (
						(
							CASE
								WHEN (
									(
										SUM(DISTINCT(Main.DocTotalSy)) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
									)
								) = 0 THEN 0
								ELSE (
									CASE
										(
											SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
										)
										WHEN 0 THEN 0
										ELSE (
											SUM(DISTINCT(Main.DiscSumSy)) / (
												SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
											)
										)
									END
								)
							END
						) * 100
					) / 100
				)
			) * 1
		END,
		GrossProfitSys = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN (
				0 - (
					(Lines.GrossBuyPr / Main.SysRate) * Lines.Quantity
				)
			) * 1
			ELSE (
				(
					(
						(
							CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
						) / Lines.NumPerMsr
					) * (Lines.Quantity * Lines.NumPerMsr)
				) - (
					(
						(
							(
								CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
							) / Lines.NumPerMsr
						) * (Lines.Quantity * Lines.NumPerMsr)
					) * (
						(
							CASE
								WHEN (
									(
										SUM(DISTINCT(Main.DocTotalSy)) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
									)
								) = 0 THEN 0
								ELSE (
									CASE
										(
											SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
										)
										WHEN 0 THEN 0
										ELSE (
											SUM(DISTINCT(Main.DiscSumSy)) / (
												SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
											)
										)
									END
								)
							END
						) * 100
					) / 100
				) - (
					(Lines.GrossBuyPr * Main.SysRate) * Lines.Quantity
				)
			) * 1
		END,
		-1 AS GrossPrcntSys,
		Lines.LineNum,
		GrossCalc = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE (
				SELECT (
						(
							(
								(
									CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
								) / Lines.NumPerMsr
							) * (Lines.Quantity * Lines.NumPerMsr)
						) - (
							(
								(
									(
										CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
									) / Lines.NumPerMsr
								) * (Lines.Quantity * Lines.NumPerMsr)
							) * (
								(
									CASE
										WHEN (
											SUM(DISTINCT(Main.DocTotal)) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
										) = 0 THEN 0
										ELSE (
											CASE
												(
													SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
												)
												WHEN 0 THEN 0
												ELSE (
													SUM(DISTINCT(Main.DiscSum)) / (
														SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
													)
												)
											END
										)
									END
								) * 100
							) / 100
						)
					) * 1
			)
		END,
		GrossCalcSys = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE (
				SELECT (
						(
							(
								(
									CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
								) / Lines.NumPerMsr
							) * (Lines.Quantity * Lines.NumPerMsr)
						) - (
							(
								(
									(
										CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
									) / Lines.NumPerMsr
								) * (Lines.Quantity * Lines.NumPerMsr)
							) * (
								(
									CASE
										WHEN (
											(
												SUM(DISTINCT(Main.DocTotalSy)) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
											)
										) = 0 THEN 0
										ELSE (
											CASE
												(
													SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
												)
												WHEN 0 THEN 0
												ELSE (
													SUM(DISTINCT(Main.DiscSumSy)) / (
														SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
													)
												)
											END
										)
									END
								) * 100
							) / 100
						)
					) * 1
			)
		END,
		Cards.GroupCode,
		ItmGrp.ItmsGrpCod
	FROM OINV Main WITH (NOLOCK)
		INNER JOIN INV1 Lines WITH (NOLOCK) ON Main.DocEntry = Lines.DocEntry
		INNER JOIN OITM Items WITH (NOLOCK) ON Items.ItemCode = Lines.ItemCode
		INNER JOIN OITB ItmGrp WITH (NOLOCK) ON ItmGrp.ItmsGrpCod = Items.ItmsGrpCod
		INNER JOIN OCRD Cards WITH (NOLOCK) ON Cards.CardCode = Main.CardCode
		INNER JOIN OSLP Slp WITH (NOLOCK) ON Slp.SlpCode = Main.SlpCode
		LEFT OUTER JOIN (
			SELECT WT0.DocEntry,
				WT0.WtSum,
				WT0.WtSumSC
			FROM OINV WT0
				INNER JOIN INV5 WT1 WITH (NOLOCK) ON WT1.AbsEntry = WT0.DocEntry
			WHERE WT1.Category = 'I'
			GROUP BY WT0.DocEntry,
				WT0.WtSum,
				WT0.WtSumSC
		) WT ON WT.DocEntry = Main.DocEntry
	WHERE Main.Instance = 0
		AND Main.Canceled <> 'C'
		AND Lines.Quantity <> 0
	GROUP BY Main.DocEntry,
		Slp.SlpCode,
		Slp.SlpName,
		Main.DocEntry,
		Main.DocNum,
		Main.DocDate,
		Items.ItemCode,
		Items.ItemName,
		Lines.TaxOnly,
		Lines.Price,
		Main.DocDueDate,
		Cards.CardCode,
		Cards.CardName,
		Cards.ProjectCod,
		Lines.GrossBuyPr,
		Lines.Quantity,
		Main.SysRate,
		Lines.LineNum,
		Lines.NumPerMsr,
		Lines.OpenCreQty,
		Main.DocTotal,
		Main.VatSum,
		ItmGrp.ItmsGrpNam,
		Lines.usebaseun,
		WT.WtSum,
		WT.WtSumSC,
		Lines.BaseType,
		Lines.NoInvtryMv,
		Cards.GroupCode,
		ItmGrp.ItmsGrpNam,
		ItmGrp.ItmsGrpCod



	UNION ALL
/*
------------------------------------------------------------------------------------------------------------
Part 2: Regular Credit Memos (Returns) 
------------------------------------------------------------------------------------------------------------
*/
	Select Main.DocEntry,
		Main.DocNum,
		Slp.SlpCode,
		Slp.SlpName,
		Cards.CardCode,
		Cards.CardName,
		Cards.ProjectCod,
		Main.DocDate,
		Main.DocDueDate,
		Items.ItemCode,
		ItmGrp.ItmsGrpNam,
		Items.ItemName,
		Quantity = CASE
			Lines.NoInvtryMv
			WHEN 'Y' THEN 0
			ELSE CASE
				Lines.usebaseun
				WHEN 'Y' THEN Lines.Quantity * -1
				ELSE (Lines.Quantity * Lines.NumPerMsr) * -1
			END
		END,
		Price = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE Lines.Price
		END,
		Sales = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE (
				(
					(
						(
							CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
						) / Lines.NumPerMsr
					) * (Lines.Quantity * Lines.NumPerMsr)
				) - (
					(
						(
							(
								CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
							) / Lines.NumPerMsr
						) * (Lines.Quantity * Lines.NumPerMsr)
					) * (
						(
							CASE
								WHEN (
									SUM(DISTINCT(Main.DocTotal)) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
								) = 0 THEN 0
								ELSE (
									CASE
										(
											SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
										)
										WHEN 0 THEN 0
										ELSE (
											SUM(DISTINCT(Main.DiscSum)) / (
												SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
											)
										)
									END
								)
							END
						) * 100
					) / 100
				)
			) * -1
		END,
		GrossProfit = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN (0 - (Lines.GrossBuyPr * Lines.Quantity)) * -1
			ELSE (
				(
					(
						(
							CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
						) / Lines.NumPerMsr
					) * (Lines.Quantity * Lines.NumPerMsr)
				) - (
					(
						(
							(
								CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
							) / Lines.NumPerMsr
						) * (Lines.Quantity * Lines.NumPerMsr)
					) * (
						(
							CASE
								WHEN (
									SUM(DISTINCT(Main.DocTotal)) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
								) = 0 THEN 0
								ELSE (
									CASE
										(
											SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
										)
										WHEN 0 THEN 0
										ELSE (
											SUM(DISTINCT(Main.DiscSum)) / (
												SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
											)
										)
									END
								)
							END
						) * 100
					) / 100
				) - (Lines.GrossBuyPr * Lines.Quantity)
			) * -1
		END,
		-1 AS GrossPrcnt,
		PriceSys = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE (Lines.Price * Main.SysRate)
		END,
		SalesSys = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE (
				(
					(
						(
							CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
						) / Lines.NumPerMsr
					) * (Lines.Quantity * Lines.NumPerMsr)
				) - (
					(
						(
							(
								CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
							) / Lines.NumPerMsr
						) * (Lines.Quantity * Lines.NumPerMsr)
					) * (
						(
							CASE
								WHEN (
									(
										SUM(DISTINCT(Main.DocTotalSy)) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
									)
								) = 0 THEN 0
								ELSE (
									CASE
										(
											SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
										)
										WHEN 0 THEN 0
										ELSE (
											SUM(DISTINCT(Main.DiscSumSy)) / (
												SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
											)
										)
									END
								)
							END
						) * 100
					) / 100
				)
			) * -1
		END,
		GrossProfitSys = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN (
				0 - (
					(Lines.GrossBuyPr / Main.SysRate) * Lines.Quantity
				)
			) * -1
			ELSE (
				(
					(
						(
							CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
						) / Lines.NumPerMsr
					) * (Lines.Quantity * Lines.NumPerMsr)
				) - (
					(
						(
							(
								CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
							) / Lines.NumPerMsr
						) * (Lines.Quantity * Lines.NumPerMsr)
					) * (
						(
							CASE
								WHEN (
									(
										SUM(DISTINCT(Main.DocTotalSy)) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
									)
								) = 0 THEN 0
								ELSE (
									CASE
										(
											SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
										)
										WHEN 0 THEN 0
										ELSE (
											SUM(DISTINCT(Main.DiscSumSy)) / (
												SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
											)
										)
									END
								)
							END
						) * 100
					) / 100
				) - (
					(Lines.GrossBuyPr * Main.SysRate) * Lines.Quantity
				)
			) * -1
		END,
		-1 AS GrossPrcntSys,
		Lines.LineNum,
		GrossCalc = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE (
				SELECT (
						(
							(
								(
									CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
								) / Lines.NumPerMsr
							) * (Lines.Quantity * Lines.NumPerMsr)
						) - (
							(
								(
									(
										CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
									) / Lines.NumPerMsr
								) * (Lines.Quantity * Lines.NumPerMsr)
							) * (
								(
									CASE
										WHEN (
											SUM(DISTINCT(Main.DocTotal)) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
										) = 0 THEN 0
										ELSE (
											CASE
												(
													SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
												)
												WHEN 0 THEN 0
												ELSE (
													SUM(DISTINCT(Main.DiscSum)) / (
														SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
													)
												)
											END
										)
									END
								) * 100
							) / 100
						)
					) * -1
			)
		END,
		GrossCalcSys = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE (
				SELECT (
						(
							(
								(
									CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
								) / Lines.NumPerMsr
							) * (Lines.Quantity * Lines.NumPerMsr)
						) - (
							(
								(
									(
										CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
									) / Lines.NumPerMsr
								) * (Lines.Quantity * Lines.NumPerMsr)
							) * (
								(
									CASE
										WHEN (
											(
												SUM(DISTINCT(Main.DocTotalSy)) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
											)
										) = 0 THEN 0
										ELSE (
											CASE
												(
													SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
												)
												WHEN 0 THEN 0
												ELSE (
													SUM(DISTINCT(Main.DiscSumSy)) / (
														SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
													)
												)
											END
										)
									END
								) * 100
							) / 100
						)
					) * -1
			)
		END,
		Cards.GroupCode,
		ItmGrp.ItmsGrpCod
	FROM ORIN Main WITH (NOLOCK)
		INNER JOIN RIN1 Lines WITH (NOLOCK) ON Main.DocEntry = Lines.DocEntry
		INNER JOIN OITM Items WITH (NOLOCK) ON Items.ItemCode = Lines.ItemCode
		INNER JOIN OITB ItmGrp WITH (NOLOCK) ON ItmGrp.ItmsGrpCod = Items.ItmsGrpCod
		INNER JOIN OCRD Cards WITH (NOLOCK) ON Cards.CardCode = Main.CardCode
		INNER JOIN OSLP Slp WITH (NOLOCK) ON Slp.SlpCode = Main.SlpCode
		LEFT OUTER JOIN (
			SELECT WT0.DocEntry,
				WT0.WtSum,
				WT0.WtSumSC
			FROM ORIN WT0
				INNER JOIN RIN5 WT1 WITH (NOLOCK) ON WT1.AbsEntry = WT0.DocEntry
			WHERE WT1.Category = 'I'
			GROUP BY WT0.DocEntry,
				WT0.WtSum,
				WT0.WtSumSC
		) WT ON WT.DocEntry = Main.DocEntry
	WHERE Main.Instance = 0
		AND Main.Canceled <> 'C'
		AND NOT EXISTS (
			SELECT 1
			FROM RIN1 T1
			WHERE T1.DocEntry = Main.DocEntry
				AND T1.BaseType = 203
		)
	GROUP BY Main.DocEntry,
		Slp.SlpCode,
		Slp.SlpName,
		Main.DocEntry,
		Main.DocNum,
		Main.DocDate,
		Items.ItemCode,
		Items.ItemName,
		Lines.TaxOnly,
		Lines.Price,
		Main.DocDueDate,
		Cards.CardCode,
		Cards.CardName,
		Cards.ProjectCod,
		Lines.GrossBuyPr,
		Lines.Quantity,
		Main.SysRate,
		Lines.LineNum,
		Lines.NumPerMsr,
		Lines.OpenCreQty,
		Main.DocTotal,
		Main.VatSum,
		Lines.usebaseun,
		WT.WtSum,
		WT.WtSumSC,
		Lines.BaseType,
		Lines.NoInvtryMv,
		Cards.GroupCode,
		ItmGrp.ItmsGrpCod,
		ItmGrp.ItmsGrpNam
	UNION ALL
/*
------------------------------------------------------------------------------------------------------------
Part 2: Canceled Credit Memos
------------------------------------------------------------------------------------------------------------
*/
	Select Main.DocEntry,
		Main.DocNum,
		Slp.SlpCode,
		Slp.SlpName,
		Cards.CardCode,
		Cards.CardName,
		Cards.ProjectCod,
		Main.DocDate,
		Main.DocDueDate,
		Items.ItemCode,
		ItmGrp.ItmsGrpNam,
		Items.ItemName,
		Quantity = CASE
			Lines.NoInvtryMv
			WHEN 'Y' THEN 0
			ELSE CASE
				Lines.usebaseun
				WHEN 'Y' THEN Lines.Quantity * 1
				ELSE (Lines.Quantity * Lines.NumPerMsr) * 1
			END
		END,
		Price = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE Lines.Price
		END,
		Sales = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE (
				(
					(
						(
							CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
						) / Lines.NumPerMsr
					) * (Lines.Quantity * Lines.NumPerMsr)
				) - (
					(
						(
							(
								CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
							) / Lines.NumPerMsr
						) * (Lines.Quantity * Lines.NumPerMsr)
					) * (
						(
							CASE
								WHEN (
									SUM(DISTINCT(Main.DocTotal)) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
								) = 0 THEN 0
								ELSE (
									CASE
										(
											SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
										)
										WHEN 0 THEN 0
										ELSE (
											SUM(DISTINCT(Main.DiscSum)) / (
												SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
											)
										)
									END
								)
							END
						) * 100
					) / 100
				)
			) * 1
		END,
		GrossProfit = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN (0 - (Lines.GrossBuyPr * Lines.Quantity)) * 1
			ELSE (
				(
					(
						(
							CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
						) / Lines.NumPerMsr
					) * (Lines.Quantity * Lines.NumPerMsr)
				) - (
					(
						(
							(
								CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
							) / Lines.NumPerMsr
						) * (Lines.Quantity * Lines.NumPerMsr)
					) * (
						(
							CASE
								WHEN (
									SUM(DISTINCT(Main.DocTotal)) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
								) = 0 THEN 0
								ELSE (
									CASE
										(
											SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
										)
										WHEN 0 THEN 0
										ELSE (
											SUM(DISTINCT(Main.DiscSum)) / (
												SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
											)
										)
									END
								)
							END
						) * 100
					) / 100
				) - (Lines.GrossBuyPr * Lines.Quantity)
			) * 1
		END,
		-1 AS GrossPrcnt,
		PriceSys = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE (Lines.Price * Main.SysRate)
		END,
		SalesSys = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE (
				(
					(
						(
							CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
						) / Lines.NumPerMsr
					) * (Lines.Quantity * Lines.NumPerMsr)
				) - (
					(
						(
							(
								CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
							) / Lines.NumPerMsr
						) * (Lines.Quantity * Lines.NumPerMsr)
					) * (
						(
							CASE
								WHEN (
									(
										SUM(DISTINCT(Main.DocTotalSy)) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
									)
								) = 0 THEN 0
								ELSE (
									CASE
										(
											SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
										)
										WHEN 0 THEN 0
										ELSE (
											SUM(DISTINCT(Main.DiscSumSy)) / (
												SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
											)
										)
									END
								)
							END
						) * 100
					) / 100
				)
			) * 1
		END,
		GrossProfitSys = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN (
				0 - (
					(Lines.GrossBuyPr / Main.SysRate) * Lines.Quantity
				)
			) * 1
			ELSE (
				(
					(
						(
							CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
						) / Lines.NumPerMsr
					) * (Lines.Quantity * Lines.NumPerMsr)
				) - (
					(
						(
							(
								CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
							) / Lines.NumPerMsr
						) * (Lines.Quantity * Lines.NumPerMsr)
					) * (
						(
							CASE
								WHEN (
									(
										SUM(DISTINCT(Main.DocTotalSy)) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
									)
								) = 0 THEN 0
								ELSE (
									CASE
										(
											SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
										)
										WHEN 0 THEN 0
										ELSE (
											SUM(DISTINCT(Main.DiscSumSy)) / (
												SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
											)
										)
									END
								)
							END
						) * 100
					) / 100
				) - (
					(Lines.GrossBuyPr * Main.SysRate) * Lines.Quantity
				)
			) * 1
		END,
		-1 AS GrossPrcntSys,
		Lines.LineNum,
		GrossCalc = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE (
				SELECT (
						(
							(
								(
									CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
								) / Lines.NumPerMsr
							) * (Lines.Quantity * Lines.NumPerMsr)
						) - (
							(
								(
									(
										CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
									) / Lines.NumPerMsr
								) * (Lines.Quantity * Lines.NumPerMsr)
							) * (
								(
									CASE
										WHEN (
											SUM(DISTINCT(Main.DocTotal)) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
										) = 0 THEN 0
										ELSE (
											CASE
												(
													SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
												)
												WHEN 0 THEN 0
												ELSE (
													SUM(DISTINCT(Main.DiscSum)) / (
														SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
													)
												)
											END
										)
									END
								) * 100
							) / 100
						)
					) * 1
			)
		END,
		GrossCalcSys = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE (
				SELECT (
						(
							(
								(
									CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
								) / Lines.NumPerMsr
							) * (Lines.Quantity * Lines.NumPerMsr)
						) - (
							(
								(
									(
										CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
									) / Lines.NumPerMsr
								) * (Lines.Quantity * Lines.NumPerMsr)
							) * (
								(
									CASE
										WHEN (
											(
												SUM(DISTINCT(Main.DocTotalSy)) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
											)
										) = 0 THEN 0
										ELSE (
											CASE
												(
													SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
												)
												WHEN 0 THEN 0
												ELSE (
													SUM(DISTINCT(Main.DiscSumSy)) / (
														SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
													)
												)
											END
										)
									END
								) * 100
							) / 100
						)
					) * 1
			)
		END,
		Cards.GroupCode,
		ItmGrp.ItmsGrpCod
	FROM ORIN Main WITH (NOLOCK)
		INNER JOIN RIN1 Lines WITH (NOLOCK) ON Main.DocEntry = Lines.DocEntry
		INNER JOIN OITM Items WITH (NOLOCK) ON Items.ItemCode = Lines.ItemCode
		INNER JOIN OITB ItmGrp WITH (NOLOCK) ON ItmGrp.ItmsGrpCod = Items.ItmsGrpCod
		INNER JOIN OCRD Cards WITH (NOLOCK) ON Cards.CardCode = Main.CardCode
		INNER JOIN OSLP Slp WITH (NOLOCK) ON Slp.SlpCode = Main.SlpCode
		LEFT OUTER JOIN (
			SELECT WT0.DocEntry,
				WT0.WtSum,
				WT0.WtSumSC
			FROM ORIN WT0
				INNER JOIN RIN5 WT1 WITH (NOLOCK) ON WT1.AbsEntry = WT0.DocEntry
			WHERE WT1.Category = 'I'
			GROUP BY WT0.DocEntry,
				WT0.WtSum,
				WT0.WtSumSC
		) WT ON WT.DocEntry = Main.DocEntry
	WHERE Main.Instance = 0
		AND Main.Canceled = 'C'
		AND NOT EXISTS (
			SELECT 1
			FROM RIN1 T1,
				RIN1 T2
			WHERE T1.DocEntry = Main.DocEntry
				AND T1.BaseType = 14
				AND T1.BaseEntry = T2.DocEntry
				AND T2.BaseType = 203
		)
	GROUP BY Main.DocEntry,
		Slp.SlpCode,
		Slp.SlpName,
		Main.DocEntry,
		Main.DocNum,
		Main.DocDate,
		Items.ItemCode,
		Items.ItemName,
		Lines.TaxOnly,
		Lines.Price,
		Main.DocDueDate,
		Cards.CardCode,
		Cards.CardName,
		Cards.ProjectCod,
		Lines.GrossBuyPr,
		Lines.Quantity,
		Main.SysRate,
		Lines.LineNum,
		Lines.NumPerMsr,
		Lines.OpenCreQty,
		Main.DocTotal,
		Main.VatSum,
		Lines.usebaseun,
		WT.WtSum,
		WT.WtSumSC,
		ItmGrp.ItmsGrpNam,
		Lines.BaseType,
		Lines.NoInvtryMv,
		Cards.GroupCode,
		ItmGrp.ItmsGrpCod
/*
------------------------------------------------------------------------------------------------------------
Part 4: Canceled Invoices
------------------------------------------------------------------------------------------------------------
*/
	UNION ALL
	Select Main.DocEntry,
		Main.DocNum,
		Slp.SlpCode,
		Slp.SlpName,
		Cards.CardCode,
		Cards.CardName,
		Cards.ProjectCod,
		Main.DocDate,
		Main.DocDueDate,
		Items.ItemCode,
		ItmGrp.ItmsGrpNam,
		Items.ItemName,
		Quantity = CASE
			Lines.NoInvtryMv
			WHEN 'Y' THEN 0
			ELSE CASE
				Lines.usebaseun
				WHEN 'Y' THEN Lines.Quantity * -1
				ELSE (Lines.Quantity * Lines.NumPerMsr) * -1
			END
		END,
		Price = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE Lines.Price
		END,
		Sales = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE (
				(
					(
						(
							CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
						) / Lines.NumPerMsr
					) * (Lines.Quantity * Lines.NumPerMsr)
				) - (
					(
						(
							(
								CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
							) / Lines.NumPerMsr
						) * (Lines.Quantity * Lines.NumPerMsr)
					) * (
						(
							CASE
								WHEN (
									SUM(DISTINCT(Main.DocTotal)) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
								) = 0 THEN 0
								ELSE (
									CASE
										(
											SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
										)
										WHEN 0 THEN 0
										ELSE (
											SUM(DISTINCT(Main.DiscSum)) / (
												SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
											)
										)
									END
								)
							END
						) * 100
					) / 100
				)
			) * -1
		END,
		GrossProfit = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN (0 - (Lines.GrossBuyPr * Lines.Quantity)) * -1
			ELSE (
				(
					(
						(
							CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
						) / Lines.NumPerMsr
					) * (Lines.Quantity * Lines.NumPerMsr)
				) - (
					(
						(
							(
								CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
							) / Lines.NumPerMsr
						) * (Lines.Quantity * Lines.NumPerMsr)
					) * (
						(
							CASE
								WHEN (
									SUM(DISTINCT(Main.DocTotal)) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
								) = 0 THEN 0
								ELSE (
									CASE
										(
											SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
										)
										WHEN 0 THEN 0
										ELSE (
											SUM(DISTINCT(Main.DiscSum)) / (
												SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
											)
										)
									END
								)
							END
						) * 100
					) / 100
				) - (Lines.GrossBuyPr * Lines.Quantity)
			) * -1
		END,
		-1 AS GrossPrcnt,
		PriceSys = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE (Lines.Price * Main.SysRate)
		END,
		SalesSys = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE (
				(
					(
						(
							CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
						) / Lines.NumPerMsr
					) * (Lines.Quantity * Lines.NumPerMsr)
				) - (
					(
						(
							(
								CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
							) / Lines.NumPerMsr
						) * (Lines.Quantity * Lines.NumPerMsr)
					) * (
						(
							CASE
								WHEN (
									(
										SUM(DISTINCT(Main.DocTotalSy)) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
									)
								) = 0 THEN 0
								ELSE (
									CASE
										(
											SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
										)
										WHEN 0 THEN 0
										ELSE (
											SUM(DISTINCT(Main.DiscSumSy)) / (
												SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
											)
										)
									END
								)
							END
						) * 100
					) / 100
				)
			) * -1
		END,
		GrossProfitSys = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN (
				0 - (
					(Lines.GrossBuyPr / Main.SysRate) * Lines.Quantity
				)
			) * -1
			ELSE (
				(
					(
						(
							CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
						) / Lines.NumPerMsr
					) * (Lines.Quantity * Lines.NumPerMsr)
				) - (
					(
						(
							(
								CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
							) / Lines.NumPerMsr
						) * (Lines.Quantity * Lines.NumPerMsr)
					) * (
						(
							CASE
								WHEN (
									(
										SUM(DISTINCT(Main.DocTotalSy)) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
									)
								) = 0 THEN 0
								ELSE (
									CASE
										(
											SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
										)
										WHEN 0 THEN 0
										ELSE (
											SUM(DISTINCT(Main.DiscSumSy)) / (
												SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
											)
										)
									END
								)
							END
						) * 100
					) / 100
				) - (
					(Lines.GrossBuyPr * Main.SysRate) * Lines.Quantity
				)
			) * -1
		END,
		-1 AS GrossPrcntSys,
		Lines.LineNum,
		GrossCalc = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE (
				SELECT (
						(
							(
								(
									CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
								) / Lines.NumPerMsr
							) * (Lines.Quantity * Lines.NumPerMsr)
						) - (
							(
								(
									(
										CAST(SUM(Lines.LineTotal) AS NUMERIC(19, 6)) / Lines.Quantity
									) / Lines.NumPerMsr
								) * (Lines.Quantity * Lines.NumPerMsr)
							) * (
								(
									CASE
										WHEN (
											SUM(DISTINCT(Main.DocTotal)) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
										) = 0 THEN 0
										ELSE (
											CASE
												(
													SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
												)
												WHEN 0 THEN 0
												ELSE (
													SUM(DISTINCT(Main.DiscSum)) / (
														SUM(DISTINCT(Main.DocTotal)) + SUM(DISTINCT(Main.DpmAmnt)) + ISNULL(WT.WtSum, 0.0) - SUM(DISTINCT(Main.VatSum)) - SUM(DISTINCT(Main.TotalExpns)) + SUM(DISTINCT(Main.DiscSum))
													)
												)
											END
										)
									END
								) * 100
							) / 100
						)
					) * -1
			)
		END,
		GrossCalcSys = CASE
			Lines.TaxOnly
			WHEN 'Y' THEN 0
			ELSE (
				SELECT (
						(
							(
								(
									CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
								) / Lines.NumPerMsr
							) * (Lines.Quantity * Lines.NumPerMsr)
						) - (
							(
								(
									(
										CAST(SUM(Lines.TotalSumSy) AS NUMERIC(19, 6)) / Lines.Quantity
									) / Lines.NumPerMsr
								) * (Lines.Quantity * Lines.NumPerMsr)
							) * (
								(
									CASE
										WHEN (
											(
												SUM(DISTINCT(Main.DocTotalSy)) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
											)
										) = 0 THEN 0
										ELSE (
											CASE
												(
													SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
												)
												WHEN 0 THEN 0
												ELSE (
													SUM(DISTINCT(Main.DiscSumSy)) / (
														SUM(DISTINCT(Main.DocTotalSy)) + SUM(DISTINCT(Main.DpmAmntSC)) + ISNULL(WT.WtSumSC, 0.0) - SUM(DISTINCT(Main.VatSumSy)) - SUM(DISTINCT(Main.TotalExpSC)) + SUM(DISTINCT(Main.DiscSumSy))
													)
												)
											END
										)
									END
								) * 100
							) / 100
						)
					) * -1
			)
		END,
		Cards.GroupCode,
		ItmGrp.ItmsGrpCod
	FROM OINV Main WITH (NOLOCK)
		INNER JOIN INV1 Lines WITH (NOLOCK) ON Main.DocEntry = Lines.DocEntry
		INNER JOIN OITM Items WITH (NOLOCK) ON Items.ItemCode = Lines.ItemCode
		INNER JOIN OITB ItmGrp WITH (NOLOCK) ON ItmGrp.ItmsGrpCod = Items.ItmsGrpCod
		INNER JOIN OCRD Cards WITH (NOLOCK) ON Cards.CardCode = Main.CardCode
		INNER JOIN OSLP Slp WITH (NOLOCK) ON Slp.SlpCode = Main.SlpCode
		LEFT OUTER JOIN (
			SELECT WT0.DocEntry,
				WT0.WtSum,
				WT0.WtSumSC
			FROM OINV WT0
				INNER JOIN INV5 WT1 WITH (NOLOCK) ON WT1.AbsEntry = WT0.DocEntry
			WHERE WT1.Category = 'I'
			GROUP BY WT0.DocEntry,
				WT0.WtSum,
				WT0.WtSumSC
		) WT ON WT.DocEntry = Main.DocEntry
	WHERE Main.Instance = 0
		AND Main.Canceled = 'C'
		AND Lines.Quantity <> 0
	GROUP BY Main.DocEntry,
		Slp.SlpCode,
		Slp.SlpName,
		Main.DocEntry,
		Main.DocNum,
		Main.DocDate,
		Items.ItemCode,
		Items.ItemName,
		Lines.TaxOnly,
		Lines.Price,
		Main.DocDueDate,
		Cards.CardCode,
		Cards.CardName,
		Cards.ProjectCod,
		Lines.GrossBuyPr,
		Lines.Quantity,
		Main.SysRate,
		Lines.LineNum,
		Lines.NumPerMsr,
		Lines.OpenCreQty,
		Main.DocTotal,
		Main.VatSum,
		ItmGrp.ItmsGrpNam,
		Lines.usebaseun,
		WT.WtSum,
		WT.WtSumSC,
		Lines.BaseType,
		Lines.NoInvtryMv,
		Cards.GroupCode,
		ItmGrp.ItmsGrpCod
	
) T0 LEFT JOIN OCRD T1 WITH (NOLOCK) ON T1.CardCode = T0.CardCode
LEFT JOIN OSLP T2 WITH (NOLOCK) ON T2.SlpCode = T1.SlpCode
```