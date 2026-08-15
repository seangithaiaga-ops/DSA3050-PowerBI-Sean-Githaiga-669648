# DSA3050 Power BI Examination — Business Intelligence Solution

**Student Name:** Sean Nderitu
**Registration Number:** 669648
**Dataset:** Hotel Booking Demand (Hospitality Performance & Cancellations)
Source: https://www.kaggle.com/datasets/jessemostipak/hotel-booking-demand?resource=download

---

## SECTION A: Dataset Selection & Understanding

### 1. Dataset Source
Pulled from Kaggle — the Hotel Booking Demand dataset (jessemostipak).

### 2. What the Dataset Represents
This is reservation-level data from two hotels, a City Hotel and a Resort Hotel. Each row is a single booking, with arrival dates, how long the guest stayed, what kind of customer made the booking, the deposit arrangement, where they're from, the average daily rate they paid, and whether the booking eventually got cancelled.

### 3. Why I Picked This Dataset
It sits at a good size — around 119,000 rows — with a genuine mix of numbers, categories, and dates to work with, rather than something already summarized into a handful of columns. It also maps naturally onto real hospitality KPIs: cancellation rate, ADR, lead time, and revenue per booking, which gave me enough to build a proper analytical story instead of just a few charts.

### 4. Main Variables
- **Booking status:** `hotel`, `is_canceled`, `lead_time`, `reservation_status`
- **Dates and stay length:** `arrival_date_year`, `arrival_date_month`, `arrival_date_day_of_month`, `stays_in_weekend_nights`, `stays_in_week_nights`
- **Guest and market info:** `adults`, `children`, `babies`, `country`, `market_segment`, `distribution_channel`, `customer_type`
- **Money-related fields:** `adr` (average daily rate), `deposit_type`, `required_car_parking_spaces`

### 5. The Problem I'm Investigating
Why bookings get cancelled, how revenue moves through the year, which customer segments actually make the hotel money, and how booking lead time plays into cancellation risk — with the goal of pointing toward better cancellation policy and inventory decisions.

### 6. Analytical Questions
1. What's the overall cancellation rate, and roughly how much revenue does it cost the hotel?
2. How do ADR and booking volume change month to month, and does that differ between the two hotels?
3. Does lead time predict cancellation likelihood, and does that relationship change depending on market segment?
4. Which customer types and market segments bring in the most revenue per booking?
5. Do cancellation rates differ noticeably by deposit type — for example, Non-Refund vs. No Deposit?

---

## SECTION B: Power Query — Data Cleaning & Transformation

### 1. Inconsistent Category Formatting
**Problem:** `market_segment` and `distribution_channel` had some irregular casing across values.
**Transformation:** Used Transform → Format → Capitalize Each Word on both columns.
**Reason:** Keeps category names consistent so they don't accidentally split into separate slices in charts and slicers.
**Result:** Both fields now show clean, uniformly capitalized categories.

### 2. Date Split Across Three Columns
**Problem:** The arrival date was broken up into `arrival_date_year`, `arrival_date_month`, and `arrival_date_day_of_month` — three separate columns instead of one usable date.
**Transformation:** Built a custom `Arrival Date` column using the `#date()` function to stitch the three parts back together, then set its type to Date.
**Reason:** I needed one real date field to build a relationship to `DimDate` and to run any time-intelligence DAX later on.
**Result:** A single, properly typed `Arrival Date` column that the model can actually use.

### 3. Missing Values in Country
**Problem:** A number of rows had a blank `country` field.
**Transformation:** Replaced the blanks with `"Unknown"`.
**Reason:** Blank values break map visuals and mess with slicer counts, so it's better to label them explicitly than leave them empty.
**Result:** Every row now has a country value, with unresolved ones clearly marked rather than hidden.

### 4. Stay Length Split Into Weekday/Weekend
**Problem:** Length of stay was divided between `stays_in_weekend_nights` and `stays_in_week_nights`, with no combined total.
**Transformation:** Added a custom column, `Total Nights`, as the sum of the two.
**Reason:** Needed one number to represent how long a guest actually stayed, both for reporting and for calculating revenue.
**Result:** A single `Total Nights` metric usable across the model.

### 5. No Total Revenue Field
**Problem:** The dataset only gives the average daily rate (`adr`) — there's no field for what a booking was actually worth in total.
**Transformation:** Added a custom column, `Booking Revenue`, calculated as `Total Nights * adr`.
**Reason:** Almost every financial KPI I wanted (total revenue, lost revenue, revenue by segment) needed a per-booking revenue figure to work from.
**Result:** A `Booking Revenue` column that feeds directly into the DAX measures in Section D.

