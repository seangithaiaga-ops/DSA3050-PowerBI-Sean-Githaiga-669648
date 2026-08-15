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
