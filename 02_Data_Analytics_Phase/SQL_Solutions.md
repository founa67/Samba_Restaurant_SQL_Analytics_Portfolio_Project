# Samba Restaurant SQL Solutions (Snowflake)

**Note:** All queries reference the `PRODUCTION` schema and are written for Snowflake SQL. Replace schema-qualified names as needed when running in your environment.

---
## Problem Statement (summary)
The queries below support the business objectives defined in the problem statement: revenue trends, product/category performance, branch benchmarking, seasonality effects including public holidays, and staff productivity. Each query includes a title, explanation, commented Snowflake SQL, and an expected output interpretation.

---
# Ad Hoc - Simple Queries 
These are foundational lookups and aggregations to get quick insights from the dataset.

### 1. Total Revenue and Transactions by Year
**Question:** What is total revenue and total number of transactions per year (2009–2022)?

```sql
-- Total revenue and transactions per year
SELECT
  YEAR(d.date) AS year,
  COUNT(DISTINCT f.sale_id) AS transactions_count,
  SUM(f.revenue) AS total_revenue_euros
FROM SAMBA_DB.PRODUCTION.FACT_SALES f
JOIN SAMBA_DB.PRODUCTION.DIM_DATE d
  ON YEAR(f.sale_ts) = YEAR(d.date) 
  AND MONTH(f.sale_ts) = MONTH(d.date)
WHERE d.date BETWEEN '2009-01-01' AND '2022-12-31'
GROUP BY YEAR(d.date)
ORDER BY YEAR(d.date);
```

**Expected output interpretation:** A two-column time series listing each year, number of distinct transactions, and total revenue. Useful for high-level trend visualization and YoY checks.

---

### 2. Top 10 Products by Revenue (All Years)
**Question:** Which menu items generated the most revenue across the entire period?

```sql
-- Top 10 products by cumulative revenue
SELECT
  p.PRODUCT_KEY,
  p.product_name,
  p.category,
  SUM(f.revenue) AS revenue_euros,
  SUM(f.volume_sold) AS total_units_sold
FROM SAMBA_DB.PRODUCTION.FACT_SALES f
JOIN SAMBA_DB.PRODUCTION.DIM_PRODUCT p ON f.PRODUCT_KEY = p.PRODUCT_KEY
GROUP BY p.PRODUCT_KEY, p.product_name, p.category
ORDER BY revenue_euros DESC
LIMIT 10;
```

**Expected output interpretation:** Top 10 products contributing most to revenue and units sold — used for menu prioritization and promotional focus.

---

### 3. Average Ticket Value by City (Most Recent Year)
**Question:** What is the average transaction value by city for the most recent year in the dataset?

```sql
-- Average ticket (average total per transaction) by city for the latest year
WITH latest_year AS (
  SELECT MAX(YEAR(date)) AS year FROM SAMBA_DB.PRODUCTION.DIM_DATE
)
SELECT
  c.city_name,
  CAST(AVG(f.revenue) AS DECIMAL(10,2)) AS avg_ticket_euros,
  COUNT(DISTINCT f.sale_id) AS transactions_count
FROM SAMBA_DB.PRODUCTION.FACT_SALES f
JOIN SAMBA_DB.PRODUCTION.DIM_DATE d
  ON YEAR(f.sale_ts) = YEAR(d.date) 
  AND MONTH(f.sale_ts) = MONTH(d.date)
JOIN SAMBA_DB.PRODUCTION.DIM_BRANCH b ON f.branch_key = b.branch_key
JOIN SAMBA_DB.PRODUCTION.DIM_CITY c ON b.city_name = c.city_name
JOIN latest_year ly ON YEAR(d.date) = ly.year
GROUP BY c.city_name
ORDER BY avg_ticket_euros DESC;
```

**Expected output interpretation:** Average spend per transaction in each city for the latest year — helps assess customer value across regions.

---

### 4. Daily Sales for a Given Branch (Branch - 18)
**Question:** What were daily total sales for branch X in January 2022?