### 6. Lead Time Was Too Granular
**Problem:** `lead_time` is stored in raw days, which isn't very useful for grouping or slicing in a dashboard.
**Transformation:** Added a conditional column, `Lead Time Group`, bucketing bookings into Last Minute (0–7 days), Short Lead (8–30 days), Medium Lead (31–90 days), and Long Lead (90+ days).
**Reason:** Bucketed categories are far easier to slice by and compare visually than hundreds of distinct day values.
**Result:** A `Lead Time Group` field used throughout the diagnostic page.

### 7. Zero-Guest Records
**Problem:** A small number of rows had zero adults, zero children, and zero babies — bookings with nobody actually staying, which looks like bad data rather than a real reservation.
**Transformation:** Filtered these rows out.
**Reason:** These records would distort occupancy and guest-count metrics if left in.
**Result:** Only rows representing valid, occupied stays remain in the dataset.

### 8. Everything Was One Flat Table
**Problem:** The raw data loads as a single unnormalized table, with no separation between transactional data and descriptive attributes.
**Transformation:** Referenced (not duplicated) `FactBookings` to create a new query, `DimHotel`, keeping only the `hotel` column and removing duplicates.
**Reason:** First step toward a proper star schema — pulls the hotel property out as its own lookup table instead of repeating the text across ~119,000 rows.
**Result:** A lean two-value `DimHotel` dimension ready to relate back to the fact table.

### 9. Cancellation Flag Was a Raw 0/1
**Problem:** `is_canceled` is stored as a binary integer, which isn't very readable in chart legends or tooltips.
**Transformation:** Added a conditional column, `Booking Status`, mapping `1` to `"Canceled"` and `0` to `"Completed"`.
**Reason:** Labels read better than raw codes, especially on an executive-facing page.
**Result:** A `Booking Status` field showing "Canceled" or "Completed" instead of 1s and 0s.

### 10. No Family vs. Non-Family Distinction
**Problem:** There was no field separating bookings with children from bookings without — useful context for demographic analysis that didn't exist yet.
**Transformation:** Added a custom column, `Guest Segment`, labeling any booking with children or babies as `"Family"` and everything else as `"Adults Only"`.
**Reason:** Wanted a quick way to compare spend and stay length between family and non-family guests.
**Result:** A working `Guest Segment` field used in the customer analysis page.

---

## SECTION C: Data Modelling

### Model Overview
The model is built as a star schema centered on `Fact_bookings`. `DimHotel`, `DimCustomer`, and `DimDate` sit around it, each providing descriptive attributes that filter the fact table. All three relationships are one-to-many, dimension to fact.

### 1. Why `Fact_bookings` Is the Fact Table
It holds the data at the lowest level of detail — one row per reservation — and carries all the core numeric fields (`adr`, `Booking Revenue`, `Total Nights`, `lead_time`, guest counts) plus the cancellation flag. Everything gets aggregated from here.

### 2. Why Each Dimension Exists
- **`DimHotel`** — pulls the two hotel properties (City Hotel, Resort Hotel) out into their own table instead of repeating that text on every one of the ~119,000 fact rows.
- **`DimCustomer`** — groups the distinct customer types (Contract, Group, Transient, Transient-Party) so I can slice by customer profile without duplicating that attribute across every transaction.
- **`DimDate`** — a proper calendar table spanning all the arrival years in the dataset. Power BI needs a dedicated date table like this for time-intelligence functions such as `SAMEPERIODLASTYEAR` to work correctly — the built-in auto date/time hierarchy isn't reliable enough for that.

### 3. Relationships
- `DimHotel[hotel]` → `Fact_bookings[hotel]`
- `DimCustomer[customer_type]` → `Fact_bookings[customer_type]`
- `DimDate[Date]` → `Fact_bookings[Arrival Date]`

### 4. Cardinality
All three relationships are one-to-many (1:*) — each hotel, each customer type, and each calendar date appears once in its dimension table and connects to many rows in `Fact_bookings`.

### 5. Cross-Filter Direction
Set to single direction, dimension → fact, across the board. This keeps filter context flowing one way and avoids the ambiguous filter paths and performance issues that bidirectional relationships can cause.

### 6. A Modelling Problem I Ran Into
When I first built `Arrival Date` in Power Query, it loaded as text instead of a date. That silently broke `DimDate` — the calendar table couldn't calculate its min/max year range properly, which threw off the whole relationship. Fixed it by explicitly setting `Arrival Date` to a Date type in Power Query before loading the model, which let `DimDate` build correctly and the relationship snap into place without issues.

---

## SECTION D: DAX & Business Calculations

