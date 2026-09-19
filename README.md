# CloudFlow FP&A Dashboard

CloudFlow FP&A Dashboard is a Power BI Project (`.pbip`) for monthly financial planning and analysis. It combines general-ledger actuals with budget and forecast data to provide a management P&L and an account-level variance bridge.

## Dashboard pages

### P&L Overview

The P&L Overview is the main management reporting page. It includes:

- A reporting month selector.
- Monthly and year-to-date reporting views.
- YoY, MoM, Budget, and Forecast comparisons.
- KPI cards for Revenue, Gross Margin, Operating Profit, and Net Profit.
- A detailed income statement with Actual, Comparison, Variance, and Variance % columns.
- Favorability formatting with arrows and colors. Green indicates a favorable result and red indicates an unfavorable result. Higher costs and expenses are treated as unfavorable, while higher revenue and profit are favorable.
- Information icons that open definitions and formulas for each P&L line.

The income statement contains these lines:

| P&L line | Definition |
| --- | --- |
| Revenue | Income from core operations before costs and expenses. |
| Cost of Goods Sold | Direct costs required to deliver the products and services sold. |
| Gross Profit | Revenue − Cost of Goods Sold. |
| Gross Margin (%) | Gross Profit ÷ Revenue. Variances are shown in percentage points. |
| Operating Expenses | Indirect costs of running the business. |
| Operating Profit | Gross Profit − Operating Expenses. |
| Other Income | Income outside core operations. |
| Other Expenses | Expenses outside core operations. |
| Net Profit | Operating Profit + Other Income − Other Expenses. |
| Depreciation add-back | Depreciation added back as a non-cash charge. |
| Less: Interest Income | Interest income removed because EBITDA excludes financing-related income. |
| EBITDA | Net Profit + Depreciation add-back − Interest Income. Operating taxes and license fees remain in operating expenses. |

### Variance Bridge

The Variance Bridge explains changes in Revenue or COGS by account. Users can select:

- **Reporting Month** — the month being analyzed.
- **Analyze** — Revenue or COGS.
- **Compare** — YoY, MoM, vs Budget, or vs Forecast.

The waterfall begins with the selected comparison value, shows each account's contribution, and ends with the selected month's actual value. The table below the chart provides the comparison value, actual value, absolute variance, and variance percentage for each account.

The chart uses a focused Y-axis range so relatively small movements remain visible beside large endpoint totals. For COGS, decreases are favorable and increases are unfavorable.

### P&L Line Help

This hidden tooltip page supplies the definitions displayed from the information icons on the P&L table. It is not intended for direct navigation.

## Comparison behavior

| Comparison | Benchmark |
| --- | --- |
| YoY | Same month or YTD period in the prior year. |
| MoM | Previous month. MoM is available for the monthly view. |
| vs Budget | Budget for the selected month or YTD period. |
| vs Forecast | Latest available forecast version for the selected month or YTD period. |

The report warns users when the selected period lacks sufficient history. It also flags forecast periods whose latest forecast version is marked as `Actualized`, because Actual vs Forecast is expected to be zero for those periods.

## Semantic model

The model uses imported data from the `cloudflow_fpa` Azure SQL database on `cloudflow-fpa-sql-kr.database.windows.net`.

Core tables include:

- `silver fact_gl` — general-ledger transactions and the main P&L and variance measures.
- `silver dim_account` — account names, types, and subtypes used to construct the P&L and bridge.
- `silver dim_customer` — customer dimension.
- `gold pnl_monthly_actual` — monthly actual P&L data.
- `gold fpa_monthly` — monthly Budget and Forecast values.
- `gold fpa_version` — forecast version metadata, including forecast-as-of dates.
- `gold fpa_month` — planning calendar attributes.

Disconnected helper tables drive report selections and display order, including `Reporting Month`, `Reporting View`, `Bridge Comparison`, `Bridge Metric`, `Bridge Month`, and `P&L Lines`.

## Project structure

```text
CloudFlow FP&A Dashboard.pbip
CloudFlow FP&A Dashboard.Report/
  definition/                 # PBIR pages, visuals, bookmarks, and report settings
CloudFlow FP&A Dashboard.SemanticModel/
  definition/                 # TMDL tables, measures, relationships, and expressions
```

The source-controlled PBIP format keeps report and semantic-model definitions as text files. Visual definitions are stored as PBIR JSON, while model objects and DAX measures are stored as TMDL.

## Opening and refreshing

1. Open `CloudFlow FP&A Dashboard.pbip` in Power BI Desktop.
2. Sign in or provide credentials for the Azure SQL source when prompted.
3. Refresh the semantic model to load current Actual, Budget, and Forecast data.
4. Save the project after Power BI applies any external definition changes.

Access to the Azure SQL server and database is required for refresh. The report can still open with its locally cached data when the source is temporarily unavailable.

## Validation notes

- Select a single reporting month for consistent KPI, P&L, and bridge results.
- Confirm that the account dimension classifies accounts into the expected `account_type` and `account_subtype` values; these mappings drive every P&L line.
- Forecast comparisons use the latest available `forecast_as_of` version.
- EBITDA reflects the account mappings currently available in the model. The implementation adds back depreciation and removes interest income; operating taxes and license fees remain included in operating expenses.
