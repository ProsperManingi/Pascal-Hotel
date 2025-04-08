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