```sql
-- Replace :branch_id with the branch ID you wish to inspect
SELECT
  DATE(f.sale_ts) AS sale_date,
  SUM(f.revenue) AS daily_revenue_euros,
  CAST(SUM(f.volume_sold) as decimal(10,0)) AS daily_units_sold
FROM SAMBA_DB.PRODUCTION.FACT_SALES f
WHERE f.branch_key = 18
  AND DATE(f.sale_ts) BETWEEN '2022-01-01' AND '2022-01-31'
GROUP BY DATE(f.sale_ts)
ORDER BY sale_date;
```

**Expected output interpretation:** Time series of daily revenue and units sold for a specific branch and month — useful for operational analysis and anomaly detection.

---

### 5. Product Mix Share by Category (Latest Year). Portfolio Analysis
**Question:** What share of revenue does each product category contribute in the latest year?

```sql
WITH latest_year AS (
  SELECT MAX(YEAR(date)) AS year FROM SAMBA_DB.PRODUCTION.DIM_DATE
),
cat_rev AS (
  SELECT p.category,
         SUM(f.revenue) AS revenue_euros
  FROM SAMBA_DB.PRODUCTION.FACT_SALES f
  JOIN SAMBA_DB.PRODUCTION.DIM_PRODUCT p ON f.PRODUCT_KEY = p.PRODUCT_KEY
  JOIN SAMBA_DB.PRODUCTION.DIM_DATE d
  ON YEAR(f.sale_ts) = YEAR(d.date) 
  AND MONTH(f.sale_ts) = MONTH(d.date)
  JOIN latest_year ly ON YEAR(d.date) = ly.year
  GROUP BY p.category
)
SELECT
  category,
  revenue_euros,
  ROUND(revenue_euros / SUM(revenue_euros) OVER () * 100, 2) AS pct_of_total_revenue
FROM cat_rev
ORDER BY revenue_euros DESC;
```

**Expected output interpretation:** Percent contribution of each category to yearly revenue — useful for strategic category investments.

---
# Ad Hoc -Intermediate Queries
These queries use joins, grouping, and basic window functions to compare branches and categories.

### 6. Branch Revenue Ranking by Year
**Question:** Rank branches by revenue for each year to see top performers and movement over time.

```sql
-- Branch ranking per year by revenue
WITH branch_year_rev AS (
  SELECT
    b.branch_key,
    b.branch_name,
    c.city_name,
    YEAR(d.date) as calendar_year,
    SUM(f.revenue) AS revenue_euros
  FROM SAMBA_DB.PRODUCTION.FACT_SALES f
  JOIN SAMBA_DB.PRODUCTION.DIM_BRANCH b ON f.branch_key = b.branch_key
  JOIN SAMBA_DB.PRODUCTION.DIM_CITY c ON b.city_name = c.city_name
  JOIN SAMBA_DB.PRODUCTION.DIM_DATE d ON YEAR(f.sale_ts) = YEAR(d.date)   AND MONTH(f.sale_ts) = MONTH(d.date)
  WHERE d.date BETWEEN '2009-01-01' AND '2022-12-31'
  GROUP BY b.branch_key, b.branch_name, c.city_name, YEAR(d.date)
)
SELECT
  calendar_year,
  branch_key,
  branch_name,
  city_name,
  revenue_euros,
  RANK() OVER (PARTITION BY calendar_year ORDER BY revenue_euros DESC) AS year_rank
FROM branch_year_rev
ORDER BY calendar_year DESC, year_rank;
```

**Expected output interpretation:** Yearly rank of each branch — highlights branches that consistently lead or improve/decline year to year.

---

### 7. Compare Branch Performance to City Average (Z-score style)
**Question:** Which branches deviate most from their city peers in terms of yearly revenue?

