---
title: "Understanding Customer Churn and Growth with SQL"
summary: This analysis serves as the very first step in understanding the customers behaviours.
series: ["SQL Reports for SAP"]
series_order: 5
tags: ["sql","reporting", "sap", "BI"]
heroStyle : "basic"
date: 2025-05-07T14:52:25+08:00
draft: false
---
In this post, I’m sharing a set of SQL scripts I’ve developed to analyse customer dynamics — specifically, how many customers are gained, lost, and how their behaviour impacts sales and gross profit over time. These scripts are designed to help commercial teams track the business impact of customer acquisition and attrition, distinguishing between absolute new customers and those tied to specific product categories.

The logic is modular, the calculations are transparent, and the aim is practical: to inform decision-making with clear visibility into customer lifecycle trends.

Let’s get into the code.
```sql
/*------------------------------------------------------------------------
# MEASURES
----------------------------------
- New Customers: counts the number of customers who are new.
- Lost Customers: counts the number of customers permanently lost.
----------------------------------
- Sales New Customers: returns the value of Sales Amount by filtering only the new customers.
- Sales Lost Customers (12M): computes the value of Sales Amount for 12 months prior to the start of the selected time period, filtering only customers permanently lost in the selected period.
----------------------------------
- Absolute: a customer is considered new the first time they buy a product, regardless of any filter present in the report.
- By category: a customer is considered new the first time they buy a product from any of the product categories selected in the report. If they buy two products of the same category then they are considered new only once, whereas if they buy two products of different categories then they are considered new twice (Not Implmented yet) 
 */
WITH CustomerWithDates AS (
  SELECT 
    T1.CardCode, 
    T1.CardName, 
    T1.ProjectCod, 
    MIN(T1.DocDate) [Date New Customer],
    EOMONTH( MIN(T1.DocDate),  0 ) [Month New Customer], 
    YEAR ( EOMONTH(  MIN(T1.DocDate), + 9 )) [FY New Customer], 
    CASE WHEN EOMONTH(  MAX(T1.DocDate), 2) > GETDATE() THEN NULL ELSE  EOMONTH(  MAX(T1.DocDate), 2) END [Date Lost Customer] ,
    --accounts inactive for 2 months are considered lost.
    CASE WHEN EOMONTH(  MAX(T1.DocDate), 2) > GETDATE() THEN NULL ELSE  YEAR ( EOMONTH (  MAX (T1.DocDate),  2 + 9  ) ) END [FY Lost Customer] 
  FROM 
    SalesAnalysis T1 WITH (NOLOCK)
  WHERE 
  T1.CANCELED != 'Y' 
  GROUP BY 
    T1.CardCode,  T1.CardName,  T1.ProjectCod

) -- SELECT
-- * 
-- FROM 
-- (
--   SELECT 
--   T0.ProjectCod, 
--   T0.FirstOrderFY, COUNT( DISTINCT  T0.CardCode ) [CountofNewCustomers]
--   FROM CustomerWithDates T0
--  GROUP BY T0.ProjectCod, T0.FirstOrderFY
--) T 
--PIVOT ( 
-- SUM( [CountofNewCustomers] ) FOR [ProjectCod] IN ( [FAW],[CRE],[WCT],[Retail],[EXP] ) 
--) AS P
--ORDER BY 1
, 
NewCustomerCount AS (
  SELECT 
    T0.ProjectCod, 
    T0.[FY New Customer], 
    COUNT(DISTINCT T0.CardCode) [CountofNewCustomers] 
  FROM 
    CustomerWithDates T0 
  GROUP BY 
    T0.ProjectCod, 
    T0.[FY New Customer]
), 
LostCustomerCount AS (
  SELECT 
    T0.ProjectCod, 
    T0.[FY Lost Customer], 
    COUNT(DISTINCT T0.CardCode) [CountofLostCustomers] 
  FROM 
    CustomerWithDates T0 
  WHERE 
    T0.[Date Lost Customer] < GETDATE() 
  GROUP BY 
    T0.ProjectCod, 
    T0.[FY Lost Customer]
), 
AccountSalesByYear AS (
  SELECT 
    T0.CardCode, 
    T0.CardName, 
    SalesFY = YEAR(
      EOMONTH(T0.DocDate, + 9)
    ), 
    T0.Sales [AccountSales], 
    T0.GrossProfit [AccountGrossProfit], 
    SUM(T0.Sales) OVER (
      PARTITION BY YEAR(
        EOMONTH(T0.DocDate, + 9)
      ) 
      ORDER BY 
        T0.DocDate ASC ROWS BETWEEN UNBOUNDED PRECEDING 
        AND UNBOUNDED FOLLOWING
    ) [TotalSales], 
    SUM(T0.GrossProfit) OVER (
      PARTITION BY YEAR(
        EOMONTH(T0.DocDate, + 9)
      ) 
      ORDER BY 
        T0.DocDate ASC ROWS BETWEEN UNBOUNDED PRECEDING 
        AND UNBOUNDED FOLLOWING
    ) [TotalGrossProfit] 
  FROM 
    SalesAnalysis T0 
  WHERE 
    T0.DocDate >= '2018/04/01'
), 
NewAccountSales AS (
  SELECT 
    T2.[FY New Customer], 
    T1.[TotalSales], 
    T1.TotalGrossProfit, 
    SUM(T1.AccountSales) [NewCustomerSales], 
    SUM(T1.AccountSales) / T1.[TotalSales] [ % of TotalSales], 
    SUM(T1.AccountGrossProfit) [NewCustomerGProfit], 
    SUM(T1.AccountGrossProfit) / T1.TotalGrossProfit [ % of TotalProfit] 
  FROM 
    AccountSalesByYear T1 
    JOIN CustomerWithDates T2 ON T2.[FY New Customer] = T1.SalesFY 
    AND T2.CardCode = T1.CardCode 
  GROUP BY 
    T2.[FY New Customer], 
    T1.[TotalSales], 
    T1.[TotalGrossProfit] --ORDER BY 1
    ), 
LostAccountWithSales AS (
  SELECT 
    T10.CardCode, 
    T10.CardName, 
    T10.SalesFY, 
    T10.TotalSales, 
    T10.TotalGrossProfit, 
    T10.AccountSales, 
    T10.AccountGrossProfit 
  FROM 
    (
      SELECT 
        T0.CardCode, 
        T0.CardName, 
        T0.SalesFY, 
        T0.[TotalSales], 
        T0.TotalGrossProfit, 
        SUM (T0.AccountSales) [AccountSales], 
        SUM(T0.[AccountGrossProfit]) [AccountGrossProfit] 
      FROM 
        AccountSalesByYear T0 
      GROUP BY 
        T0.CardCode, 
        T0.CardName, 
        T0.SalesFY, 
        T0.[TotalSales], 
        T0.TotalGrossProfit
    ) T10 
    JOIN CustomerWithDates T2 ON T2.[FY Lost Customer] = T10.SalesFY + 1 
    AND T2.CardCode = T10.CardCode
), 
LostSales AS (
  SELECT 
    T0.SalesFY, 
    SUM(T0.[AccountSales]) [Prev Year Sales of Lost Accounts], 
    T0.TotalSales, 
    SUM(T0.[AccountSales]) / T0.TotalSales [Pct of Sales], 
    SUM(T0.[AccountGrossProfit]) [Prev Year GP of Lost Accounts], 
    T0.TotalGrossProfit, 
    SUM(T0.[AccountGrossProfit])/ T0.TotalGrossProfit [Pct of GP] 
  FROM 
    LostAccountWithSales T0 --SELECT * LostAccountSales
  GROUP BY 
    T0.SalesFY, 
    T0.TotalSales, 
    T0.TotalGrossProfit
) 
SELECT 
  * 
FROM 
  CustomerWithDates




  SELECT 
    T0.CardCode, 
    T0.CardName, 
	
    SalesFY = YEAR(
      EOMONTH(T0.DocDate, + 9)
    ), 
    SUM(T0.Sales) [AccountSales], 
    SUM( T0.GrossProfit)  [AccountGrossProfit]
  FROM 
    SalesAnalysis T0 
	
  WHERE 
    T0.DocDate >= '2010/04/01'
GROUP BY T0.CardCode, 
    T0.CardName, 
	YEAR(
      EOMONTH(T0.DocDate, + 9)
    ) 
```
