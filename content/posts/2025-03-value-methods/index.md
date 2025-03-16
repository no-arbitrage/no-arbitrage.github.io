---
title: Valuation Methods
summary: 
categories: ["Finance","Valuation"]
series: ["Value Fundamentals"]
series_order: 4
tags: ["post","finance", "M&A"]
#externalUrl: ""
#showSummary: true
date: 2025-03-14
draft: false
---

{{< lead >}}
The financial term 'No-arbitrage' is also a pricing approach, upon which Black-Scholes-Merton (BSM) model was developed. It assumes that there are no risk-free profit opportunities in the market. 
{{< /lead >}}

In the realm of asset valuation, several primary approaches and methodologies are employed to determine an asset's fair value. 
<details>
<summary>All Corporate Finance Valuation Models that I've come across so far. </summary>

| **Valuation Approach**                | **Description**                                                                                        | **Best Suited For**                                                                                          | **Key Advantages**                                                             | **Key Limitations**                                                                     | Prominence |
|---------------------------------------|--------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------|-------------|
| **Discounted Cash Flow (DCF)**        | Fundamental valuation method using projected future cash flows discounted to present value. Including enterprise DCF, and equity DCF.          | Most general-purpose, applicable to various assets, especially stable companies with predictable cash flows. | Theoretically sound, considers time value of money and intrinsic value.        | Highly sensitive to assumptions (growth rates, discount rate, etc.), complex to model.  | ⭐⭐⭐⭐⭐       |
| **Enterprise DCF (FCFF)**             | Values the entire firm by discounting free cash flows before debt service.                             | Firms with stable operations and available financial projections.                                            | Ignores capital structure, making comparisons across firms easier.             | Requires accurate forecasting and WACC assumptions.                                     | ⭐⭐⭐⭐⭐       |
| **Equity DCF (FCFE)**                 | Values only the equity portion by discounting cash flows available to shareholders.                    | Firms with stable leverage or companies undergoing significant financial restructuring.                      | Focuses on shareholder value, useful for firms with predictable debt policies. | More sensitive to capital structure assumptions, more complex for leveraged firms.      | ⭐⭐⭐⭐⭐       |
| **Dividend Discount Model (DDM)**     | Values equity based on expected future dividends discounted to present value.                          | Mature companies with stable and predictable dividend payouts (e.g., utilities, consumer staples).           | Simple and direct for firms with consistent dividends.                         | Not useful for non-dividend-paying firms or companies with irregular dividends.         | ⭐⭐⭐⭐        |
| **Adjusted Present Value (APV)**      | Separates the value of an unlevered firm from financing effects (debt tax shield, financial distress). | Firms with changing capital structure, LBOs, highly leveraged companies.                                     | Explicitly models financing effects, useful for highly leveraged deals.        | More complex than DCF, requires detailed assumptions on tax shields and distress costs. | ⭐⭐⭐         |
| **Relative Valuation (Multiples)**    | Values a company based on market comparables (e.g., P/E, EV/EBITDA).                                   | Publicly traded firms, industries with ample comparables (tech, retail, industrials).                        | Quick, widely used by investors and analysts.                                  | Relies on accurate and relevant comparables, ignores firm-specific fundamentals.        | ⭐⭐⭐⭐⭐       |
| **Sum-Of-The-Parts (SOTP) Valuation** | Values a company by valuing its individual business segments separately.                               | Conglomerates, diversified businesses with distinct operating segments.                                      | Captures hidden value in diversified firms.                                    | Requires strong segment-level data, risk of mispricing synergies.                       | ⭐⭐⭐⭐⭐       |
| **Option Pricing Models**             | Values assets with embedded optionality (e.g., Black-Scholes, binomial models).                        | Startups, natural resources, R&D-heavy firms, distressed companies.                                          | Captures flexibility and optionality ignored in traditional models.            | Complex, requires strong assumptions about volatility and time to exercise.             | ⭐⭐          |
| **Leveraged Buyout (LBO) Analysis**   | Models a private equity firm’s acquisition of a target with significant debt financing.                | Private equity deals, highly leveraged transactions.                                                         | Assesses feasibility and return potential of an LBO.                           | Not a true valuation method; depends on deal structure and debt assumptions.            | ⭐⭐⭐         |
| **Liquidation Valuation**             | Values a company based on its breakup value in case of insolvency or distress.                         | Distressed firms, bankruptcy proceedings.                                                                    | Provides a floor value for assets.                                             | Ignores going-concern value, often undervalues assets.                                  | ⭐⭐          |
| **Economic Value Added (EVA)**        | Measures a company’s value creation by subtracting cost of capital from operating profits.             | Internal performance assessment, capital allocation decisions.                                               | Focuses on real economic profitability, aligns management incentives.          | Rarely used in investment decisions, complex adjustments required.                      | ⭐           |