```sql
WITH branch_year_rev AS (
  SELECT
    b.branch_key,
    b.branch_name,
    c.city_id,
    c.city_name,
    YEAR(d.date) as calendar_year,
    SUM(f.revenue) AS revenue_euros
  FROM SAMBA_DB.PRODUCTION.FACT_SALES f
  JOIN SAMBA_DB.PRODUCTION.DIM_BRANCH b ON f.branch_key = b.branch_key
  JOIN SAMBA_DB.PRODUCTION.DIM_CITY c ON b.city_name = c.city_name
  JOIN SAMBA_DB.PRODUCTION.DIM_DATE d ON YEAR(f.sale_ts) = YEAR(d.date)   AND MONTH(f.sale_ts) = MONTH(d.date)
  GROUP BY b.branch_key, b.branch_name, c.city_id, c.city_name, YEAR(d.date)
),
city_stats AS (
  SELECT city_id, calendar_year,
         CAST(AVG(revenue_euros) as decimal(10,2)) AS city_avg_rev,
         CAST(STDDEV_POP(revenue_euros) as decimal(10,2)) AS city_rev_stddev
         
  FROM branch_year_rev
  GROUP BY city_id, calendar_year
)
SELECT
  r.calendar_year,
  r.city_name,
  r.branch_name,
  r.revenue_euros,
  s.city_avg_rev,
  s.city_rev_stddev,
  CASE WHEN s.city_rev_stddev = 0 THEN NULL
       ELSE ROUND((r.revenue_euros - s.city_avg_rev) / s.city_rev_stddev, 2)
  END AS rev_z
FROM branch_year_rev r
JOIN city_stats s
  ON r.city_id = s.city_id AND r.calendar_year = s.calendar_year
ORDER BY rev_z DESC NULLS LAST
LIMIT 50;
```

**Expected output interpretation:** Branches with highest positive or negative z-scores versus their city peers — useful to prioritize investigation and replicate best practices.

---

### 8. Top Categories per City (Latest Year)
**Question:** What are the top 3 categories by revenue in each city for the latest year?

```sql
WITH latest_year AS (
  SELECT MAX(YEAR(date)) AS year FROM SAMBA_DB.PRODUCTION.DIM_DATE
),
city_cat_rev AS (
  SELECT c.city_name, p.category, SUM(f.revenue) AS revenue_euros
  FROM SAMBA_DB.PRODUCTION.FACT_SALES f
  JOIN SAMBA_DB.PRODUCTION.DIM_PRODUCT p ON f.product_key = p.product_key
  JOIN SAMBA_DB.PRODUCTION.DIM_BRANCH b ON f.branch_key = b.branch_key
  JOIN SAMBA_DB.PRODUCTION.DIM_CITY c ON b.city_name = c.city_name
  JOIN SAMBA_DB.PRODUCTION.DIM_DATE d ON YEAR(f.sale_ts) = YEAR(d.date)   AND MONTH(f.sale_ts) = MONTH(d.date)
  JOIN latest_year ly ON YEAR(d.date) = ly.year
  GROUP BY c.city_name, p.category
)
SELECT city_name, category, revenue_euros
FROM (
  SELECT
    city_name, category, revenue_euros,
    ROW_NUMBER() OVER (PARTITION BY city_name ORDER BY revenue_euros DESC) AS rn
  FROM city_cat_rev
)
WHERE rn <= 3
ORDER BY city_name, revenue_euros DESC;
```

**Expected output interpretation:** Top 3 revenue-driving categories in each city — guides localized marketing and assortment decisions.

---

### 9. Holiday Sales Lift (Holiday vs Non-Holiday Same Week)
**Question:** On average, how much uplift do public holidays create in daily sales compared to other same-week days?

```sql
-- Compare average daily sales on holidays vs non-holidays within the same ISO week
WITH sales_with_meta AS (
  SELECT f.sale_date, d.iso_week, d.calendar_year, SUM(f.total_amount_euros) AS daily_revenue
  FROM PRODUCTION.FACT_SALES f
  JOIN PRODUCTION.DIM_DATE d ON f.sale_date = d.date
  GROUP BY f.sale_date, d.iso_week, d.calendar_year
),
holiday_flags AS (
  SELECT date AS hol_date, TRUE AS is_holiday FROM PRODUCTION.DIM_KENYA_PUBLIC_HOLIDAYS
)
SELECT
  s.calendar_year,
  AVG(CASE WHEN h.hol_date IS NOT NULL THEN s.daily_revenue END) AS avg_revenue_holiday,
  AVG(CASE WHEN h.hol_date IS NULL THEN s.daily_revenue END) AS avg_revenue_non_holiday,
  ROUND(
    (AVG(CASE WHEN h.hol_date IS NOT NULL THEN s.daily_revenue END) - AVG(CASE WHEN h.hol_date IS NULL THEN s.daily_revenue END))
    / NULLIF(AVG(CASE WHEN h.hol_date IS NULL THEN s.daily_revenue END),0) * 100, 2
  ) AS pct_lift_vs_non_holiday
FROM sales_with_meta s
LEFT JOIN holiday_flags h ON s.sale_date = h.hol_date
GROUP BY s.calendar_year
ORDER BY s.calendar_year;
```