I built 12 measures in total, grouped into a dedicated `_AllMeasures` table, covering booking volume, cancellation impact, and revenue growth over time. Below are the six I think are most important to the analysis.

### 1. `Total Revenue`
Sums `Booking Revenue` across all reservations — the headline financial number on the dashboard. Uses `SUM()`, and responds dynamically to whatever date range, hotel, or customer filter is active. Sits at the top of Page 1 as a KPI card.

### 2. `Cancellation Rate %`
Divides cancelled bookings by total bookings using `DIVIDE()` and `CALCULATE()`, with zero-handling built in so it doesn't error out on an empty filter context. This is really the measure the whole exam is built around — it shows up as a KPI card on Page 1 and drives most of the diagnostic work on Page 2.

### 3. `Lost Revenue`
Uses `CALCULATE()` with an explicit filter for `is_canceled = 1` to isolate the revenue tied to cancelled bookings only. This quantifies the actual financial cost of cancellations rather than just the rate, and it's featured on Page 3 where I dig into why cancellations happen.

### 4. `Prior Year Revenue`
`CALCULATE()` combined with `SAMEPERIODLASTYEAR()` to pull the equivalent revenue figure from a year earlier. This only works because of the dedicated `DimDate` table with continuous dates — it's used as the baseline for the year-over-year comparison on Page 1.

### 5. `YoY Revenue Growth %`
Built with a `VAR` to hold the prior-year figure and `DIVIDE()` to calculate the percentage change against it. Shows whether the business is growing or shrinking year on year, and sits alongside `Total Revenue` on the executive KPI header.

### 6. `Revenue Contribution %`
Uses `DIVIDE()` and `CALCULATE()` with `ALL()` applied to the denominator, so the total in the denominator ignores whatever local filter is active (e.g. a selected market segment) while the numerator still respects it. This is what lets the segment breakdown on Page 2 show each segment's share of total revenue rather than just its raw value.

---

## SECTION E: Dashboard Design & Visualization

The report is split into three pages, moving from a high-level executive view down to root-cause diagnostics.

### Page 1: Executive Overview
Meant to give leadership the big picture at a glance. KPI cards up top for `Total Revenue`, `Total Bookings`, `Cancellation Rate %`, and ADR, a line chart tracking revenue across arrival dates over the full multi-year span, and a donut chart splitting revenue between the City and Resort hotels. Year and hotel-property slicers sit at the top so the whole page filters together.

### Page 2: Segment & Customer Analysis
Digs into who's actually booking and how much they're worth. A clustered bar chart compares revenue across market segments (Online TA, Offline TA/TO, Direct, Corporate), a stacked column chart breaks down booking volume by customer type and completion status, and a summary table ranks top countries of origin by revenue. Everything here cross-filters against the customer and channel selections.

### Page 3: Diagnostic & Cancellation Analysis
This is where I try to explain *why* cancellations happen, not just how many. KPI cards for `Lost Revenue`, total cancelled bookings, and high-lead-time bookings sit up top, a clustered column chart shows how cancellation rate scales across the lead-time buckets from Section B, and a matrix cross-tabs lost revenue by market segment and deposit type. A lead-time slicer lets you zoom into specific risk windows.

---

## SECTION F: Business Insights & Recommendations

### Key Findings
- **Cancellations are eating into revenue more than expected.** The overall cancellation rate is high enough that it's a real drag on projected income, not just background noise — a meaningful chunk of "booked" revenue never actually converts.
- **Long lead times are risky.** Bookings made more than 90 days out cancel far more often than last-minute ones. Without a non-refundable deposit attached, a long booking window seems to invite cancellation.
- **OTAs bring volume but also risk.** Online Travel Agencies generate the most bookings overall, but they also carry noticeably higher cancellation rates than Direct or Corporate channels.
- **Family vs. transient guests behave differently.** Transient guests pay the highest ADR, while family bookings stay longer on average but show up in smaller numbers overall.

### Recommendations
1. **Tighten deposit policy on long-lead bookings.** Require a non-refundable deposit, or a tiered cancellation fee, for any booking made more than 60 days out to cut down on high-lead-time attrition.
2. **Push direct bookings harder.** Offer perks like a free upgrade or flexible late checkout for guests booking straight through the hotel's own website, to pull volume away from high-commission, high-cancellation OTA channels.
3. **Run length-of-stay promotions in off-peak months.** Minimum-stay discounts during slower periods could help stabilize occupancy and lift total revenue per guest.
4. **Consider dynamic pricing for peak-season transient and corporate travelers.** Since these segments already carry the highest ADR, adjusting pricing upward during peak arrival months could capture more of that willingness to pay.

