# DAX Measures & Data Model

Data model, calculated columns, parameters and DAX measures used in [`LexEuropa_Legal_Dashboard.pbix`](../LexEuropa_Legal_Dashboard.pbix).

## Data model (star schema)

| Table | Rows | Role |
|---|---|---|
| `Fact_Cases` | 15,000 | One row per case: hours, revenue, costs, profit, invoice, outstanding balance, satisfaction, risk, utilisation |
| `Dim_Case` | 15,000 | Case name, status, outcome, complexity, type, practice area, priority |
| `Dim_Lawyer` | 200 | Name, gender, age group, experience, seniority, department |
| `Dim_Client` | 800 | Industry, company size, country, region, strategic flag |
| `Dim_Office` | 15 | City, country, region, latitude/longitude (for the map) |
| `Dim_Date` | 1,096 | Calendar 2024–2026 |

**Relationships:** `Fact_Cases` → `Dim_Lawyer`, `Dim_Client`, `Dim_Office` and `Dim_Date` (on Open Date) are many-to-one. `Fact_Cases` → `Dim_Case` is one-to-one.

## Calculated column

```dax
Is Decided Outcome =
    IF ( Dim_Case[Case Outcome] IN { "Won", "Lost", "Settled" }, 1, 0 )
```

Used so that win rate = won ÷ decided cases, excluding Ongoing and Withdrawn cases.

## Field parameters

| Parameter | Lets the user switch the chart between |
|---|---|
| `Parameter` (Executive page) | Seniority · Client Region · Priority · Case Type · Department |
| `Parameter 2` (Lawyer page) | Age Group · Gender · Seniority |

## Measures

### Core KPIs

```dax
Total_Case =
    COUNTROWS(Fact_Cases)
```

```dax
Total_Revenue =
    SUM(Fact_Cases[Revenue (EUR)])
```

```dax
Total_Profit =
    SUM(Fact_Cases[Profit (EUR)])
```

```dax
Avg_Profit_Margin =
    AVERAGE(Fact_Cases[Profit Margin %])
```

```dax
Avg_Days_Open =
    AVERAGE(Fact_Cases[Days Open])
```

```dax
Avg_Satisfaction =
    AVERAGE(Fact_Cases[Client Satisfaction Score])
```

```dax
Avg_risk_score =
    AVERAGE(Fact_Cases[Risk Score])
```

```dax
Total_Invoice =
    SUM(Fact_Cases[Invoice Amount (EUR)])
```

```dax
Total_Oustanding_Balance =
    SUM(Fact_Cases[Outstanding Balance (EUR)])
```

```dax
Total_Client =
    COUNTROWS(Dim_Client)
```

### Case outcomes & win rate

```dax
Won Cases =
    CALCULATE ( COUNTROWS ( Fact_Cases ), Dim_Case[Case Outcome] = "Won" )
```

```dax
Lost Cases =
    CALCULATE ( COUNTROWS ( Fact_Cases ), Dim_Case[Case Outcome] = "Lost" )
```

```dax
Decided Cases =
    CALCULATE ( COUNTROWS ( Fact_Cases ), Dim_Case[Is Decided Outcome] = 1 )
```

```dax
Win Rate % =
    DIVIDE ( [Won Cases], [Decided Cases] )
```

```dax
Lost_Rate =
    DIVIDE( [Lost Cases], [Decided Cases])
```

```dax
SettledCases =
    CALCULATE ( COUNTROWS ( Fact_Cases ), Dim_Case[Case Outcome] = "Settled" )
```

```dax
OngoingCase =
    CALCULATE ( COUNTROWS ( Fact_Cases ), Dim_Case[Case Outcome] = "Ongoing" )
```

```dax
WithdrawnCase =
    CALCULATE ( COUNTROWS ( Fact_Cases ), Dim_Case[Case Outcome] = "Withdrawn" )
```

### Lawyer performance score

```dax
Total_Lawyer =
    COUNT(Dim_Lawyer[Lawyer ID])
```