**Expected output interpretation:** Yearly percentage lift (or decline) in daily revenue when the day is a public holiday. Useful for promotional planning and staffing during holidays.

---
# Advanced Queries (5)
Advanced analytics using window functions, advanced time intelligence, cohort-like comparisons, and product-branch interactions.

### 10. Rolling 12-Month Revenue Growth per Branch (Month-end)
**Question:** What is the 12-month rolling revenue and YoY change per branch (monthly grain)?

```sql
-- 12-month rolling revenue and YoY pct change by branch at month level
WITH monthly_branch_rev AS (
  SELECT
    b.branch_id,
    b.branch_name,
    DATE_TRUNC('month', d.date) AS month_start,
    SUM(f.total_amount_euros) AS revenue_month
  FROM PRODUCTION.FACT_SALES f
  JOIN PRODUCTION.DIM_BRANCH b ON f.branch_id = b.branch_id
  JOIN PRODUCTION.DIM_DATE d ON f.sale_date = d.date
  GROUP BY b.branch_id, b.branch_name, DATE_TRUNC('month', d.date)
),
rolling AS (
  SELECT
    branch_id,
    branch_name,
    month_start,
    SUM(revenue_month) OVER (PARTITION BY branch_id ORDER BY month_start ROWS BETWEEN 11 PRECEDING AND CURRENT ROW) AS rolling_12m_rev,
    LAG(SUM(revenue_month) OVER (PARTITION BY branch_id ORDER BY month_start ROWS BETWEEN 11 PRECEDING AND CURRENT ROW), 12) OVER (PARTITION BY branch_id ORDER BY month_start) AS prev_12m_rev
  FROM monthly_branch_rev
)
SELECT
  branch_id,
  branch_name,
  month_start,
  rolling_12m_rev,
  prev_12m_rev,
  CASE WHEN prev_12m_rev IS NULL OR prev_12m_rev = 0 THEN NULL
       ELSE ROUND((rolling_12m_rev - prev_12m_rev) / prev_12m_rev * 100, 2)
  END AS pct_change_vs_prev_12m
FROM rolling
ORDER BY branch_id, month_start DESC;
```

**Expected output interpretation:** For each branch and month, show a trailing 12-month revenue and percent change versus the prior 12 months — useful for detecting sustained growth or decline.

---

### 11. Product Affinity: Products Frequently Bought Together (Top pairs)- (Market Basket Analysis)
**Question:** Which product pairs are most frequently sold in the same transaction (basket affinity)?

```sql
-- Frequent product pairs in same transaction (approximate, top pairs)
WITH tx_products AS (
  SELECT transaction_id, product_id
  FROM PRODUCTION.FACT_SALES
  WHERE product_id IS NOT NULL
  GROUP BY transaction_id, product_id
),
pairs AS (
  SELECT
    a.product_id AS product_a,
    b.product_id AS product_b,
    COUNT(*) AS cooccurrence_count
  FROM tx_products a
  JOIN tx_products b
    ON a.transaction_id = b.transaction_id
   AND a.product_id < b.product_id -- avoid duplicate reversed pairs and self-pairing
  GROUP BY a.product_id, b.product_id
)
SELECT
  pa.product_name AS product_a_name,
  pb.product_name AS product_b_name,
  p.cooccurrence_count
FROM pairs p
JOIN PRODUCTION.DIM_PRODUCT pa ON p.product_a = pa.product_id
JOIN PRODUCTION.DIM_PRODUCT pb ON p.product_b = pb.product_id
ORDER BY p.cooccurrence_count DESC
LIMIT 50;
```

