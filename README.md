# Pascal Hotels
## Table of Contents

- [Project Overview](#project-overview)
- [Data Sources](#data-sources)
- [Tools](#tools)
- [Data Preparation](#data-preparation)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Data Analysis](#data-analysis)
- [Results/Findings](#results/findings)
- [Recommendations](#recommendations)

## Project Overview
This a analysis project of Pascal Hotels datasets across its different hotels in the country and the overview of revenue as thse management trys to gain key insights for business progression.

![Screenshot (82)](https://github.com/user-attachments/assets/c81589ff-acaf-471b-9010-e04fcf46a1b8)



## Data Source
The primary datasets include: "dim_date.csv", "dim_hotels.csv", "dim_rooms.csv", "fact_booking_clean" and "facts_aggregated_bookings_clean.csv" these contain the the key information of the hotel operations.

## Tools Used
- MS SQL Server
- MS Power BI

## Data Cleaning & Preparation
In the intial data preparation phase, we peformed the following tasks:

- Created a database on SQL server for Mitron Bank.
- Then added tables to the database using the csv files.
- Joined the tables to form 1 with comprehensive report.
- Then transformed the data to Power BI for visualisation.

## Explanatory Data Analysis
EDA involved looking at the sample dataset provided by the bank as they seek to launch the new credit card and answer key questions such as:
1. What platform is likely to be used by customers for bookings?
2. Which hotel bring in the most revenue?
3. Which class of rooms are most prefered.
4. Which days and months did the hotels generate most revenue?
5. How many of the bookings where actually successful?

## Data Analysis
```sql
-- Use Common Table Expressions (CTE) to join datasets with alternative column names
WITH FactBookings_CTE AS (
    SELECT
        fb.booking_id AS BookingID,
        fb.booking_date AS BookingDate,
        fb.check_in_date AS CheckInDate,
        fb.checkout_date AS CheckOutDate,
        fb.property_id AS PropertyID,
        dh.property_name AS PropertyName,
        dh.category AS HotelCategory,
        dh.city AS HotelCity,
        fb.room_category AS RoomID,
        dr.room_class AS RoomClass,
        fb.no_guests AS GuestCount,
        fb.booking_status AS BookingStatus,
        fb.revenue_generated AS RevenueGenerated,
        fb.revenue_realized AS RevenueRealized,
        fb.booking_platform AS BookingPlatform,
        dd.day_type AS DayType,
        dd.mmm_yy AS MonthYear
    FROM fact_bookings_clean fb
    INNER JOIN dbo.dim_hotels dh ON fb.property_id = dh.property_id
    INNER JOIN dbo.dim_rooms dr ON fb.room_category = dr.room_id
    INNER JOIN dbo.dim_date dd ON fb.check_in_date = dd.date
    WHERE fb.booking_id IS NOT NULL
      AND fb.property_id IS NOT NULL
      AND fb.room_category IS NOT NULL
      AND fb.check_in_date IS NOT NULL
),

FactAggregatedBookings_CTE AS (
    SELECT
        fab.property_id AS PropertyID,
        dh.property_name AS PropertyName,
        dh.category AS HotelCategory,
        dh.city AS HotelCity,
        fab.check_in_date AS CheckInDate,
        fab.room_category AS RoomID,
        dr.room_class AS RoomClass,
        fab.successful_bookings AS GuestCount,
        fab.capacity AS Capacity,
        'aggregated' AS BookingStatus,
        dd.day_type AS DayType,
        dd.mmm_yy AS MonthYear
    FROM fact_aggregated_bookings_clean fab
    INNER JOIN dbo.dim_hotels dh ON fab.property_id = dh.property_id
    INNER JOIN dbo.dim_rooms dr ON fab.room_category = dr.room_id
    INNER JOIN dbo.dim_date dd ON fab.check_in_date = dd.date
    WHERE fab.property_id IS NOT NULL
      AND fab.room_category IS NOT NULL
      AND fab.check_in_date IS NOT NULL
)

-- Combine the two CTEs
SELECT BookingID, BookingDate, CheckInDate, CheckOutDate, PropertyID, PropertyName, HotelCategory, HotelCity, RoomID, RoomClass, GuestCount, BookingStatus, RevenueGenerated, RevenueRealized, BookingPlatform, DayType, MonthYear
FROM FactBookings_CTE

UNION ALL

SELECT NULL AS BookingID, NULL AS BookingDate, CheckInDate, NULL AS CheckOutDate, PropertyID, PropertyName, HotelCategory, HotelCity, RoomID, RoomClass, GuestCount, BookingStatus, NULL AS RevenueGenerated, NULL AS RevenueRealized, NULL AS BookingPlatform, DayType, MonthYear
FROM FactAggregatedBookings_CTE;
```
## Results/Findings
### 📊 Booking Status Distribution (Top Left)
- Most bookings are "Checked Out" – significantly higher than all other statuses.
- "Cancelled" bookings are the next most frequent.
- "No Show" and "Aggregated" bookings are minimal.

### 💵Revenue by Booking Platform (Top Right)
- The "Others" platform generated the highest revenue.
- "Makeyourtrip" follows as the second-largest contributor.
- Platforms like "logtrip", "direct online", and "tripster" show moderate revenue.
- "Direct offline" contributes the least.

### 🏨 Revenue by Property Name (Bottom Left)
- Atliq Palace and Atliq Exotica are top revenue-generating properties.
- Atliq City and Atliq Blu also perform well.
- Atliq Seasons generates the least revenue by a significant margin.

### 🛏️ Revenue by Room Class (Bottom Center)
- "Elite" and "Premium" room classes are top earners.
- "Standard" and "Presidential" follow, with Presidential being the lowest among the four.

### 📅 Revenue by Month & Day Type (Bottom Right)
- Weekends generate significantly more revenue than weekdays.
- Revenue peaks in May and June for both weekdays and weekends.
- There's a slight decline in July for both day types.

## Recommendations 

💰 1. Focus on High-Revenue Properties
- Properties to prioritize: Atliq Palace, Atliq Exotica, Atliq City
- Recommendation: Invest more in marketing, service quality, and capacity expansion for these top-performing properties.
- Action: Offer exclusive packages or loyalty programs to retain high-paying guests.

🛏️ 2. Promote High-Performing Room Classes
- Top Room Classes: Elite and Premium
- Recommendation: Upsell Elite and Premium rooms through promotions or bundled services (e.g., breakfast, late checkout).
- Action: Highlight these rooms on booking platforms and during direct bookings.

📅 3. Maximize Weekend Revenue
- Observation: Weekends generate more revenue than weekdays.
- Recommendation: Launch targeted weekend getaway deals, seasonal promotions, or experiences tailored to weekend travelers.
- Action: Adjust staffing and inventory planning to handle weekend demand peaks.

🌐 4. Optimize Platform Strategy
- Top platforms: “Others” and “Makeyourtrip”
- Recommendation: Identify what makes “Others” successful—could be B2B channels or corporate bookings. Increase collaboration or negotiate better terms with high-performing platforms.
- Action: De-prioritize or renegotiate with low-performing platforms like "direct offline" and "journey."

📉 5. Reduce Booking Cancellations
- Observation: High number of cancellations
- Recommendation: Investigate reasons for cancellations—could be price sensitivity, flexible policies, or poor user experience.
- Action: Introduce non-refundable discounts, improve booking terms clarity, and strengthen pre-arrival communication.

🚫 6. Address ‘No Shows’
- Recommendation: Implement partial prepayments or no-show penalties.
- Action: Use automated reminders (SMS/email) before check-in.

📆 7. Strengthen Low-Performing Months
- Observation: July shows declining revenue
- Recommendation: Run promotional campaigns or special events to boost July bookings.
- Action: Offer early bird discounts or partner with local events to drive demand.

🏨 8. Boost Performance of Underperforming Properties
- Example: Atliq Seasons
- Recommendation: Reevaluate pricing, amenities, and local competition.
- Action: Test promotions or reposition the property for a different segment (e.g., budget-friendly or boutique).