```dax
AVG_Lawyer_Utlization =
    AVERAGE(Fact_Cases[Lawyer Utilization %])
```

```dax
Normalized Revenue =
    DIVIDE (
        [Total_Revenue] - CALCULATE ( MINX ( ALL ( Dim_Lawyer[Lawyer ID] ), [Total_Revenue] ) ),
        CALCULATE ( MAXX ( ALL ( Dim_Lawyer[Lawyer ID] ),  [Total_Revenue] ) )
        - CALCULATE ( MINX ( ALL ( Dim_Lawyer[Lawyer ID] ),  [Total_Revenue] ) )
    )
```

```dax
Normalized Win Rate =
    DIVIDE (
        [Win__Rate %] - CALCULATE ( MINX ( ALL ( Dim_Lawyer[Lawyer ID] ), [Win__Rate %] ) ),
        CALCULATE ( MAXX ( ALL ( Dim_Lawyer[Lawyer ID] ), [Win__Rate %] ) )
        - CALCULATE ( MINX ( ALL ( Dim_Lawyer[Lawyer ID] ), [Win__Rate %] ) )
    )
```

```dax
Normalized Satisfaction =
    DIVIDE (
        [Avg_Satisfaction] - CALCULATE ( MINX ( ALL ( Dim_Lawyer[Lawyer ID] ), [Avg_Satisfaction] ) ),
        CALCULATE ( MAXX ( ALL ( Dim_Lawyer[Lawyer ID] ), [Avg_Satisfaction] ) )
        - CALCULATE ( MINX ( ALL ( Dim_Lawyer[Lawyer ID] ), [Avg_Satisfaction] ) )
    )
```

```dax
Lawyer Performance Score =
    ( [Normalized Revenue] * 0.4 )
    + ( [Normalized Win Rate] * 0.3 )
    + ( [Normalized Satisfaction] * 0.3 )
```

### Client analysis

```dax
Client Revenue Rank =
    RANKX (
        ALL ( Dim_Client[Client ID] ),
        [Total_Revenue],
        ,
        DESC
    )
```

```dax
Top 10 Client Revenue =
    CALCULATE (
        [Total_Revenue],
        TOPN ( 10, ALL ( Dim_Client[Client ID] ), [Total_Revenue], DESC )
    )
```

```dax
Firm Total Revenue =
    CALCULATE ( [Total_Revenue], ALL ( Dim_Client ) )
```

```dax
Top 10 Revenue Concentration % =
    DIVIDE ( [Top 10 Client Revenue], [Firm Total Revenue] )
```

### Case risk & recommendation

```dax
Case_Pressure_Index =
    ([Avg_risk_score]/100) * 0.4
    + DIVIDE([Avg_Days_Open],900) * 0.3
    +DIVIDE([Total_Oustanding_Balance], [Total_Invoice] ) * 0.3
```

```dax
Case Recommendation =
    VAR PressureLevel = [Case_Pressure_Index]
    VAR RevenueLevel = [Total_Revenue]
    VAR AvgRev = CALCULATE ( AVERAGEX ( ALL ( Fact_Cases[Case ID] ), [Total_Revenue] ) )
    RETURN
    SWITCH (
        TRUE (),
        PressureLevel >= 0.65 && RevenueLevel >= AvgRev, 
            "🔴 Critical Priority — escalate to partner-level review immediately",
        PressureLevel >= 0.65 && RevenueLevel < AvgRev, 
            "🟠 Hidden Risk — low revenue, high pressure. Consider settlement or resource reallocation",
        PressureLevel >= 0.45 && PressureLevel < 0.65, 
            "🟡 Monitor — add to watch list, review monthly",
        PressureLevel < 0.45 && RevenueLevel >= AvgRev, 
            "🟢 Star Performer — replicate approach/best practices",

        "⚪ Routine — no action needed"
    )
```

