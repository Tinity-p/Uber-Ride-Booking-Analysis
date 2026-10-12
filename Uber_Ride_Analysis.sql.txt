-- =====================================================
-- Uber Ride Booking Analysis
-- SQL Analysis Project
-- Database: uber_analysis
-- Table: bookings
-- =====================================================

USE uber_analysis;

-- =====================================================
-- 1. Booking Overview
-- =====================================================

-- Total number of bookings
SELECT
    COUNT(*) AS total_bookings
FROM bookings;

-- Unique Booking IDs
SELECT
    COUNT(DISTINCT `Booking ID`) AS unique_booking_ids
FROM bookings;

-- Unique Customer IDs
SELECT
    COUNT(DISTINCT `Customer ID`) AS unique_customer_ids
FROM bookings;



-- =====================================================
-- 2. Booking Status Analysis
-- =====================================================

-- Booking status breakdown
SELECT
    `Booking Status`,
    COUNT(*) AS total_bookings
FROM bookings
GROUP BY `Booking Status`
ORDER BY total_bookings DESC;


-- Overall completion rate
SELECT
    COUNT(*) AS total_bookings,

    SUM(
        CASE
            WHEN `Booking Status` = 'Completed' THEN 1
            ELSE 0
        END
    ) AS completed_bookings,

    ROUND(
        100.0 * SUM(
            CASE
                WHEN `Booking Status` = 'Completed' THEN 1
                ELSE 0
            END
        ) / COUNT(*),
        2
    ) AS completion_rate
FROM bookings;


-- Non-completed booking breakdown
SELECT
    `Booking Status`,
    COUNT(*) AS total_bookings
FROM bookings
WHERE `Booking Status` <> 'Completed'
GROUP BY `Booking Status`
ORDER BY total_bookings DESC;



-- =====================================================
-- 3. Cancellation Analysis
-- =====================================================

-- Customer cancellation reasons
SELECT
    `Reason for cancelling by Customer`,
    COUNT(*) AS total_cancellations
FROM bookings
WHERE `Booking Status` = 'Cancelled by Customer'
GROUP BY `Reason for cancelling by Customer`
ORDER BY total_cancellations DESC;


-- Driver cancellation reasons
SELECT
    `Driver Cancellation Reason`,
    COUNT(*) AS total_cancellations
FROM bookings
WHERE `Booking Status` = 'Cancelled by Driver'
GROUP BY `Driver Cancellation Reason`
ORDER BY total_cancellations DESC;


-- Incomplete ride reasons
SELECT
    `Incomplete Rides Reason`,
    COUNT(*) AS total_incomplete
FROM bookings
WHERE `Booking Status` = 'Incomplete'
GROUP BY `Incomplete Rides Reason`
ORDER BY total_incomplete DESC;



-- =====================================================
-- 4. Booking Value Analysis
-- =====================================================

-- Total Booking Value from completed rides
SELECT
    SUM(
        CAST(`Booking Value` AS DECIMAL(10,2))
    ) AS total_booking_value
FROM bookings
WHERE `Booking Status` = 'Completed';


-- Average Booking Value per completed ride
SELECT
    ROUND(
        AVG(
            CAST(`Booking Value` AS DECIMAL(10,2))
        ),
        2
    ) AS average_booking_value
FROM bookings
WHERE `Booking Status` = 'Completed';


-- Booking Value by vehicle type
SELECT
    `Vehicle Type`,
    SUM(
        CAST(`Booking Value` AS DECIMAL(10,2))
    ) AS total_booking_value
FROM bookings
WHERE `Booking Status` = 'Completed'
GROUP BY `Vehicle Type`
ORDER BY total_booking_value DESC;


-- Average Booking Value by vehicle type
SELECT
    `Vehicle Type`,
    COUNT(*) AS completed_rides,
    ROUND(
        AVG(
            CAST(`Booking Value` AS DECIMAL(10,2))
        ),
        2
    ) AS average_booking_value
FROM bookings
WHERE `Booking Status` = 'Completed'
GROUP BY `Vehicle Type`
ORDER BY average_booking_value DESC;




-- =====================================================
-- 5. Vehicle Type Analysis
-- =====================================================

-- Completed rides by vehicle type
SELECT
    `Vehicle Type`,
    COUNT(*) AS completed_rides
FROM bookings
WHERE `Booking Status` = 'Completed'
GROUP BY `Vehicle Type`
ORDER BY completed_rides DESC;


-- Booking status by vehicle type
SELECT
    `Vehicle Type`,
    `Booking Status`,
    COUNT(*) AS total_bookings
FROM bookings
GROUP BY
    `Vehicle Type`,
    `Booking Status`
ORDER BY
    `Vehicle Type`,
    total_bookings DESC;


-- Completion rate by vehicle type
SELECT
    `Vehicle Type`,
    COUNT(*) AS total_bookings,

    SUM(
        CASE
            WHEN `Booking Status` = 'Completed' THEN 1
            ELSE 0
        END
    ) AS completed_rides,

    ROUND(
        100.0 * SUM(
            CASE
                WHEN `Booking Status` = 'Completed' THEN 1
                ELSE 0
            END
        ) / COUNT(*),
        2
    ) AS completion_rate
