# LexEuropa Legal Group: Case & Lawyer Performance Dashboard (Power BI)

A Power BI dashboard for **LexEuropa Legal Group**, a fictional European law firm, built for the [FP20 Analytics ZoomCharts Challenge 39](https://zoomcharts.com/en/microsoft-power-bi-custom-visuals/challenges/fp20-analytics-challenge-39). It analyses **15,000 cases, 200 lawyers, 800 clients and 15 offices** (2024–2026) to help partners monitor performance, improve profitability, balance workload and understand what drives good outcomes.

> **Headline findings:** complex, high-risk cases are the firm's profit engine. Critical cases are 5% of volume but produce 57% of profit at a 60% margin. Client satisfaction depends almost entirely on the case outcome (8.5/10 when won vs 3.5 when lost), not on cost or speed. The gender profit gap is a seniority gap: profit per case is equal within each level, but men hold 9 of the 12 Partner roles.

![Executive Performance](images/01_executive_performance.png)

## Business questions (from the challenge brief)

1. Which practice areas and offices generate the highest revenue and profit?
2. How do case outcomes vary across departments and lawyers?
3. Which lawyers have the highest utilisation and workload?
4. How do risk, case complexity and duration affect profitability?
5. Which clients and practice areas contribute the most value?
6. How does client satisfaction relate to case performance?

## Dashboard pages

| Page | What it shows |
|---|---|
| **Executive Performance** | Revenue, profit, margin, win rate and case count with year-on-year arrows; revenue by practice area and office; revenue trend with drill-down (year → quarter → month → day); **ZoomCharts Drill Down Map** of office revenue; case-type table; a field parameter to switch the breakdown (Seniority, Client Region, Priority, Case Type, Department) |
| **Case Performance** | Win rate, risk score and margin KPIs; profit by complexity, practice area and client company size; **ZoomCharts Drill Down Combo** of case outcomes by practice area; practice-area table with days open, satisfaction and workload |
| **Lawyer Performance** | Utilisation, days open and win rate KPIs; outcomes (won, lost, settled, ongoing, withdrawn) per lawyer; satisfaction and win rate by age group, gender or seniority (field parameter); **Top 10 lawyers by a weighted performance score** |
| **Summary** | Written key findings and recommendations |

Every page has a Year slicer and KPI cards comparing each value with the previous year.

| | |
|---|---|
| ![Case Performance](images/02_case_performance.png) | ![Lawyer Performance](images/03_lawyer_performance.png) |

## Key findings

| Firm at a glance (2024–2026) | |
|---|---|
| Revenue | €2.80B |
| Profit | €1.59B (49.6% avg margin) |
| Win rate | 38.5% |
| Outstanding balance | €773M (27.6% of invoiced) |

1. **Growth:** 2025 was the growth year (cases +22%, revenue +79%). 2026 levelled off (revenue -3.1%). Margin stays at 49.5–49.7% every year.
2. **Practice areas & offices:** Regulatory Compliance (€433.8M) and M&A (€423.4M) lead on revenue. Frankfurt (€365.6M) and London (€358.6M) are the top offices, and Western Europe is the top client region. Margin is almost identical across practice areas (49.3–49.9%).
3. **Complexity & risk drive profit:** margin rises from 45% (Low complexity) to 60% (Critical), and from 45% to 58% as risk score increases. Critical cases are 5% of volume but 57% of profit.
4. **Lawyers:** win rate rises with seniority (Associate 35% → Partner 44%). Associates' low revenue per case comes from their case mix (71% Low complexity), not performance. All 12 lawyers above 40% utilisation are Partners.
5. **Gender gap = seniority gap:** profit per case is the same for men and women at each level, but men hold 9 of 12 Partner and 22 of 35 Counsel roles.
6. **Clients:** revenue is well spread (top 10 clients = 4.1%). Profit per case is similar for every company size. €773M is still unpaid.
7. **Satisfaction follows outcome:** 8.5 (won), 6.5 (settled), 3.5 (lost). There's virtually no link with margin, risk or duration.
8. **Watch:** win rate dropped to 34.6% in 2026, though 45% of 2026 cases are still open.

📄 **Full analysis with evidence tables and recommendations:** [docs/key_findings.md](docs/key_findings.md)

## Recommendations

1. Grow Critical and High complexity work, which earns the highest margins.
2. Relieve Partner workload by delegating Medium-complexity cases to Senior Associates.
3. Develop Associates on higher-complexity cases alongside Partners and Counsel.
4. Address the gender gap through promotion into Counsel and Partner roles.
5. Investigate the 2026 win-rate drop as cases close, starting with Intellectual Property (lowest win rate).
6. Collect the €773M outstanding balance.

## Data model & techniques

- **Star schema:** `Fact_Cases` linked to `Dim_Case`, `Dim_Lawyer`, `Dim_Client`, `Dim_Office` and `Dim_Date`.
- **60+ DAX measures**, including:
  - win rate on decided cases only
  - year-on-year comparisons with ▲/▼ arrows (`DATEADD`, `SAMEPERIODLASTYEAR`)
  - conditional-formatting colour measures
  - a **weighted lawyer performance score** (min–max normalised revenue, win rate and satisfaction)
  - client revenue rank and concentration (`RANKX`, `TOPN`)
  - a case pressure index with automatic action recommendations
  - a 3-month moving average
- **Field parameters** to switch chart dimensions.
- **ZoomCharts Drill Down visuals:** a map (office revenue) and a combo chart (case outcomes by practice area).

📄 **All measures:** [docs/dax_measures.md](docs/dax_measures.md)

## Tools & skills

Power BI Desktop · DAX · Power Query · star-schema data modelling · time intelligence · field parameters · ZoomCharts Drill Down visuals · conditional formatting · data storytelling

## How to open

1. Install [Power BI Desktop](https://www.microsoft.com/power-bi/desktop) (free, Windows).
2. Download the dataset from the [challenge page](https://zoomcharts.com/en/microsoft-power-bi-custom-visuals/challenges/fp20-analytics-challenge-39).
3. Open `LexEuropa_Legal_Dashboard.pbix`. If asked, go to **Transform data → Data source settings** and point the source to your copy of the Excel file.

## Repository structure

```
├── README.md
├── LexEuropa_Legal_Dashboard.pbix     # Power BI dashboard
├── docs/
│   ├── key_findings.md                # full analysis & recommendations
│   └── dax_measures.md                # data model, parameters, DAX
└── images/                            # dashboard screenshots
```

*Dataset: FP20 Analytics Challenge 39 (synthetic data). All amounts in euros.*
