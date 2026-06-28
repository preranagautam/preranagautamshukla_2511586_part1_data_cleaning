# Part 1: Business Data Cleaning, Validation & Excel Reporting

## Problem Summary

A retail company exports order-level sales data from multiple internal systems. The raw dataset (`raw_orders.xlsx`) contained numerous data quality issues including inconsistent text formatting, mixed date formats, duplicate records, missing values, invalid discounts, and sales calculation mismatches. This project cleans and validates the data, applies business rules, documents all issues found, and produces summary pivot reports for business review.

---

## Dataset Description

- **File:** `data/raw_orders.xlsx`
- **Raw Records:** 932 rows × 21 columns
- **Data Period:** January 2024 – October 2025
- **Source:** Multiple internal retail order management systems

### Columns in the Raw Dataset

| Column | Description |
|--------|-------------|
| order_id | Unique order identifier |
| order_date | Date the order was placed |
| ship_date | Date the order was shipped |
| customer_id | Unique customer identifier |
| customer_name | Full name of the customer |
| segment | Customer segment (Consumer, Corporate, Home Office, Small Business) |
| region | Geographic region (North, South, East, West) |
| state | Indian state |
| city | City of delivery |
| category | Product category (Furniture, Technology, Office Supplies) |
| sub_category | Product sub-category |
| product_name | Name of the product |
| ship_mode | Shipping method used |
| quantity | Number of units ordered |
| unit_price | Price per unit |
| discount | Discount applied (decimal format, 0 to 1) |
| sales | Reported sales value |
| cost | Cost of goods |
| profit | Reported profit |
| payment_status | Payment status (Paid, Pending, Refunded, Failed) |
| order_status | Order status (Completed, Cancelled, Returned) |

---

## Tools Used

- **Microsoft Excel** — Primary tool for data cleaning, text standardization, calculated columns, pivot tables, and reports
- **Python 3 (pandas)** — Used for date parsing across 6 inconsistent formats
- **openpyxl** — Python library for reading and writing Excel files
- **Git & GitHub** — Version control and submission

---

## Cleaning Steps Performed

### Step 1 — Preserved Raw Data
Original `raw_orders.xlsx` kept unchanged. All cleaning done in a separate `cleaned_orders.xlsx`.

### Step 2 — Removed Exact Duplicate Rows
Used Excel's **Data → Remove Duplicates** to identify and remove **20 exact duplicate rows**. First occurrence was kept.

### Step 3 — Flagged Conflicting Duplicate Order IDs
After removing exact duplicates, **12 unique order IDs** still appeared more than once with different field values (different sales amounts, different statuses). These were kept in the cleaned file and flagged in the `data_quality_flag` column for business review.

### Step 4 — Cleaned All Text Columns
Applied the following to all 10 text columns (`customer_name`, `segment`, `region`, `state`, `city`, `category`, `sub_category`, `ship_mode`, `payment_status`, `order_status`):
- `TRIM()` to remove leading, trailing, and extra internal spaces
- `PROPER()` to standardize all values to Title Case
- `SUBSTITUTE()` to fix double spaces (e.g. "Small  Business" → "Small Business")
- Find & Replace for remaining inconsistencies

**Examples of values corrected:**
- "COMPLETED", "completed", "  Completed" → "Completed"
- "CANCELLED", "cancelled" → "Cancelled"
- "PENDING", "  Pending", "failed" → "Pending", "Failed"
- "SMALL BUSINESS", "Small  Business" → "Small Business"
- "NORTH", "north", "  North" → "North"
- "OFFICE SUPPLIES", "Office  Supplies" → "Office Supplies"
- "STANDARD CLASS", "Standard  Class" → "Standard Class"

### Step 5 — Filled Missing Values (Business Rules)
- **25 missing `region` values** → Filled with "Unknown"
- **21 missing `ship_mode` values** → Filled with "Unknown"
- **18 missing `discount` values** → Set to 0 (all had valid quantity, unit_price, and sales fields)

### Step 6 — Cleaned and Standardized Dates
Dates were stored in 6 different formats across `order_date` and `ship_date`. Used Python to parse all formats and standardize to `YYYY-MM-DD`. Created `order_date_clean` and `ship_date_clean` columns. Original date columns were preserved unchanged.

**Date formats found:**
- `21 Jul 2024` (DD Mon YYYY)
- `08/31/2024` (MM/DD/YYYY)
- `2024-05-24` (YYYY-MM-DD)
- `28-11-2024` (DD-MM-YYYY)
- `15 Jun 2024` (DD Mon YYYY)
- `04/16/2025` (MM/DD/YYYY)

Identified **21 records** where ship_date was earlier than order_date — flagged as invalid.

### Step 7 — Cleaned Discount Values
- Converted string discounts "70%" and "85%" to decimal equivalents (0.70, 0.85)
- Flagged **15 negative discount** records as Invalid
- Flagged **15 discounts above 50%** as Invalid (unusually high; threshold based on retail business norms — see Assumptions)
- Set 18 missing discounts to 0 where all other sales fields were valid

### Step 8 — Created All Calculated Columns
Added the following columns to `cleaned_orders.xlsx`:

