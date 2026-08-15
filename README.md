# DSA3050 Power BI Examination - Business Intelligence Solution
**Student Name:** [Sean Nderitu]  
**Registration Number:** [669648]  
**Dataset:** Hotel Booking Demand (Hospitality Performance & Cancellations) 
https://www.kaggle.com/datasets/jessemostipak/hotel-booking-demand?resource=download

---

## SECTION A: Dataset Selection & Understanding

### 1. Dataset Source
The dataset was obtained from Kaggle / Open Data Repository (Hotel Booking Demand Dataset).

### 2. What the Dataset Represents
This dataset contains reservation records for two hotels: a City Hotel and a Resort Hotel. It captures arrival dates, length of stay, customer types, deposit types, country of origin, average daily rates (ADR), and reservation cancellation statuses.

### 3. Selection Justification
Selected because it provides ~119,000 multi-dimensional records with an ideal balance of numerical metrics, categorical attributes, and dates. It supports critical hospitality KPIs like Cancellation Rate, Average Daily Rate (ADR), Lead Time, and Revenue per Available Room (RevPAR).

### 4. Main Available Variables
- **Hotel & Booking Status:** `hotel`, `is_canceled`, `lead_time`, `reservation_status`
- **Temporal Fields:** `arrival_date_year`, `arrival_date_month`, `arrival_date_day_of_month`, `stays_in_weekend_nights`, `stays_in_week_nights`
- **Guest & Market Attributes:** `adults`, `children`, `babies`, `country`, `market_segment`, `distribution_channel`, `customer_type`
- **Financial Metrics:** `adr` (Average Daily Rate), `deposit_type`, `required_car_parking_spaces`

### 5. Analytical Problem
Investigating high booking cancellation rates, seasonal revenue fluctuations, customer segment profitability, and lead-time patterns to optimize hotel inventory and cancellation policies.

### 6. Analytical Questions
1. What is the overall booking cancellation rate, and how much potential revenue is lost due to canceled bookings?
2. How does monthly ADR (Average Daily Rate) and booking volume vary between City Hotel and Resort Hotel?
3. What is the relationship between booking lead time and cancellation likelihood across different market segments?
4. Which customer types and market segments generate the highest average revenue per booking?
5. How do cancellation rates differ based on deposit types (e.g., Non-Refund vs. No Deposit)?

## SECTION B: POWER QUERY – DATA CLEANING & TRANSFORMATION

### 1. Inconsistent Category Formatting
- **Problem:** The `market_segment` and `distribution_channel` fields had irregular casing.
- **Transformation:** Applied `Format -> Capitalize Each Word`.
- **Reason:** Standardizes values so categories are not split during visual slicing.
- **Result:** Clean, uniformly capitalized market categories.

### 2. Disparate Date Components
- **Problem:** Arrival year, month, and day were stored across 3 separate columns.
- **Transformation:** Created custom `Arrival Date` column using M function `#date()`.
- **Reason:** Required for setting up continuous relationships with `DimDate`.
- **Result:** A single valid `Date` column for time intelligence.

### 3. Null and Missing Values
- **Problem:** The `country` field contained null/missing values.
- **Transformation:** Replaced null values with `"Unknown"`.
- **Reason:** Prevents missing value blanks in geography visuals and slicers.
- **Result:** Categorized unknown regions cleanly.

### 4. Separate Weekend/Weekday Nights
- **Problem:** Total length of stay was divided across weekend and week nights.
- **Transformation:** Added custom column `Total Nights` (`stays_in_weekend_nights` + `stays_in_week_nights`).
- **Reason:** Provides a single metric to evaluate guest duration.
- **Result:** Unified total stay night metric.

### 5. Missing Total Revenue Field
- **Problem:** Dataset only supplied Average Daily Rate (`adr`), not total reservation value.
- **Transformation:** Added custom column `Booking Revenue` (`Total Nights` * `adr`).
- **Reason:** Needed to compute core monetary performance measures.
- **Result:** Accurate overall booking monetary value.

### 6. Continuous Ungrouped Lead Times
- **Problem:** `lead_time` in days was too granular for visual grouping.
- **Transformation:** Added conditional column `Lead Time Group` bucketed into 0-7 days, 8-30 days, 31-90 days, 90+ days.
- **Reason:** Enables strategic lead time analysis.
- **Result:** Categorical lead time buckets.