</details>


## 1. Discounted Cash Flow (DCF) Analysis

This approach estimates the present value of an asset based on its expected future cash flows, discounted back using a rate that reflects the asset's risk. DCF methods can be tailored to specific valuation needs:

- **Free Cash Flow to Firm (FCFF):** Calculates the present value of cash flows available to all capital providers, discounted at the Weighted Average Cost of Capital (WACC).

- **Free Cash Flow to Equity (FCFE):** Focuses on cash flows available to equity shareholders after accounting for all expenses, reinvestments, and debt repayments, discounted at the cost of equity.

- **Dividend Discount Model (DDM):** Values a company based on the present value of its expected future dividends, suitable for firms with stable and predictable dividend payouts.

## 2. Adjusted Present Value (APV)

Rooted in Modigliani-Miller theorem, APV separates the value of an all-equity firm from the present value of financing benefits, such as tax shields from debt interest. This method is particularly useful for firms with changing capital structures.

## 3. Relative Valuation (Multiples)

This approach involves valuing an asset by comparing it to similar assets using valuation multiples derived from market data. Common multiples include:

- **Precedent Transaction Multiples:** Analyzes past transactions of comparable assets to determine valuation benchmarks.
    
- **Comparable Company Multiples:** Uses ratios like Price-to-Earnings (P/E), Enterprise Value-to-EBITDA (EV/EBITDA), and Price-to-Book (P/B) from similar publicly traded companies to estimate value.

## 4. Sum-of-the-Parts (SOTP) Valuation

This approach involves valuing each business unit or subsidiary of a conglomerate separately and summing them to obtain the total enterprise value.

## 5. Option Pricing Models

Used to value assets with option-like characteristics, this type of models include Black-Scholes with its various forms and binomial options pricing.

* **Contingent Claim Valuation**:  It's particularly applicable for valuing financial derivatives or investments with embedded options.

* **Real Options Valuation**: Extending option pricing theory to capital budgeting decisions, this method evaluates investment opportunities as options, recognizing managerial flexibility in decision-making under uncertainty. Equity under high uncertainty behaves much like a long call option and debt behaves much like a short put option.

## 6. Leveraged Buyout (LBO) Model

An LBO model assesses the acquisition of a company primarily financed through debt. The objective is to determine the potential returns to equity investors under various financial structures and operating scenarios. Key components of an LBO model include:

* **Source of Funds & Use of Funds:** Determining the optimal mix of debt and equity financing.

* **Cash Flow Projections:** Estimating future cash flows to service debt and provide returns to equity holders.

* **Debt Schedule & Repayment Waterfall**: some call it debt sculpture , including further components:
    * **Interest Expense**: Calculated based on the outstanding debt balance and interest rates.
    * **Debt Repayment**: Scheduled repayments of principal, often based on cash flow availability or mandatory amortization.
    * **Covenants**: Financial ratios or conditions that the company must maintain (e.g., debt-to-EBITDA ratio)

* **Exit Assumptions:** Planning the sale or public offering of the company after a holding period, typically 3-7 years, to realize investment gains.

* **Sensitive Analysis (e.g. Monte Carlo or Scenario Analysis)**

LBO models are extensively used by private equity firms to evaluate potential investments. However, recent trends indicate challenges in this approach due to rising interest rates and increased default rates among overleveraged companies. For instance, the private equity industry has faced scrutiny for practices that may destabilize acquired companies and broader economic systems.

Incorporating both LBO and liquidation valuation methodologies provides a comprehensive perspective on a company's valuation, encompassing both strategic acquisition scenarios and downside risk assessments.

## 7. Liquidation Valuation Approach

This approach estimates the net proceeds if a company's assets were sold individually, often under distressed conditions. It's particularly relevant in bankruptcy scenarios or when assessing a company's minimum value. The liquidation value is typically lower than the going concern value due to the expedited nature of asset sales and potential discounts applied. Understanding liquidation value is crucial for stakeholders to assess potential recoveries in worst-case scenarios.

## 8. Economic Value Added (EVA)

EVA measures a company's financial performance by subtracting the cost of capital from its operating profit. It reflects the true economic profit and is used to assess value creation beyond required returns. However it uses accounting data as model inputs, this fact instantly upsets financially minded me.