| Column | Formula / Logic |
|--------|----------------|
| `cleaned_discount` | Standardized discount (strings converted, negatives flagged, missing → 0) |
| `calculated sales` | quantity × unit_price × (1 − cleaned_discount) |
| `calculated_profit` | calculated_sales − cost |
| `profit_margin` | calculated_profit ÷ calculated_sales |
| `shipping_delay_days` | ship_date − order_date (in days) |
| `order_month` | Month number extracted from order_date |
| `order_year` | Year extracted from order_date |
| `date_issue` | Flags date problems (OK / Ship date before order date) |
| `discount_issue` | Flags discount problems |
| `sales_vs_calculated_sales` | Difference between reported and calculated sales |
| `data_issue` | Combined description of all issues per record |
| `data_quality_flag` | Overall record quality (clean / warning / invalid) |

---

## Business Rules Applied

| Rule Area | Action Taken |
|-----------|-------------|
| Missing region | Filled as "Unknown"; flagged in quality report |
| Missing ship_mode | Filled as "Unknown"; flagged in quality report |
| Missing discount | Set to 0 where quantity, unit_price, and sales were all valid |
| Negative discount | Flagged as Invalid in `discount_issue` and `data_quality_flag` |
| Discount above allowed range (>50%) | Flagged as Invalid |
| Cancelled orders | Excluded from completed sales pivot summaries |
| Failed payments | Excluded from completed sales pivot summaries |
| Refunded orders | Summarized separately in pivot sheet |
| Ship date before order date | Flagged as Invalid in `date_issue` column |
| Exact duplicate rows | Removed; documented in data quality report |
| Conflicting duplicate order IDs | Kept in cleaned file; flagged for review |

---

## Summary of Data Quality Issues Found

| Issue | Count |
|-------|-------|
| Raw records (original file) | 932 |
| Exact duplicate rows removed | 20 |
| Records in cleaned file | 912 |
| Missing region (filled "Unknown") | 25 |
| Missing ship_mode (filled "Unknown") | 21 |
| Missing discount (set to 0) | 18 |
| Negative discounts flagged | 15 |
| Discounts above 50% flagged | 15 |
| Date issues (ship before order) | 21 |
| Sales vs calculated sales mismatches | 33 |
| Conflicting duplicate order IDs | 12 unique IDs (24 records flagged) |
| **Clean records** | **786** |
| **Warning records** | **43** |
| **Invalid records** | **83** |

### Order Status Breakdown (after cleaning)

| Status | Count |
|--------|-------|
| Completed | 604 |
| Returned | 163 |
| Cancelled | 145 |

### Payment Status Breakdown (after cleaning)

| Status | Count |
|--------|-------|
| Paid | 686 |
| Pending | 86 |
| Refunded | 71 |
| Failed | 69 |

---

## Summary of Pivot Reports (`pivot_summary.xlsx`)

All sales and profit figures below are based on **Completed + Paid records only (581 records)** as per business rules.

**Total Completed Sales: ₹57,08,361.35 | Total Profit: ₹16,73,084.17 | Overall Profit Margin: 29.31%**

### Pivot 1 — Sales and Profit by Region
*(Sorted by Total Sales, descending)*

| Region | Total Sales (₹) | Total Profit (₹) |
|--------|----------------|-----------------|
| South | 15,10,590.80 | 4,34,277.28 |
| West | 14,72,204.30 | 4,22,755.03 |
| East | 13,71,633.39 | 4,12,183.27 |
| North | 11,86,383.57 | 3,51,861.04 |
| Unknown | 1,67,549.28 | 52,007.54 |

### Pivot 2 — Sales and Profit by Category & Sub-Category
*(Sorted by Total Sales, descending)*

| Category | Total Sales (₹) | Total Profit (₹) |
|----------|----------------|-----------------|
| Technology | 20,53,014.91 | 6,21,704.99 |
| Furniture | 18,85,131.01 | 5,52,540.39 |
| Office Supplies | 17,70,215.44 | 4,98,838.80 |

Top sub-categories: Copiers (₹6,57,117.51), Storage (₹5,17,608.52), Chairs (₹5,63,619.47)

### Pivot 3 — Order Count by Ship Mode
*(All 912 records)*

| Ship Mode | Order Count |
|-----------|-------------|
| Standard Class | 242 |
| First Class | 223 |
| Second Class | 222 |
| Same Day | 204 |
| Unknown | 21 |

### Pivot 4 — Profit Margin by Customer Segment

| Segment | Total Sales (₹) | Total Profit (₹) | Profit Margin |
|---------|----------------|-----------------|---------------|
| Home Office | 15,07,456.72 | 4,54,988.32 | 30.18% |
| Consumer | 13,73,712.34 | 4,09,149.67 | 29.78% |
| Corporate | 12,81,492.56 | 3,75,021.66 | 29.26% |
| Small Business | 15,45,699.74 | 4,33,924.53 | 28.07% |

### Pivot 5 — Refunded/Cancelled/Failed Orders by Region

**Failed & Refunded payments:**