### 7. Invalid Zero-Guest Records
- **Problem:** Occasional records showed 0 adults, 0 children, and 0 babies.
- **Transformation:** Applied row filter to exclude entries where total guest count equals zero.
- **Reason:** Cleans invalid transactional noise from analysis.
- **Result:** Dataset filtered to valid guest stays only.

### 8. Flat Table Schema Architecture
- **Problem:** Data was loaded as one single unnormalized flat table.
- **Transformation:** Referenced `FactBookings` to spawn `DimHotel` with unique hotel categories.
- **Reason:** Transitions architecture toward a normalized Star Schema.
- **Result:** Lean dimension lookup query created.

### 9. Binary Numeric Status Flag
- **Problem:** `is_canceled` was stored as a binary integer (`0` or `1`), making charts and legend titles unintuitive.
- **Transformation:** Created conditional column `Booking Status` mapping `1` to `"Canceled"` and `0` to `"Completed"`.
- **Reason:** Replaces raw codes with descriptive labels for professional visual storytelling.
- **Result:** Clear categorical labels (`Canceled` vs. `Completed`) across all visualizations.

### 10. Unsegmented Occupancy Demographics
- **Problem:** No explicit categorical field existed to distinguish family trips from non-family groups.
- **Transformation:** Added custom column `Guest Segment` categorizing bookings with children/babies as `"Family"` and all others as `"Adults Only"`.
- **Reason:** Enables targeted customer demographic filtering and spend evaluation.
- **Result:** Functional demographic dimension field for comparative profiling.

## SECTION C: DATA MODELLING

### Data Model Architecture & Technical Explanation

#### Model Overview
The data model uses a Star Schema architecture centered on `Fact_bookings`. `DimHotel`, `DimCustomer`, and `DimDate` surround the fact table, providing descriptive attributes used to filter transactional metrics. One-to-many relationships were established between each dimension and the central fact table.

#### 1. Why `Fact_bookings` Was Selected as the Fact Table
`Fact_bookings` contains individual reservation records at the lowest level of detail. It houses the primary numerical metrics—such as `adr` (Average Daily Rate), `Booking Revenue`, `Total Nights`, `lead_time`, and guest counts—as well as transactional status indicators (`is_canceled`). This makes it the central engine for aggregations and KPI calculations.

#### 2. Why Each Dimension Table Was Created
- **`DimHotel`:** Created to isolate property categories (`City Hotel` vs. `Resort Hotel`). Normalizing this field eliminates redundant text strings across ~119,000 fact rows and optimizes memory efficiency.
- **`DimCustomer`:** Created to group distinct customer types (`Contract`, `Group`, `Transient`, `Transient-Party`) for targeted demographic slicing without duplicating customer attributes across transactional rows.
- **`DimDate`:** Created as a dedicated calendar lookup covering continuous dates across all arrival years. A separate date table is required in Power BI to support Time Intelligence DAX functions (such as `SAMEPERIODLASTYEAR`) without relying on implicit auto-date hierarchies.

#### 3. Relationships Used
- `DimHotel[hotel]` -> `Fact_bookings[hotel]`
- `DimCustomer[customer_type]` -> `Fact_bookings[customer_type]`
- `DimDate[Date]` -> `Fact_bookings[Arrival Date]`

#### 4. Cardinality Decisions
**One-to-Many (1:*)** cardinality was applied across all three relationships:
- Each hotel property, customer type, and calendar date appears once (1) in its respective dimension table and connects to multiple reservation records (*) in `Fact_bookings`.

#### 5. Cross-Filter Direction Decisions
**Single Cross-Filter Direction** (Dimension -> Fact) was strictly enforced to ensure filter context flows unidirectionally from lookup dimensions down to `Fact_bookings`, eliminating ambiguous circular filter paths and performance degradation.

#### 6. Modelling Challenges Encountered & Resolutions
- **Date Type Mismatch Error:** `Arrival Date` in `Fact_bookings` initially had text formatting upon import, causing DAX calendar generation errors when calculating minimum and maximum year bounds.
- **Resolution:** Explicitly updated `Arrival Date` to a strict `Date` data type in Power Query prior to model loading, ensuring `DimDate` generated continuously and established clean referential integrity.

## SECTION D: DAX & BUSINESS CALCULATIONS

A total of 12 DAX measures were constructed within a dedicated `_AllMeasures` table to evaluate operational throughput, financial loss from cancellations, and temporal growth dynamics.

### Key DAX Explanations

