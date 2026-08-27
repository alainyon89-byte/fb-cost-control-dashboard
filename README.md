# fb-cost-control-dashboard
Power BI F&amp;B cost control dashboard built on 4 years of hotel data. Menu engineering, occupancy analysis, cover profitability, and DAX correlation measures. Hospitality finance portfolio project. #powerbi #dax #hospitality #hotel #fnb #menu-engineering #data-analytics #portfolio #python  pandas
# F&B Cost Control Dashboard — Sandton Sun & Towers

A portfolio Power BI dashboard built on four years of synthetic and real F&B operational data (2022–2025), designed for a Finance/GM audience in the hospitality industry.

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
Three headline KPIs (Total F&B Revenue, F&B Margin %, Profit per Cover) with delta labels vs budget and prior year. One margin trend area chart across 2022–2025. Secondary strip for COGS %, Labour %, Var Margin pp, and Revenue per Cover. Designed to be read in under 10 seconds.

### Page 2 — Cost Deep Dive
Outlet-level COGS % vs budget (horizontal bar with conditional colouring), monthly COGS trend, year-on-year COGS + Labour waterfall, and outlet COGS mix stacked bar. Driven by a disconnected Outlets table using `SWITCH(SELECTEDVALUE())` for dynamic outlet filtering.

### Page 3 — Cover Profitability
Dual line (Revenue/cover vs Profit/cover), scatter plot of Covers vs Profit/cover coloured by year (the hero visual — 2023 bottom-left, 2025 top-right), cost per cover stacked column by year, and Actual vs Budget revenue/cover bar+line.

### Page 4 — Occupancy Influence
Dynamic Pearson correlation measures (DAX) for Occupancy vs Revenue (r=0.865) and Occupancy vs Margin (r=0.568). Two scatter plots with trend lines showing the contrast. ADR vs Revenue/cover dual-axis line. Key insight: occupancy reliably drives revenue but not margin.

### Page 5 — Menu Engineering
Built from real Simphony POS data (anonymised, scaled). Menu Engineering matrix (Star/Plowhorse/Puzzle/Dog classification), top 10 revenue items ranked bar, revenue by category and menu class, and table zone performance. Oxtail leads at R7.25M across four years.

---

## Data sources

| Source | Description | Rows |
|--------|-------------|------|
| Monthly P&L (synthetic) | 48 months of F&B P&L across 5 outlets | 48 |
| Menu Item Sales (Simphony POS) | Item-level sales 2022–2025, anonymised | 1,050 |
| Table Sales (Simphony POS) | Table performance by zone, anonymised | 1,105 |
| Server Performance (Simphony POS) | Server revenue totals, fully anonymised | 333 |

**Anonymisation applied to real data:** Revenue and quantity figures scaled by factor 0.9025. Staff names replaced with Server_XX codes. All patterns, rankings, and proportions preserved.

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

```dax
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

## Key findings

- **2023 was the worst year:** Margin collapsed to 20.8%, COGS hit 34.3%, profit per cover fell to R45
- **2025 was the best year:** Margin recovered to 30.1%, profit per cover grew to R72 — the highest in four years
- **Occupancy vs Revenue correlation = 0.865** — strong. A full hotel reliably drives F&B revenue
- **Occupancy vs Margin correlation = 0.568** — moderate. A full hotel does not guarantee a good margin
- **Oxtail is the #1 revenue item** at R7.25M across four years — a genuine Star
- **Room Service runs the highest COGS %** (38.2%) but contributes only ~5% of revenue
- **Banqueting is the largest revenue zone** at R114M, ahead of the Restaurant floor at R74M

---

## Tools used

- **Power BI Desktop** — dashboard build and publishing
- **Python** (pandas, openpyxl, numpy) — data enrichment, anonymisation, menu engineering classification
- **Simphony POS** — source system for item-level and table-level sales data
- **Microsoft Excel** — intermediate data format

---

## Files in this repo

| File | Description |
|------|-------------|
| `FB_Control_Case_Study_ENRICHED.xlsx` | Enriched monthly P&L dataset with budget, variance, and per-cover columns |
| `FB_Menu_Engineering_CLEAN.xlsx` | Cleaned and anonymised menu engineering dataset |
| `Menu_Engineering_CLEAN.csv` | CSV version for clean Power BI import |
| `Category_Summary_CLEAN.csv` | Category-level summary for Power BI |
| `FB_Dashboard_Plain_Guide.docx` | Plain-language reviewer guide |
| `Portfolio_project_V4.pdf` | PDF export of the dashboard |
| `Portfolio_project.pbix` | PDF export of the dashboard |

---

## About

Built as a portfolio piece demonstrating hospitality finance analytics, Power BI data modelling, and DAX measure development. The P&L dataset is synthetic but modelled on realistic hotel F&B performance patterns. Menu and table data is from a real property, anonymised.

*Alain Yon — August 2026*