| Region | Failed | Refunded | Total |
|--------|--------|----------|-------|
| North | 20 | 20 | 40 |
| South | 20 | 20 | 40 |
| West | 14 | 14 | 28 |
| East | 13 | 14 | 27 |
| Unknown | 2 | 3 | 5 |
| **Total** | **69** | **71** | **140** |

**Cancelled & Returned orders:**

| Region | Cancelled | Returned | Total |
|--------|-----------|----------|-------|
| North | 48 | 37 | 85 |
| South | 35 | 37 | 72 |
| East | 30 | 45 | 75 |
| West | 30 | 40 | 70 |
| Unknown | 2 | 4 | 6 |
| **Total** | **145** | **163** | **308** |

### Pivot 6 — Monthly Sales Trend

| Year | Total Sales (₹) | Orders |
|------|----------------|--------|
| 2024 | 30,71,051.00 | 328 |
| 2025 | 26,37,310.35 | 274 |

**Top 5 months by sales:**
1. February 2025 — ₹3,94,995.72
2. June 2024 — ₹3,91,026.42
3. October 2024 — ₹3,25,433.92
4. March 2025 — ₹3,22,614.29
5. September 2025 — ₹3,17,689.50

---

## Key Business Insights

1. **South region leads in sales** — ₹15.1L in completed sales, followed closely by West (₹14.7L). North is the lowest at ₹11.9L, suggesting potential for growth or operational challenges in that region.

2. **Technology is the top-performing category** — ₹20.5L in sales (36% of total), with Copiers alone contributing ₹6.6L. Furniture and Office Supplies are close behind.

3. **Home Office segment has the highest profit margin (30.18%)** — Small Business has the lowest at 28.07%, indicating possible higher discounting or cost pressures in that segment.

4. **86.2% of records are clean** — 786 out of 912 records passed all quality checks, indicating the dataset is largely reliable but requires attention to discount and date fields.

5. **33 sales mismatches detected** — The reported `sales` column does not match the formula `quantity × unit_price × (1 − discount)` for 33 records, suggesting possible system-level rounding differences or data entry errors.

6. **308 cancelled/returned orders across all regions** — North has the highest count (85 records), which may indicate fulfillment or customer satisfaction issues in that region.

7. **Standard Class is the most used shipping mode** — 242 orders (26.5%), but all four modes are used fairly evenly, suggesting no strong customer preference.

8. **25 records have unknown region** — These contribute ₹1.67L in sales and cannot be attributed to any region, highlighting the need for better data capture at the point of order entry.

9. **February 2025 was the peak sales month** — ₹3,94,995.72, nearly double some slower months. Seasonal trends should be investigated for inventory and staffing planning.

10. **30 discount records were invalid** — 15 negative and 15 excessively high discounts indicate a need for validation controls at the data entry level in the source systems.

---

## Assumptions and Limitations

### Assumptions
1. Discount values are stored as fractions (0.0 to 1.0), not percentages — "70%" and "85%" were converted to 0.70 and 0.85 accordingly
2. A discount greater than 50% (0.5) is considered "above allowed range" and flagged as Invalid, as no maximum discount policy was specified in the assignment. This threshold is based on general retail business norms
3. For missing discount values, 0 was assumed where all other sales fields (quantity, unit_price, sales) were valid and present
4. Ambiguous date formats (e.g. where MM/DD and DD/MM could both be valid) were parsed as MM/DD/YYYY based on the data context
5. For conflicting duplicate order IDs, both records were retained rather than arbitrarily deleting one — this decision requires domain expert review
6. Only records with `order_status = Completed` AND `payment_status = Paid` were included in sales and profit pivot summaries
7. The priority order for the `issue_type` label in Pivot 5 is: Cancelled > Returned > Failed > Refunded

### Limitations
1. **Date ambiguity** — Some dates could legitimately be read as either DD/MM or MM/DD; the parsed version may be incorrect for a small number of records
2. **Conflicting duplicates unresolved** — The 12 conflicting order IDs were flagged but not resolved; a business stakeholder must determine the correct record
3. **"Unknown" fills are placeholders** — The 25 unknown regions and 21 unknown ship modes should be sourced from the original system rather than left as Unknown
4. **Sales mismatch root cause unknown** — The 33 mismatches between reported and calculated sales may be due to rounding in the source system, promotional overrides, or data entry errors — this cannot be determined from the data alone
5. **No product catalog validation** — Product names were standardized for spacing and casing but were not validated against a master product catalog
6. **Discount threshold is assumed** — The 50% maximum discount threshold was not provided in the assignment; actual business policy may differ

---

## Screenshots

| File | Description |
|------|-------------|
| `screenshots/raw_data_preview.png` | Raw dataset before any cleaning — shows mixed date formats, casing issues, and inconsistent values |
| `screenshots/cleaned_data_preview.png` | Cleaned dataset with all calculated columns — shows standardized values and quality flags |
| `screenshots/pivot_summary_1.png` | Sales and Profit by Region pivot — sorted by total sales descending |
| `screenshots/pivot_summary_2.png` | Monthly Sales Trend pivot — showing 2024 vs 2025 monthly breakdown |
