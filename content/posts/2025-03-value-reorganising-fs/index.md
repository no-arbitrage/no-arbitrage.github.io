---
title: Reorganising Financial Statements for Valuation Analysis
summary: 
categories: ["Finance",]
series: ["Value Fundamentals"]
series_order: 4
tags: ["post","finance", "valuation"]
#externalUrl: ""
#showSummary: true
date: 2025-03-11
draft: false
---

{{< lead >}}
How many valuation methods do you know?
{{< /lead >}}

## Introduction
By analysing these concepts, we will go through a process - adjusting/ reorganising the balance sheet to arrive at a **Economic Balance Sheet** in corporate finance perspectives.

## Balance Sheet 
Total Asset = Total Funds Invested
***Total Funds Invested =***
* Non-operating asset
	* Derivatives, 
	* Excess real estate,
	* Discontinued operations,
	* Receivables from [[Financial subsidiaries]] (credit card receivables), 
	* Excess cash and marketable securities, 
	* Nonconsolidated subsidiaries (investments in associates) and Equity Investments
	* [[Overfunded pension assets]]
	* [[Tax loss carry-forwards (NOLs)]]
* Invested Capital, incl. Goodwill
	* Goodwill and acquired intangibles
		* ROIC with goodwill measures a company’s ability to create value after paying acquisition premiums. 
		* ROIC without goodwill measures the competitiveness of the underlying business.
		* Two adjustments needed to evaluate the effect of goodwill properly
			* subtract deferred-tax liabilities related to the amortization of acquired intangibles (due to write-ups) [[Understanding Intangible write-ups in M&A]]
			* Add back cumulative amortization and impairment.
			  goodwill do not wear out, nor are they replaceable.
	* Invested Capital, exc. Good Will 
		* Operating Working Capital
			* Operating Current Asset
				* working cash balances, 
				* trade accounts receivable, 
				* inventory, and 
				* prepaid expenses. 
				* Specifically excluded are **excess cash and marketable securities**
			* Minus Operating Liabilities
				* Liabilities that are related to the ongoing operations of the firm.
					* suppliers (accounts payable), 
					* employees (accrued salaries), 
					* customers (as either prepayments deferred membership fees), and
					* government (income taxes payable).
				* a liability is deemed operating rather than financial should be netted from operating assets and incorporated into free cash flow. 
				* Interest-bearing liabilities are nonoperating and should not be netted from operating assets, but rather valued separately
				* 
		* PP&E & Other Capitalised Investments
			* Book value of property, plant, and equipment net of accumulated depreciation in operating assets.

***Sources of Capital =*** 
* **Debt**:  
	* Interest-bearing debt from banks and public capital markets 
* **Debt equivalents**:  
	* Off-balance-sheet debt and 
	* One-time debts owed to others that are not part of ongoing operations e.g., 
		* severance payments as part of a restructuring, 
		* reserves for plant decommissioning and for restructuring
		* an unfunded pension liability, unfunded postretirement medical liabilities
		* expected environmental remediation following a plant closure
* **Equity** 
	* Common stock
	* additional paid-in capital
	* retained earnings
	* Treasury stock (repurchased as an offset account)
	* accumulated other comprehensive income
		* currency adjustments, 
		* aggregate unrealized gains and losses from liquid assets whose value has changed but that have not yet been sold, and 
		* pension plan fluctuations within a certain band.
* **Equity equivalents** : 
	* Balance sheet accounts that arise because of noncash adjustments to retained earnings; similar to debt equivalents but not deducted from enterprise value to determine equity value. e.g.,
	* income-smoothing provisions
	* most deferred-tax accounts (Separating deferred taxes into operating and nonoperating items can be challenging)
		* deferred-tax assets related to past losses should be classified as a nonoperating asset and valued separately. not equity equivalents.
		* Deferred-tax liabilities related to amortization of acquired intangibles should be netted against acquired intangibles. not equity equivalents.
* **Hybrid securities**: 
	* Claims that have equity characteristics but are not yet part of owners’ equity (e.g., convertible debt and employee options)
* **Noncontrolling interest by other companies**: 
	* External shareholders’ minority position in any of the company's consolidated subsidiaries
	* If a noncontrolling interest exists, treat the balance sheet amount as a source of financing.
	* Treat the earnings attributable to any noncontrolling interest as a financing cash flow similar to dividends.
Summary of Balance Sheet Adjustments 
* So the key idea here is to 
	* separate the operating from non-operating; and then 
	* separating sources of capital to the Total Funds Invested.

## Income Statements
The goal of adjustments here is to 
1. Ensure that your profit calculation flows solely from operations.
2. Normalised statements
In many companies, nonoperating items are embedded within operating expenses. most common nonoperating items are related to 
* nonoperating portion of pension expense.
* embedded interest expenses from **operating leases**, 
* one-time restructuring charges hidden in the cost of sale
* material one-time items in operating expenses
After adjustments, estimating operating cash taxes (assuming an all-equity operating level) #ProcessToTaxReconciliation
## Cash Flow Statement
The goal is to estimate free cashflow to all investors. 
Total Cash Flow = 
* **Free Cash Flow (FCFF)** = 
	* NOPAT - net operating profit after taxes
		* after-tax operating profit available to all investors.
	* Plus Non-Cash Operating Expenses 
		* Depreciation & Amortisation
			* Only add back amortization de ducted from revenues to compute NOPAT, such as the amortization of capitalized software or purchased customer contracts. Do not add back the amortization from acquired intangibles and impairments to NOPAT because they were not subtracted from revenue in calculating NOPAT
		* share-based employee compensation
			* Because Since employees have a new claim on cash flows, this claim must be incorporated into the valuation, either as part of cash flow or as a separate calculation
	- Less Investments in Invested Capital (reinvestments)
		- Change in operating working capital.
		- Capital expenditures, net of disposals.
		- Change in capitalized operating leases.
		- Investment in goodwill and acquired intangibles.
		- Change in other long-term operating assets, net of long-term liabilities.
		For most assets and liabilities, the year-to-year change in a balance sheet account will suitably approximate net investment. This will not always be the case. Currency translations, acquisitions, write-offs, and accounting changes
* **Nonoperating cash flows**
	* Nonoperating income and expenses & taxes
	* Cash flow related to excess cash and marketable securities.
	* Cash flow from other nonoperating assets.
