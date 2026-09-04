# F&B Cost Control Dashboard

Power BI F&B cost control dashboard built on modelled hotel operational data. Menu engineering, occupancy analysis, cover profitability, and DAX correlation measures. Hospitality finance portfolio project. #powerbi #dax #hospitality #hotel #fnb #menu-engineering #data-analytics #portfolio #python #pandas

A portfolio Power BI dashboard built on four years of synthetic F&B operational data, modelled on realistic patterns from upscale hotel food and beverage operations, designed for a Finance/GM audience in the hospitality industry.

---

## The business problem

Hotel F&B departments generate significant revenue but are notoriously hard to manage. Costs shift by outlet, occupancy drives volume but not always margin, and most GMs are flying blind on which menu items are actually making money. This dashboard was built to answer four specific questions a GM actually asks:

1. How healthy is F&B overall — at a glance?
2. Where is food cost creeping, and which outlet is driving it?
3. Are we busier and better, or just busier?
4. Does a full hotel automatically mean a good F&B month?

---

## Dashboard pages

### Page 1 — Executive Summary
Three headline KPIs (Total F&B Revenue, F&B Margin %, Profit per Cover) with delta labels vs budget and prior year. One margin trend area chart across the modelled period. Secondary strip for COGS %, Labour %, Var Margin pp, and Revenue per Cover. Designed to be read in under 10 seconds.

### Page 2 — Cost Deep Dive
Outlet-level COGS % vs budget (horizontal bar with conditional colouring), monthly COGS trend, year-on-year COGS + Labour waterfall, and outlet COGS mix stacked bar. Driven by a disconnected Outlets table using `SWITCH(SELECTEDVALUE())` for dynamic outlet filtering.

### Page 3 — Cover Profitability
Dual line (Revenue/cover vs Profit/cover), scatter plot of Covers vs Profit/cover coloured by year, cost per cover stacked column by year, and Actual vs Budget revenue/cover bar+line.

### Page 4 — Occupancy Influence
Dynamic Pearson correlation measures (DAX) for Occupancy vs Revenue and Occupancy vs Margin. Two scatter plots with trend lines showing the contrast. ADR vs Revenue/cover dual-axis line. Key insight: occupancy reliably drives revenue but not margin to the same degree.

### Page 5 — Menu Engineering
Menu Engineering matrix (Star/Plowhorse/Puzzle/Dog classification), top 10 revenue items ranked bar, revenue by category and menu class, and table zone performance, built on POS-style item-level sales data.

---

## Data sources

All data in this project is synthetic, generated to reflect realistic patterns, ratios, and seasonality found in upscale hotel F&B operations. No real property, guest, staff, or transaction data is included.

| Source | Description | Rows |
|---|---|---|
| Monthly P&L (synthetic) | 48 months of F&B P&L across 5 outlet types | 48 |
| Menu Item Sales (synthetic) | Item-level sales, modelled on POS export structure | 1,050 |
| Table Sales (synthetic) | Table performance by zone | 1,105 |
| Server Performance (synthetic) | Server revenue totals | 333 |

---

## Data model

```
Calendar (daily) ──── Monthly P&L (monthly, via Date_Key)
Menu Engineering ──── (standalone — no date relationship)
Category Summary ──── (standalone)
Zone Summary ──────── (standalone)
Table Performance ─── (standalone)
Outlets ───────────── (disconnected table for SWITCH pattern)
```

---

## Key DAX measures

```
-- Dynamic Pearson correlation (no native CORR in DAX)
M_Corr Occ Revenue =
VAR MeanOcc = AVERAGE('Monthly P&L'[Occupancy %])
VAR MeanRev = AVERAGE('Monthly P&L'[Total FB Revenue])
VAR Numerator = SUMX('Monthly P&L', ([Occupancy %] - MeanOcc) * ([Total FB Revenue] - MeanRev))
VAR DenomOcc = SQRT(SUMX('Monthly P&L', ([Occupancy %] - MeanOcc) ^ 2))
VAR DenomRev = SQRT(SUMX('Monthly P&L', ([Total FB Revenue] - MeanRev) ^ 2))
RETURN DIVIDE(Numerator, DenomOcc * DenomRev, 0)

-- YoY that bypasses slicer filter context
M_Covers YoY Text =
VAR CurrentYear = MAX('Monthly P&L'[Year])
VAR CurrentCovers = SUM('Monthly P&L'[Covers Est])
VAR PriorCovers = CALCULATE(
    SUM('Monthly P&L'[Covers Est]),
    ALL('Monthly P&L'),
    'Monthly P&L'[Year] = CurrentYear - 1
)
VAR CoversVar = DIVIDE(CurrentCovers - PriorCovers, PriorCovers, 0)
RETURN
IF(ISBLANK(PriorCovers), "Select a year",
    IF(CoversVar >= 0,
        "+" & FORMAT(CoversVar * 100, "0.0") & "% YoY",
        "-" & FORMAT(ABS(CoversVar * 100), "0.0") & "% YoY"))

-- Outlet COGS % via disconnected table
M_Outlet COGS Pct =
SWITCH(
    SELECTEDVALUE(Outlets[Outlet]),
    "Restaurant", [M_Restaurant COGS Pct],
    "Bar", [M_Bar COGS Pct],
    "Breakfast", [M_Breakfast COGS Pct],
    "Functions", [M_Functions COGS Pct],
    "Room Service", [M_RoomSvc COGS Pct],
    BLANK()
)
```

---

## Illustrative findings

These are patterns observable in the modelled dataset, included to demonstrate the type of insight the dashboard is designed to surface, not results from any specific property.

- Margin and COGS% show meaningful year-on-year swings driven by cost pressure and menu mix
- Occupancy vs Revenue shows a strong positive correlation — a full hotel reliably drives F&B revenue
- Occupancy vs Margin shows a weaker correlation — a full hotel does not guarantee a good margin
- Room service-style outlets typically carry the highest COGS % relative to their revenue contribution
- Banqueting/functions revenue can exceed restaurant floor revenue in a full-service upscale property

---

## Tools used

- **Power BI Desktop** — dashboard build and publishing
- **Python** (pandas, openpyxl, numpy) — data generation, modelling, and menu engineering classification
- **Microsoft Excel** — intermediate data format

---

## Files in this repo

| File | Description |
|---|---|
| `FB_Control_Case_Study_ENRICHED.xlsx` | Enriched monthly P&L dataset with budget, variance, and per-cover columns |
| `Menu_Engineering_CLEAN.csv` | Cleaned menu engineering dataset |
| `Category_Summary_CLEAN.csv` | Category-level summary for Power BI |
| `Portfolio_project_V4.pdf` | PDF export of the dashboard |
| `Portfolio_project.pbix` | Power BI source file |

---

## About

Built as a portfolio piece demonstrating hospitality finance analytics, Power BI data modelling, and DAX measure development, using a fully synthetic dataset modelled on realistic hotel F&B performance patterns.

*Alain Yon*