1. **`Total Revenue`**
   - **Calculates:** Gross monetary revenue generated across reservation records.
   - **Utility:** Core high-level financial performance indicator.
   - **Functions Used:** `SUM()`
   - **Filter Context:** Evaluates dynamically based on active date ranges, hotel properties, or customer types.
   - **Dashboard Placement:** Page 1 Executive KPI Header.

2. **`Cancellation Rate %`**
   - **Calculates:** Proportion of total reservations that resulted in cancellation.
   - **Utility:** Measures inventory churn and revenue risk.
   - **Functions Used:** `DIVIDE()`, `CALCULATE()`
   - **Filter Context:** Evaluates divide logic with zero-handling across selected filter dimensions.
   - **Dashboard Placement:** Page 1 Executive KPI Card & Page 2 Cancellation Analysis.

3. **`Lost Revenue`**
   - **Calculates:** Monetary value attributed to canceled reservations.
   - **Utility:** Quantifies total financial leakage from booking cancellations.
   - **Functions Used:** `CALCULATE()`, `SUM()`
   - **Filter Context:** Explicitly filters `Fact_bookings` for `is_canceled = 1`.
   - **Dashboard Placement:** Page 3 Diagnostic Analysis.

4. **`Prior Year Revenue`**
   - **Calculates:** Total revenue generated in the equivalent historical period one year prior.
   - **Utility:** Establishes temporal comparative baselines.
   - **Functions Used:** `CALCULATE()`, `SAMEPERIODLASTYEAR()`
   - **Filter Context:** Requires continuous dates supplied by `DimDate`.
   - **Dashboard Placement:** Page 1 Multi-Year Trend Analysis.

5. **`YoY Revenue Growth %`**
   - **Calculates:** Percentage change in revenue compared to the prior year.
   - **Utility:** Evaluates annual business growth acceleration or deceleration.
   - **Functions Used:** `VAR`, `DIVIDE()`
   - **Filter Context:** Compares current time slice to the same window in `DimDate` for the prior year.
   - **Dashboard Placement:** Page 1 Executive KPI Header.

6. **`Revenue Contribution %`**
   - **Calculates:** Proportion of total global revenue generated by a specific segment or hotel category.
   - **Utility:** Pinpoints high-value revenue drivers across dimensions.
   - **Functions Used:** `DIVIDE()`, `CALCULATE()`, `ALL()`
   - **Filter Context:** Uses `ALL()` on the denominator to remove local dimension filters.
   - **Dashboard Placement:** Page 2 Market Segment Breakdown.

  ## SECTION E: DASHBOARD DESIGN & VISUALIZATION

The reporting solution is structured into 3 interactive, purpose-built report pages designed to guide stakeholders from macro executive performance down to root-cause operational diagnostics.

### Page Breakdown & Analytical Focus

#### Page 1: Executive Overview
- **Objective:** Provide senior leadership with immediate visibility into global operational health, top-line revenue performance, and overall cancellation impact.
- **Key Visuals:**
  - KPI Header Cards displaying `Total Revenue`, `Total Bookings`, `Cancellation Rate %`, and `ADR`.
  - Line Chart visualizing multi-year revenue trends across arrival dates.
  - Donut Chart breaking down overall revenue contribution by hotel property type (City vs. Resort).
- **Interactivity:** Global Year and Hotel Property slicers to dynamically filter executive metrics.

#### Page 2: Segment & Customer Analysis
- **Objective:** Analyze purchasing behavior across customer profiles, distribution paths, and market segments to identify profitability drivers.
- **Key Visuals:**
  - Clustered Bar Chart analyzing revenue yield across market segments (`Online TA`, `Offline TA/TO`, `Direct`, `Corporate`).
  - Stacked Column Chart evaluating total booking volume across customer types (`Transient`, `Contract`, `Group`) split by completion status.
  - Summary Table mapping top international customer origins by revenue.
- **Interactivity:** Cross-filtering enabled across customer demographics and market channels.

#### Page 3: Diagnostic & Cancellation Analysis
- **Objective:** Investigate booking attrition patterns, lead-time vulnerabilities, and financial loss attributes to inform revenue protection strategies.
- **Key Visuals:**
  - Diagnostic KPI Cards displaying `Lost Revenue`, `Total Canceled Bookings`, and `High Lead Time Bookings`.
  - Clustered Column Chart illustrating how cancellation rates scale across lead-time risk buckets (Last Minute vs. Long Lead).
  - Financial Loss Matrix cross-tabulating lost revenue across market segments and deposit types.
- **Interactivity:** Lead-time group slicers allowing dynamic deep-dives into high-risk reservation windows.

