# MySQL Date Datatypes and Functions - Complete Guide

## Table of Contents
1. [Date Datatypes](#date-datatypes)
2. [MySQL Date Functions](#mysql-date-functions)
3. [Practical Examples](#practical-examples)
4. [Best Practices](#best-practices)

---

## Date Datatypes

MySQL provides several date and time datatypes to store temporal data. Each has specific characteristics, storage requirements, and use cases.

### 1. DATE

**Description**: Stores date values (year, month, day) without time information.

**Format**: `YYYY-MM-DD`
**Range**: `1000-01-01` to `9999-12-31`
**Storage**: 3 bytes

**Examples**:
```sql
CREATE TABLE events (
    id INT PRIMARY KEY,
    event_date DATE
);

INSERT INTO events VALUES 
(1, '2024-12-25'),
(2, '2024-01-01'),
(3, '2024-07-04');

-- Query examples
SELECT * FROM events WHERE event_date = '2024-12-25';
SELECT * FROM events WHERE event_date BETWEEN '2024-01-01' AND '2024-12-31';
```

### 2. TIME

**Description**: Stores time values (hours, minutes, seconds) without date information.

**Format**: `HH:MM:SS` or `HHH:MM:SS` (for times > 24 hours)
**Range**: `-838:59:59` to `838:59:59`
**Storage**: 3 bytes

**Examples**:
```sql
CREATE TABLE schedules (
    id INT PRIMARY KEY,
    start_time TIME,
    duration TIME
);

INSERT INTO schedules VALUES 
(1, '09:30:00', '02:15:00'),
(2, '14:45:30', '01:30:45'),
(3, '23:59:59', '00:01:00');

-- Query examples
SELECT * FROM schedules WHERE start_time > '12:00:00';
SELECT *, ADDTIME(start_time, duration) AS end_time FROM schedules;
```

### 3. DATETIME

**Description**: Stores both date and time information.

**Format**: `YYYY-MM-DD HH:MM:SS`
**Range**: `1000-01-01 00:00:00` to `9999-12-31 23:59:59`
**Storage**: 8 bytes

**Examples**:
```sql
CREATE TABLE appointments (
    id INT PRIMARY KEY,
    appointment_datetime DATETIME
);

INSERT INTO appointments VALUES 
(1, '2024-12-25 10:30:00'),
(2, '2024-01-15 14:45:30'),
(3, '2024-06-30 09:00:00');

-- Query examples
SELECT * FROM appointments WHERE appointment_datetime > '2024-06-01 00:00:00';
SELECT DATE(appointment_datetime) AS appointment_date FROM appointments;
```

### 4. TIMESTAMP

**Description**: Stores date and time with automatic timezone conversion and automatic updates.

**Format**: `YYYY-MM-DD HH:MM:SS`
**Range**: `1970-01-01 00:00:01` UTC to `2038-01-19 03:14:07` UTC
**Storage**: 4 bytes

**Key Features**:
- Automatically converts from current timezone to UTC for storage
- Converts back to current timezone for retrieval
- Can auto-initialize and auto-update

**Examples**:
```sql
CREATE TABLE user_activity (
    id INT PRIMARY KEY,
    user_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

INSERT INTO user_activity (id, user_id) VALUES (1, 101);

-- The timestamps are automatically set
SELECT * FROM user_activity;
```

### 5. YEAR

**Description**: Stores year values only.

**Format**: `YYYY`
**Range**: `1901` to `2155` (4-digit format)
**Storage**: 1 byte

**Examples**:
```sql
CREATE TABLE cars (
    id INT PRIMARY KEY,
    model VARCHAR(50),
    manufacture_year YEAR
);

INSERT INTO cars VALUES 
(1, 'Toyota Camry', 2023),
(2, 'Honda Civic', 2022),
(3, 'Ford Focus', 2021);

-- Query examples
SELECT * FROM cars WHERE manufacture_year = 2023;
SELECT * FROM cars WHERE manufacture_year BETWEEN 2020 AND 2023;
```

---

## MySQL Date Functions

MySQL provides numerous built-in functions for working with date and time values.

### Current Date/Time Functions

#### NOW() / CURRENT_TIMESTAMP / CURRENT_TIMESTAMP()
Returns current date and time.

```sql
SELECT NOW();                    -- 2024-05-26 14:30:25
SELECT CURRENT_TIMESTAMP;        -- 2024-05-26 14:30:25
SELECT CURRENT_TIMESTAMP();      -- 2024-05-26 14:30:25
```

#### CURDATE() / CURRENT_DATE / CURRENT_DATE()
Returns current date.

```sql
SELECT CURDATE();               -- 2024-05-26
SELECT CURRENT_DATE;            -- 2024-05-26
SELECT CURRENT_DATE();          -- 2024-05-26
```

#### CURTIME() / CURRENT_TIME / CURRENT_TIME()
Returns current time.

```sql
SELECT CURTIME();               -- 14:30:25
SELECT CURRENT_TIME;            -- 14:30:25
SELECT CURRENT_TIME();          -- 14:30:25
```

### Date Extraction Functions

#### YEAR(), MONTH(), DAY()
Extract specific parts from date values.

```sql
SELECT YEAR('2024-05-26');      -- 2024
SELECT MONTH('2024-05-26');     -- 5
SELECT DAY('2024-05-26');       -- 26

-- Practical example
SELECT 
    appointment_datetime,
    YEAR(appointment_datetime) AS year,
    MONTH(appointment_datetime) AS month,
    DAY(appointment_datetime) AS day
FROM appointments;
```

#### HOUR(), MINUTE(), SECOND()
Extract time components.

```sql
SELECT HOUR('14:30:25');        -- 14
SELECT MINUTE('14:30:25');      -- 30
SELECT SECOND('14:30:25');      -- 25

-- With datetime
SELECT 
    appointment_datetime,
    HOUR(appointment_datetime) AS hour,
    MINUTE(appointment_datetime) AS minute
FROM appointments;
```

#### DAYNAME(), MONTHNAME()
Get names of days and months.

```sql
SELECT DAYNAME('2024-05-26');   -- Sunday
SELECT MONTHNAME('2024-05-26'); -- May

SELECT 
    appointment_datetime,
    DAYNAME(appointment_datetime) AS day_name,
    MONTHNAME(appointment_datetime) AS month_name
FROM appointments;
```

#### DAYOFWEEK(), DAYOFMONTH(), DAYOFYEAR()
Get numeric day representations.

```sql
SELECT DAYOFWEEK('2024-05-26'); -- 1 (Sunday=1, Monday=2, etc.)
SELECT DAYOFMONTH('2024-05-26');-- 26
SELECT DAYOFYEAR('2024-05-26'); -- 147
```

#### WEEK(), WEEKOFYEAR()
Get week numbers.

```sql
SELECT WEEK('2024-05-26');      -- 21
SELECT WEEKOFYEAR('2024-05-26');-- 21
```

#### QUARTER()
Get quarter of the year.

```sql
SELECT QUARTER('2024-05-26');   -- 2
SELECT QUARTER('2024-11-15');   -- 4
```

### Date Formatting Functions

#### DATE_FORMAT()
Format dates according to specified format strings.

```sql
-- Common format specifiers:
-- %Y: 4-digit year    %y: 2-digit year
-- %M: Month name      %m: Month number (01-12)
-- %D: Day with suffix %d: Day number (01-31)
-- %H: Hour (00-23)    %h: Hour (01-12)
-- %i: Minutes         %s: Seconds
-- %W: Weekday name    %w: Weekday (0=Sunday)

SELECT DATE_FORMAT('2024-05-26 14:30:25', '%W, %M %D, %Y');
-- Result: Sunday, May 26th, 2024

SELECT DATE_FORMAT('2024-05-26 14:30:25', '%m/%d/%Y %h:%i %p');
-- Result: 05/26/2024 02:30 PM

SELECT DATE_FORMAT('2024-05-26', '%Y-%m');
-- Result: 2024-05

-- Practical example
SELECT 
    appointment_datetime,
    DATE_FORMAT(appointment_datetime, '%W, %M %D at %h:%i %p') AS formatted_appointment
FROM appointments;
```

#### TIME_FORMAT()
Format time values.

```sql
SELECT TIME_FORMAT('14:30:25', '%h:%i %p');  -- 02:30 PM
SELECT TIME_FORMAT('14:30:25', '%H:%i');     -- 14:30
```

### Date Arithmetic Functions

#### DATE_ADD() / ADDDATE()
Add intervals to dates.

```sql
SELECT DATE_ADD('2024-05-26', INTERVAL 1 DAY);     -- 2024-05-27
SELECT DATE_ADD('2024-05-26', INTERVAL 1 WEEK);    -- 2024-06-02
SELECT DATE_ADD('2024-05-26', INTERVAL 1 MONTH);   -- 2024-06-26
SELECT DATE_ADD('2024-05-26', INTERVAL 1 YEAR);    -- 2025-05-26
SELECT DATE_ADD('2024-05-26', INTERVAL -1 DAY);    -- 2024-05-25

-- With time
SELECT DATE_ADD('2024-05-26 14:30:25', INTERVAL 2 HOUR);
-- Result: 2024-05-26 16:30:25

-- Practical example: Find appointments in next 7 days
SELECT * FROM appointments 
WHERE appointment_datetime BETWEEN NOW() 
AND DATE_ADD(NOW(), INTERVAL 7 DAY);
```

#### DATE_SUB() / SUBDATE()
Subtract intervals from dates.

```sql
SELECT DATE_SUB('2024-05-26', INTERVAL 1 DAY);     -- 2024-05-25
SELECT DATE_SUB('2024-05-26', INTERVAL 1 MONTH);   -- 2024-04-26
SELECT DATE_SUB('2024-05-26', INTERVAL 1 YEAR);    -- 2023-05-26
```

#### ADDTIME() / SUBTIME()
Add or subtract time from datetime/time values.

```sql
SELECT ADDTIME('2024-05-26 14:30:25', '02:15:30');
-- Result: 2024-05-26 16:45:55

SELECT SUBTIME('2024-05-26 14:30:25', '01:15:30');
-- Result: 2024-05-26 13:14:55

SELECT ADDTIME('10:30:00', '02:15:30');  -- 12:45:30
```

### Date Difference Functions

#### DATEDIFF()
Calculate difference in days between two dates.

```sql
SELECT DATEDIFF('2024-05-26', '2024-05-20');  -- 6
SELECT DATEDIFF('2024-05-20', '2024-05-26');  -- -6

-- Practical example: Calculate age in days
SELECT 
    user_id,
    birth_date,
    DATEDIFF(CURDATE(), birth_date) AS age_in_days
FROM users;
```

#### TIMEDIFF()
Calculate time difference between two time/datetime values.

```sql
SELECT TIMEDIFF('14:30:25', '10:15:20');  -- 04:15:05
SELECT TIMEDIFF('2024-05-26 14:30:25', '2024-05-26 10:15:20');
-- Result: 04:15:05
```

#### TIMESTAMPDIFF()
Calculate difference between two timestamps in specified units.

```sql
SELECT TIMESTAMPDIFF(YEAR, '1990-05-26', '2024-05-26');    -- 34
SELECT TIMESTAMPDIFF(MONTH, '2024-01-15', '2024-05-26');   -- 4
SELECT TIMESTAMPDIFF(DAY, '2024-05-20', '2024-05-26');     -- 6
SELECT TIMESTAMPDIFF(HOUR, '2024-05-26 10:00:00', '2024-05-26 14:30:00'); -- 4

-- Practical example: Calculate employee tenure
SELECT 
    employee_id,
    hire_date,
    TIMESTAMPDIFF(YEAR, hire_date, CURDATE()) AS years_employed,
    TIMESTAMPDIFF(MONTH, hire_date, CURDATE()) AS months_employed
FROM employees;
```

### Date Conversion Functions

#### STR_TO_DATE()
Convert string to date using format specifiers.

```sql
SELECT STR_TO_DATE('26-05-2024', '%d-%m-%Y');     -- 2024-05-26
SELECT STR_TO_DATE('May 26, 2024', '%M %d, %Y');  -- 2024-05-26
SELECT STR_TO_DATE('2024/05/26 14:30', '%Y/%m/%d %H:%i');
-- Result: 2024-05-26 14:30:00

-- Practical example: Convert text dates to proper DATE format
UPDATE events 
SET event_date = STR_TO_DATE(event_date_text, '%m/%d/%Y')
WHERE event_date_text IS NOT NULL;
```

#### DATE() / TIME()
Extract date or time part from datetime values.

```sql
SELECT DATE('2024-05-26 14:30:25');  -- 2024-05-26
SELECT TIME('2024-05-26 14:30:25');  -- 14:30:25

-- Practical example: Group by date ignoring time
SELECT 
    DATE(appointment_datetime) AS appointment_date,
    COUNT(*) AS appointments_count
FROM appointments
GROUP BY DATE(appointment_datetime);
```

#### UNIX_TIMESTAMP() / FROM_UNIXTIME()
Convert between MySQL datetime and Unix timestamp.

```sql
-- Convert datetime to Unix timestamp
SELECT UNIX_TIMESTAMP('2024-05-26 14:30:25');  -- 1716734225

-- Convert Unix timestamp to datetime
SELECT FROM_UNIXTIME(1716734225);  -- 2024-05-26 14:30:25

-- Current Unix timestamp
SELECT UNIX_TIMESTAMP();  -- Current timestamp
```

### Date Validation Functions

#### LAST_DAY()
Get the last day of the month for a given date.

```sql
SELECT LAST_DAY('2024-02-15');  -- 2024-02-29 (leap year)
SELECT LAST_DAY('2024-04-15');  -- 2024-04-30

-- Practical example: Find month-end dates
SELECT 
    employee_id,
    salary_date,
    LAST_DAY(salary_date) AS month_end
FROM salaries;
```

#### MAKEDATE()
Create date from year and day of year.

```sql
SELECT MAKEDATE(2024, 147);  -- 2024-05-26 (147th day of 2024)
SELECT MAKEDATE(2024, 1);    -- 2024-01-01
SELECT MAKEDATE(2024, 366);  -- 2024-12-31 (leap year)
```

#### MAKETIME()
Create time from hour, minute, second.

```sql
SELECT MAKETIME(14, 30, 25);  -- 14:30:25
SELECT MAKETIME(2, 15, 0);    -- 02:15:00
```

### Timezone Functions

#### CONVERT_TZ()
Convert datetime from one timezone to another.

```sql
-- Convert from UTC to EST
SELECT CONVERT_TZ('2024-05-26 14:30:25', '+00:00', '-05:00');
-- Result: 2024-05-26 09:30:25

-- Convert from PST to UTC
SELECT CONVERT_TZ('2024-05-26 14:30:25', '-08:00', '+00:00');
-- Result: 2024-05-26 22:30:25
```

---

## Practical Examples

### 1. Age Calculation
```sql
-- Calculate age in years
SELECT 
    user_id,
    birth_date,
    TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) AS age_years,
    FLOOR(DATEDIFF(CURDATE(), birth_date) / 365.25) AS age_years_precise
FROM users;
```

### 2. Business Days Calculation
```sql
-- Find records from last 30 business days (excluding weekends)
SELECT * FROM orders 
WHERE order_date >= DATE_SUB(CURDATE(), INTERVAL 30 DAY)
AND DAYOFWEEK(order_date) NOT IN (1, 7);  -- Exclude Sunday(1) and Saturday(7)
```

### 3. Monthly Reports
```sql
-- Monthly sales summary
SELECT 
    YEAR(order_date) AS year,
    MONTH(order_date) AS month,
    MONTHNAME(order_date) AS month_name,
    COUNT(*) AS total_orders,
    SUM(order_amount) AS total_sales
FROM orders
GROUP BY YEAR(order_date), MONTH(order_date)
ORDER BY year, month;
```

### 4. Date Range Queries
```sql
-- Find records in current month
SELECT * FROM appointments 
WHERE YEAR(appointment_datetime) = YEAR(CURDATE())
AND MONTH(appointment_datetime) = MONTH(CURDATE());

-- Find records in last quarter
SELECT * FROM sales 
WHERE sale_date >= DATE_SUB(CURDATE(), INTERVAL 3 MONTH);
```

### 5. Working with Schedules
```sql
-- Find overlapping appointments
SELECT a1.*, a2.*
FROM appointments a1
JOIN appointments a2 ON a1.id < a2.id
WHERE a1.appointment_datetime <= DATE_ADD(a2.appointment_datetime, INTERVAL 1 HOUR)
AND a2.appointment_datetime <= DATE_ADD(a1.appointment_datetime, INTERVAL 1 HOUR);
```

### 6. Date Formatting for Reports
```sql
-- User-friendly date formatting
SELECT 
    order_id,
    DATE_FORMAT(order_date, '%W, %M %D, %Y') AS formatted_date,
    DATE_FORMAT(order_date, '%m/%d/%Y') AS short_date,
    CASE 
        WHEN DATEDIFF(CURDATE(), order_date) = 0 THEN 'Today'
        WHEN DATEDIFF(CURDATE(), order_date) = 1 THEN 'Yesterday'
        WHEN DATEDIFF(CURDATE(), order_date) <= 7 THEN CONCAT(DATEDIFF(CURDATE(), order_date), ' days ago')
        ELSE DATE_FORMAT(order_date, '%M %d, %Y')
    END AS relative_date
FROM orders;
```

---

## Best Practices

### 1. Datatype Selection
- Use **DATE** for date-only values (birthdays, event dates)
- Use **TIME** for time-only values (daily schedules, durations)
- Use **DATETIME** for specific moments in time without timezone concerns
- Use **TIMESTAMP** for audit trails and when timezone conversion is needed
- Use **YEAR** for year-only data (manufacture years, graduation years)

### 2. Indexing Date Columns
```sql
-- Create indexes on frequently queried date columns
CREATE INDEX idx_order_date ON orders(order_date);
CREATE INDEX idx_created_at ON user_activity(created_at);

-- For range queries, consider composite indexes
CREATE INDEX idx_user_date ON user_activity(user_id, created_at);
```

### 3. Handling NULL Values
```sql
-- Use COALESCE for default dates
SELECT COALESCE(last_login, '1970-01-01') AS last_login_safe FROM users;

-- Use IFNULL for calculations
SELECT IFNULL(DATEDIFF(CURDATE(), last_login), 999) AS days_since_login FROM users;
```

### 4. Date Validation
```sql
-- Validate date ranges in applications
SELECT * FROM events 
WHERE event_date BETWEEN '2024-01-01' AND '2024-12-31'
AND event_date IS NOT NULL;

-- Check for valid dates when inserting
INSERT INTO events (event_date) 
SELECT '2024-02-29' 
WHERE STR_TO_DATE('2024-02-29', '%Y-%m-%d') IS NOT NULL;
```

### 5. Performance Considerations
- Use date functions in SELECT clause rather than WHERE clause when possible
- Avoid using functions on indexed columns in WHERE clauses
- Use BETWEEN for date ranges instead of multiple comparisons
- Consider partitioning large tables by date ranges

### 6. Timezone Handling
```sql
-- Always be explicit about timezones
SET time_zone = '+00:00';  -- Set to UTC

-- Store timestamps in UTC, convert for display
SELECT 
    event_id,
    CONVERT_TZ(event_timestamp, '+00:00', '-05:00') AS local_time
FROM events;
```

### 7. Common Pitfalls to Avoid
- Don't use strings for date storage
- Don't ignore timezone differences
- Don't forget leap years in calculations
- Don't use deprecated 2-digit year formats
- Be careful with month/day ordering in different locales

---

This guide covers the essential date datatypes and functions in MySQL. Practice with these examples and adapt them to your specific use cases for effective date and time handling in your database applications.