**Expected output interpretation:** Top 50 product pairs most commonly purchased together; useful for cross-sell bundles and menu placement.

---

### 12. Category Margin Heatmap — Category x City
**Question:** Which city-category combinations produce the highest average margin (and where are margins weakest)?

```sql
-- Average margin percentage by category and city
SELECT
  c.city_name,
  p.category,
  ROUND(AVG(NULLIF((f.total_amount_euros - f.cost_amount_euros) / NULLIF(f.total_amount_euros,0),0) * 100), 2) AS avg_margin_pct,
  SUM(f.total_amount_euros) AS revenue_euros,
  COUNT(DISTINCT f.transaction_id) AS transaction_count
FROM PRODUCTION.FACT_SALES f
JOIN PRODUCTION.DIM_PRODUCT p ON f.product_id = p.product_id
JOIN PRODUCTION.DIM_BRANCH b ON f.branch_id = b.branch_id
JOIN PRODUCTION.DIM_CITY c ON b.city_id = c.city_id
GROUP BY c.city_name, p.category
ORDER BY avg_margin_pct DESC NULLS LAST;
```

**Expected output interpretation:** A list ranking city-category pairs by average margin percentage — helps focus assortment and pricing by geography.

---

### 13. Staff Performance: Revenue per Staff-hour 
**Question:** Which staff members deliver highest revenue per recorded hours (productivity measure)?

```sql
-- Assumes FACT_SALES has staff_id and possibly shift_hours in DIM_STAFF or similar.
-- If shift_hours are not available, this query uses staff scheduled hours in DIM_STAFF.shift_hours_monthly (example)
WITH staff_sales AS (
  SELECT s.staff_id, s.staff_name, SUM(f.total_amount_euros) AS revenue_euros
  FROM PRODUCTION.FACT_SALES f
  JOIN PRODUCTION.DIM_STAFF s ON f.staff_id = s.staff_id
  GROUP BY s.staff_id, s.staff_name
),
staff_hours AS (
  -- If you have a monthly recorded hours field; adjust as needed.
  SELECT staff_id, SUM(shift_hours) AS total_hours
  FROM PRODUCTION.DIM_STAFF -- replace with a real timesheet table if available
  GROUP BY staff_id
)
SELECT
  ss.staff_id,
  ss.staff_name,
  ss.revenue_euros,
  sh.total_hours,
  CASE WHEN sh.total_hours IS NULL OR sh.total_hours = 0 THEN NULL
       ELSE ROUND(ss.revenue_euros / sh.total_hours, 2)
  END AS revenue_per_hour_euros
FROM staff_sales ss
LEFT JOIN staff_hours sh ON ss.staff_id = sh.staff_id
ORDER BY revenue_per_hour_euros DESC NULLS LAST
LIMIT 50;
```

**Expected output interpretation:** Ranking of staff by revenue generated per hour — best interpreted with caution (influence of shift assignments and role differences).

---

### 14. Year-over-Year Sales Growth by Category with Statistical Significance-like Check
**Question:** Which categories show consistent YoY growth and which are declining?

```sql
-- YoY growth per category and simple persistence check
WITH cat_year_rev AS (
  SELECT p.category, d.calendar_year, SUM(f.total_amount_euros) AS revenue_euros
  FROM PRODUCTION.FACT_SALES f
  JOIN PRODUCTION.DIM_PRODUCT p ON f.product_id = p.product_id
  JOIN PRODUCTION.DIM_DATE d ON f.sale_date = d.date
  GROUP BY p.category, d.calendar_year
)
SELECT
  cy.category,
  cy.year,
  cy.revenue_euros,
  LAG(cy.revenue_euros) OVER (PARTITION BY cy.category ORDER BY cy.year) AS prev_year_rev,
  CASE WHEN LAG(cy.revenue_euros) OVER (PARTITION BY cy.category ORDER BY cy.year) IS NULL THEN NULL
       ELSE ROUND((cy.revenue_euros - LAG(cy.revenue_euros) OVER (PARTITION BY cy.category ORDER BY cy.year)) 
                  / LAG(cy.revenue_euros) OVER (PARTITION BY cy.category ORDER BY cy.year) * 100, 2)
  END AS pct_yoy
FROM cat_year_rev cy
ORDER BY category, year;
```