FROM bookings
GROUP BY `Vehicle Type`
ORDER BY completion_rate DESC;




-- =====================================================
-- 6. Location Analysis
-- =====================================================

-- Top 10 pickup locations
SELECT
    `Pickup Location`,
    COUNT(*) AS total_bookings
FROM bookings
GROUP BY `Pickup Location`
ORDER BY total_bookings DESC
LIMIT 10;


-- Top 10 drop locations
SELECT
    `Drop Location`,
    COUNT(*) AS total_bookings
FROM bookings
GROUP BY `Drop Location`
ORDER BY total_bookings DESC
LIMIT 10;




-- =====================================================
-- 7. Time Analysis
-- =====================================================

-- Bookings by hour
SELECT
    HOUR(
        STR_TO_DATE(`Time`, '%H:%i:%s')
    ) AS booking_hour,
    COUNT(*) AS total_bookings
FROM bookings
GROUP BY booking_hour
ORDER BY total_bookings DESC;


-- Monthly booking trend
SELECT
    DATE_FORMAT(
        STR_TO_DATE(`Date`, '%Y-%m-%d'),
        '%Y-%m'
    ) AS booking_month,
    COUNT(*) AS total_bookings
FROM bookings
GROUP BY booking_month
ORDER BY booking_month;


-- Monthly completed rides and total Booking Value
SELECT
    DATE_FORMAT(
        STR_TO_DATE(`Date`, '%Y-%m-%d'),
        '%Y-%m'
    ) AS booking_month,

    COUNT(*) AS completed_rides,

    ROUND(
        SUM(
            CAST(
                NULLIF(TRIM(`Booking Value`), '')
                AS DECIMAL(10,2)
            )
        ),
        2
    ) AS total_booking_value
FROM bookings
WHERE `Booking Status` = 'Completed'
GROUP BY booking_month
ORDER BY booking_month;


-- Monthly average Booking Value
SELECT
    DATE_FORMAT(
        STR_TO_DATE(`Date`, '%Y-%m-%d'),
        '%Y-%m'
    ) AS booking_month,

    COUNT(*) AS completed_rides,

    ROUND(
        AVG(
            CAST(
                NULLIF(TRIM(`Booking Value`), '')
                AS DECIMAL(10,2)
            )
        ),
        2
    ) AS average_booking_value
FROM bookings
WHERE `Booking Status` = 'Completed'
GROUP BY booking_month
ORDER BY booking_month;





-- =====================================================
-- 8. Distance & Rating Analysis
-- =====================================================

-- Average and maximum ride distance for completed rides
SELECT
    ROUND(
        AVG(
            CAST(
                NULLIF(TRIM(`Ride Distance`), '')
                AS DECIMAL(10,2)
            )
        ),
        2
    ) AS avg_ride_distance,

    MAX(
        CAST(
            NULLIF(TRIM(`Ride Distance`), '')
            AS DECIMAL(10,2)
        )
    ) AS max_ride_distance
FROM bookings
WHERE `Booking Status` = 'Completed';


-- Average driver and customer ratings
SELECT
    ROUND(
        AVG(
            CAST(
                NULLIF(TRIM(`Driver Ratings`), '')
                AS DECIMAL(10,2)
            )
        ),
        2
    ) AS avg_driver_rating,

    ROUND(
        AVG(
            CAST(
                NULLIF(TRIM(`Customer Rating`), '')
                AS DECIMAL(10,2)
            )
        ),
        2
    ) AS avg_customer_rating
FROM bookings
WHERE `Booking Status` = 'Completed';





-- =====================================================
-- 9. Payment Method Analysis
-- =====================================================

-- Payment methods across all bookings
SELECT
    `Payment Method`,
    COUNT(*) AS total_bookings
FROM bookings
GROUP BY `Payment Method`
ORDER BY total_bookings DESC;


-- Payment methods for completed rides
SELECT
    `Payment Method`,
    COUNT(*) AS completed_rides
FROM bookings
WHERE `Booking Status` = 'Completed'
GROUP BY `Payment Method`
ORDER BY completed_rides DESC;




-- =====================================================
-- 10. KPI Summary
-- =====================================================

SELECT
    COUNT(*) AS total_bookings,

    SUM(
        CASE
            WHEN `Booking Status` = 'Completed' THEN 1
            ELSE 0
        END
    ) AS completed_rides,

    SUM(
        CASE
            WHEN `Booking Status` <> 'Completed' THEN 1
            ELSE 0
        END
    ) AS non_completed_rides,

    ROUND(
        100.0 * SUM(
            CASE
                WHEN `Booking Status` = 'Completed' THEN 1
                ELSE 0
            END
        ) / COUNT(*),
        2
    ) AS completion_rate,

    ROUND(
        SUM(
            CASE
                WHEN `Booking Status` = 'Completed'
                THEN CAST(NULLIF(TRIM(`Booking Value`), '') AS DECIMAL(10,2))
                ELSE 0
            END
        ),
        2
    ) AS total_booking_value,

    ROUND(
        AVG(
            CASE
                WHEN `Booking Status` = 'Completed'
                THEN CAST(NULLIF(TRIM(`Booking Value`), '') AS DECIMAL(10,2))
            END
        ),
        2
    ) AS average_booking_value

FROM bookings;



