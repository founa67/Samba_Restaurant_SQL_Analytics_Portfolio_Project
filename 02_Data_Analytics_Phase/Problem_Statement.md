# Samba Restaurant Chain – Sales and Product Performance Analytics

## Executive Summary
This portfolio project presents a formal business case and analytical approach for the **Samba Restaurant Chain**. Using transactional and dimensional data from the `PRODUCTION` schema (2009–2022, ~500k sales records), this analysis aims to deliver actionable insights to drive revenue growth, optimize product assortments, benchmark branch performance across Kenyan cities, and understand seasonality and public-holiday impacts on sales.

## Business Context & Motivation
Samba operates multiple restaurant branches across five Kenyan cities. The data were prepared during the Samba Data Engineering Portfolio project and are now in the analytics phase. Executives require a set of reliable SQL analyses and reproducible queries (Snowflake) to inform pricing, promotions, inventory, staffing, and strategic expansion decisions.

## Data Sources & Scope
All data are stored in the `PRODUCTION` schema and include:
- `PRODUCTION.FACT_SALES` — granular sales transactions (timestamped, quantities, prices, revenue, product and branch references)
- `PRODUCTION.DIM_DATE` — canonical calendar / date dimension
- `PRODUCTION.DIM_PRODUCT` — product/menu item attributes and categories
- `PRODUCTION.DIM_CITY` — city-level attributes for branch grouping and regional analysis
- `PRODUCTION.DIM_BRANCH` — branch-level metadata (location, opening date, type)
- `PRODUCTION.DIM_STAFF` — staff metadata for productivity/shift analysis
- `PRODUCTION.DIM_KENYA_PUBLIC_HOLIDAYS` — national holidays to analyze seasonality/holiday effects

Timeframe: **January 1, 2009 — December 31, 2022** (inclusive). Geography: **5 Kenyan cities** (branch-level analysis aggregated by city).

## Key Analytical Objectives
1. **Revenue Growth & Time Trends** — quantify yearly and monthly trends, YoY growth, and identify inflection points.  
2. **Product & Category Performance** — identify top-selling and highest-margin products/categories and assess sustainability of product mix.  
3. **Branch Benchmarking** — compare branches across cities on revenue, average ticket, and growth; identify underperforming locations.  
4. **Seasonality & Holiday Impact** — measure sales lift or drop around public holidays and seasonal periods.  
5. **Staff & Operational Productivity** — link staff activity to sales where possible to surface staffing efficiency and training needs.

## Expected Business Outcomes
- Clear list of top-performing products and categories to inform menu engineering and promotional focus.  
- Branch-level scorecard highlighting high-opportunity stores and underperformers for operational improvement or potential closure.  
- Evidence-based recommendations for holiday promotions and staffing adjustments.  
- A curated set of Snowflake SQL queries ready for inclusion in your analytics portfolio or as parametrized reports for BI tools.

## Stakeholders
- Executive leadership (Strategy & Growth) — revenue and expansion decisions.  
- Operations & Store Managers — branch performance and staffing recommendations.  
- Merchandising / Menu Team — product and category optimization.  
- Data & Analytics Team — operationalize queries in dashboards and scheduled reports.