**Expected output interpretation:** YoY percentage change for each category by year — identifies categories with sustained growth or decline. Use with charts to spot consistent patterns.

---

### 15. Branch Comparison Report: Multi-metric Scorecard & Cluster-like Buckets
**Question:** Create a composite score for branches combining revenue growth, avg ticket, and margin to classify branches into Performance Buckets (Top, Mid, Low).

```sql
-- Create a composite z-score-like metric and bucket branches into performance groups
WITH branch_metrics AS (
  SELECT
    b.branch_id,
    b.branch_name,
    c.city_name,
    SUM(f.total_amount_euros) AS total_revenue,
    AVG(f.total_amount_euros) AS avg_ticket,
    AVG(NULLIF((f.total_amount_euros - f.cost_amount_euros) / NULLIF(f.total_amount_euros,0),0) * 100) AS avg_margin_pct
  FROM PRODUCTION.FACT_SALES f
  JOIN PRODUCTION.DIM_BRANCH b ON f.branch_id = b.branch_id
  JOIN PRODUCTION.DIM_CITY c ON b.city_id = c.city_id
  GROUP BY b.branch_id, b.branch_name, c.city_name
),
metrics_stats AS (
  SELECT
    AVG(total_revenue) AS avg_rev, STDDEV_POP(total_revenue) AS std_rev,
    AVG(avg_ticket) AS avg_ticket_all, STDDEV_POP(avg_ticket) AS std_ticket,
    AVG(avg_margin_pct) AS avg_margin_all, STDDEV_POP(avg_margin_pct) AS std_margin
  FROM branch_metrics
)
SELECT
  bm.branch_id,
  bm.branch_name,
  bm.city_name,
  bm.total_revenue,
  bm.avg_ticket,
  bm.avg_margin_pct,
  -- z-scores
  ROUND((bm.total_revenue - ms.avg_rev) / NULLIF(ms.std_rev,0), 2) AS z_rev,
  ROUND((bm.avg_ticket - ms.avg_ticket_all) / NULLIF(ms.std_ticket,0), 2) AS z_ticket,
  ROUND((bm.avg_margin_pct - ms.avg_margin_all) / NULLIF(ms.std_margin,0), 2) AS z_margin,
  -- composite score (simple weighted sum, weights adjustable)
  ROUND(
    COALESCE((bm.total_revenue - ms.avg_rev) / NULLIF(ms.std_rev,0),0) * 0.5
    + COALESCE((bm.avg_ticket - ms.avg_ticket_all) / NULLIF(ms.std_ticket,0),0) * 0.3
    + COALESCE((bm.avg_margin_pct - ms.avg_margin_all) / NULLIF(ms.std_margin,0),0) * 0.2
  ,2) AS composite_score,
  CASE
    WHEN composite_score >= 1.0 THEN 'Top'
    WHEN composite_score BETWEEN 0 AND 1.0 THEN 'Mid'
    ELSE 'Low'
  END AS performance_bucket
FROM branch_metrics bm
CROSS JOIN metrics_stats ms
ORDER BY composite_score DESC;
```

**Expected output interpretation:** A ranked list of branches with composite score and buckets. Use to target best practices, replications, or interventions for low-performing stores.

---
# Final notes & recommendations
1. Parameterize queries (e.g., `:branch_id`, `:start_date`, `:end_date`) before embedding into dashboards or scheduled jobs.  
2. Materialize intermediate heavy queries into Snowflake transient tables or results caching if used frequently in dashboards.  
3. Consider adding a denormalized reporting table (monthly_aggregates) to accelerate BI dashboards for executive stakeholders.  
4. Validate costs (e.g., `cost_amount_euros`) and missing data handling before computing margins.  
5. Document assumptions (time zone, currency, refunds handling) alongside the queries in your portfolio README.