```dax
Days Open 3M Moving Avg =
    AVERAGEX (
        DATESINPERIOD ( Dim_Date[Date], MAX ( Dim_Date[Date] ), -3, MONTH ),
        [Avg_Days_Open]
    )
```

### Previous-year values

```dax
PY_Revenue =
    CALCULATE([Total_Revenue],SAMEPERIODLASTYEAR(Dim_Date[Date]))
```

```dax
PY_Profit =
    CALCULATE([Total_Profit],SAMEPERIODLASTYEAR(Dim_Date[Date]))
```

```dax
PY_TotalCase =
    CALCULATE([Total_Case],SAMEPERIODLASTYEAR(Dim_Date[Date]))
```

```dax
PY_WinRate =
    CALCULATE([Win Rate %], SAMEPERIODLASTYEAR(Dim_Date[Date]))
```

```dax
PY_Satisfaction =
    CALCULATE([Avg_Satisfaction], SAMEPERIODLASTYEAR(Dim_Date[Date]))
```

```dax
PY_Avg_Profit_Margin =
    CALCULATE([Avg_Profit_Margin], SAMEPERIODLASTYEAR(Dim_Date[Date]))
```

```dax
PY_Average_Day_Open =
    CALCULATE([Avg_Days_Open], SAMEPERIODLASTYEAR(Dim_Date[Date]))
```

```dax
PY_Oustanding_Balance =
    CALCULATE( [Total_Oustanding_Balance], SAMEPERIODLASTYEAR(Dim_Date[Date]))
```

```dax
YoY%_Revenue =
    DIVIDE([Total_Revenue] - [PY_Revenue], [PY_Revenue])
```

```dax
YoY%_Profit_Margin =
    DIVIDE( [Avg_Profit_Margin] -[PY_Avg_Profit_Margin],[PY_Avg_Profit_Margin])
```

```dax
YoY%_Oustanding_Balance =
    DIVIDE([Total_Oustanding_Balance] -[PY_Oustanding_Balance],[PY_Oustanding_Balance])
```

```dax
YoY%_Satisfaction =
    [Avg_Satisfaction] - [PY_Satisfaction]
```

```dax
YoY%_DaysOpen =
    [Avg_Days_Open] -[PY_Average_Day_Open]
```

### Year-on-year change with arrows (KPI cards)

All KPI cards use the same pattern: compare the current value with the same period last year, then show an up or down arrow and the % change. Example:

```dax
%Revenue_VS_Last_Year =
    VAR _CY = [Total_Revenue]
    VAR _PY = 
    CALCULATE(
        [Total_Revenue],
        DATEADD('Dim_Date'[Date], -1, YEAR)
    )
    VAR _perc = DIVIDE( _CY - _PY, _PY)
    VAR _format = 
    SWITCH(
        TRUE(),
        ISBLANK(_perc), BLANK(),
        _perc > 0, UNICHAR(11165) & " " & FORMAT( _perc, "0.0%"),
        _perc < 0, UNICHAR(11167) & " " & FORMAT( _perc*-1, "0.0%" ),
        FORMAT(_perc, "0.0%")
    )
    RETURN
    _format
```

The same pattern is used for: `%VS_Last_Year`, `Format_%VS_Last_Year`, `%Profit_VS_Last_Year`, `%AVg_Profit_Margin_VS_Last_Year`, `%winRate_VS_Last_Year`, `CF_WinRate`, `%VS_Avg_Lawyer_Utilization`, `CF_Avg_Lawyer_Utilization`, `%AVg_DayOpen`, `CF_DayOpen`, `%Profit_Margin`, `%AVg_risk_score`. Measures starting with `CF_` return a colour (green/red) for conditional formatting.
Helper and duplicate measures not shown above: `Win__Rate %` (same as `Win Rate %`, used inside `Normalized Win Rate`), `Win2Rate %`, `CasesLost`, `Last_Year_Total_Case`, `Last_Year_Avg_Lawyer_Untlization`, `Last_Year_Avg_riskmargin`, `Last Refresh`.
