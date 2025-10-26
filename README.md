# Problem 3 — Supplier Performance & Cost Variance Analysis (2 hours)

## Business Context
You're part of the procurement analytics team at a manufacturing company. Leadership wants
to understand which suppliers reliably deliver on time and within cost expectations — and
what factors drive risk (late deliveries, overruns).

You are given **three datasets** that live in different systems today:
- `purchase_orders.csv`: individual PO line-level transactional data
- `supplier_master.csv`: supplier attributes (region, tier, size, quality rating)
- `material_cost_standards.csv`: standard cost per material and fiscal year

Your job is to **join, clean, and engineer features** that summarize performance at the
supplier level.

---

## Files Provided

### 1. `purchase_orders.csv`
Each row is a purchase order line for a specific material from a specific supplier.

Columns:
- `po_id`
- `po_date`
- `supplier_id`
- `material_id`
- `scheduled_delivery_date`
- `actual_delivery_date`
- `currency` (USD or EUR)
- `po_value` (in the stated currency)
- `actual_cost` (in the stated currency)

Notes:
- Some `actual_delivery_date` values are missing (open POs).
- Some deliveries arrive early (negative delay) or very late (10+ days late).

### 2. `supplier_master.csv`
Columns:
- `supplier_id`
- `supplier_name`
- `region` (messy: e.g. "North America", "N. America", "EU", "APAC", etc.)
- `tier` (1 = strategic, 2 = preferred, 3 = approved / backup)
- `supplier_size` (Small / Medium / Large)
- `quality_rating` (0-100 score, higher is better)

Notes:
- Region values are intentionally inconsistent across rows. You should normalize them
  into consistent buckets (e.g. "North America", "Europe", "Asia Pacific", "LATAM").

### 3. `material_cost_standards.csv`
Columns:
- `material_id`
- `fiscal_year`
- `standard_cost_usd`

Notes:
- This represents the *expected* unit cost for that material in that fiscal year.

---

## Tasks

### 1. Join & Clean
- Join `purchase_orders` ↔ `supplier_master` using `supplier_id`.
- Join `purchase_orders` ↔ `material_cost_standards` using
  (`material_id`, fiscal year of the PO).
  - Fiscal year can be derived from `po_date`.
- Standardize cost to USD. Assume:
  - If `currency == "USD"`, use `po_value` / `actual_cost` as-is.
  - If `currency == "EUR"`, convert to USD using a flat factor
    **1 EUR = 1.08 USD** for both `po_value` and `actual_cost`.
- Create a cleaned `region_std` field that maps variants like "N. America" → "North America",
  "EU" → "Europe", "APAC" → "Asia Pacific", etc.
- Drop POs where `actual_delivery_date` is missing from any *delay-based* calculations,
  but keep them in spend metrics.

### 2. Feature Engineering
For each PO line, create:
- `delivery_delay_days` = `actual_delivery_date - scheduled_delivery_date`
  (in days, can be negative if early)
- `late_flag` = 1 if `delivery_delay_days > 3`, else 0
- `fiscal_year` from `po_date`
- `standard_cost_usd` from the cost table merge
- `cost_variance_pct` =
  (`actual_cost_usd - standard_cost_usd`) / `standard_cost_usd`

Also create a `po_value_usd` and `actual_cost_usd` after currency conversion.

### 3. Supplier-Level Aggregation
Aggregate to one row per `supplier_id`:
- `avg_delay_days`
- `%_late` (percentage of POs with `late_flag=1` out of those with known delivery dates)
- `avg_cost_var_pct` (mean `cost_variance_pct`)
- `total_orders` (count of PO lines)
- `avg_po_value_usd` (mean `po_value_usd`)
- `region_std`
- `tier`
- `supplier_size`
- `quality_rating`

Output this as `supplier_performance_summary.csv`.

### 4. Insights / Mini-Report
Answer briefly (Markdown or notebook cell output is fine):
1. Which supplier regions (normalized `region_std`) show the *highest* average delay?
2. Are late deliveries correlated with cost overruns? (e.g. compare avg `cost_variance_pct`
   for late vs. on-time POs)
3. Which 5 suppliers look best overall (low delay, low % late, and low avg cost variance)?

---

## Deliverables
1. `supplier_performance_summary.csv` with 1 row per supplier.
2. A short README / Markdown (or notebook cell) describing:
   - How you performed the joins
   - How you handled currency and missing dates
   - Key takeaways from the 3 questions above

This is primarily a **data wrangling and feature engineering** exercise. You do not
need to train a model.

---

## Scoring Rubric (Guidance)
- Correct / robust joins and fiscal year alignment .............. 30%
- Clean handling of messy data (currency, missing dates, regions) 30%
- Quality of engineered features and aggregations ................ 25%
- Clarity of findings / business interpretation .................. 15%

Target time: ~2 hours.
