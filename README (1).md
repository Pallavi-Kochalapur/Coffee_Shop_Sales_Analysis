# ☕ Coffee Shop Sales — Power BI + MySQL Portfolio Project

An end-to-end data analytics project that takes raw coffee shop transaction data through **MySQL** (cleaning, transformation, business-question queries) and into a fully interactive **Power BI** dashboard.

Coffee Shop Sales Dashboard[Coffee_Shop_Sales.pbix]
<img width="1207" height="675" alt="image" src="https://github.com/user-attachments/assets/2f686d0d-0581-48c2-949b-7359007d5608" />

---

## 📌 Project Overview

This project simulates a real-world data analyst workflow for a coffee shop chain with three store locations (Hell's Kitchen, Astoria, Lower Manhattan). Raw transactional data is cleaned and queried in MySQL to answer core business questions, then modeled and visualized in Power BI to deliver an interactive, month-sliced sales report.

**Workflow:**

```
Raw CSV Data → MySQL (clean, type-cast, query) → Power BI (data model, DAX, dashboard)
```

---

## 🧰 Tech Stack

| Tool | Purpose |
|---|---|
| **MySQL / MySQL Workbench** | Data cleaning, type conversion, and business-question querying |
| **Power BI Desktop** | Data modeling, DAX measures, and interactive dashboard design |

---

## 🗂️ MySQL — Steps Followed

1. Data walkthrough
2. Raw data file preparation
3. Creating the database
4. Importing the file
5. Cleaning the imported file
6. Changing data types
7. Firing SQL queries for business requirements
8. Storing results
9. Preparing SQL documentation

**SQL concepts/functions used:**

`STR_TO_DATE` · `ROUND` · `SUM` · `COUNT` · `AVG` · `LAG` (window function) · `MONTH` · `DAY` · `DAYOFWEEK` · `HOUR` · `SELECT` / `ALIAS` · `MAX` / `MIN` · `ALTER TABLE` · `UPDATE` · `CHANGE COLUMN` · `WHERE` · `GROUP BY` · `CASE` · `ORDER BY` · `LIMIT` · Window Functions · Joins · Subqueries

---

## 🎯 Problem Statement

### KPI Requirements

**1. Total Sales Analysis**
- Calculate total sales for each respective month
- Determine month-on-month (MoM) increase/decrease in sales
- Calculate the difference in sales between the selected month and the previous month

**2. Total Orders Analysis**
- Calculate total number of orders for each respective month
- Determine MoM increase/decrease in the number of orders
- Calculate the difference in orders between the selected month and the previous month

**3. Total Quantity Sold Analysis**
- Calculate total quantity sold for each respective month
- Determine MoM increase/decrease in quantity sold
- Calculate the difference in quantity sold between the selected month and the previous month

### Chart / Visual Requirements

**1. Calendar Heat Map**
- Dynamically adjusts based on the month selected in the slicer
- Each day is color-coded by sales volume (darker = higher sales)
- Tooltips show Sales, Orders, and Quantity on hover

**2. Sales by Weekday vs. Weekend**
- Segments sales into weekdays vs. weekends
- Surfaces whether performance differs meaningfully between the two

**3. Sales by Store Location**
- Visualizes sales across all store locations
- Includes MoM difference metrics per store, tied to the slicer

**4. Daily Sales with Average Line**
- Line/bar chart of daily sales for the selected month
- Average daily sales line overlaid
- Bars above/below average are highlighted

**5. Sales by Product Category**
- Compares performance across product categories
- Surfaces which categories drive the most revenue

**6. Top 10 Products by Sales**
- Ranks and displays the top 10 products by sales volume

**7. Sales by Day & Hour**
- Heat map of sales across day-of-week and hour-of-day
- Tooltips show Sales, Orders, and Quantity for each day-hour cell

---

## 🗄️ SQL Queries

All queries were run against a `coffee_shop_sales` table in MySQL. Full explanations are included inline as comments.

### 1. Data Cleaning & Type Conversion

```sql
-- Convert transaction_date to a proper date format
UPDATE coffee_shop_sales
SET transaction_date = STR_TO_DATE(transaction_date, '%d-%m-%Y');

-- Alter transaction_date column to DATE data type
ALTER TABLE coffee_shop_sales
MODIFY COLUMN transaction_date DATE;

-- Convert transaction_time to a proper time format
UPDATE coffee_shop_sales
SET transaction_time = STR_TO_DATE(transaction_time, '%H:%i:%s');

-- Alter transaction_time column to TIME data type
ALTER TABLE coffee_shop_sales
MODIFY COLUMN transaction_time TIME;

-- Inspect column data types
DESCRIBE coffee_shop_sales;

-- Fix the malformed transaction_id column name (BOM artifact)
ALTER TABLE coffee_shop_sales
CHANGE COLUMN `ï»¿transaction_id` transaction_id INT;
```

### 2. Total Sales (+ MoM Growth)

```sql
SELECT ROUND(SUM(unit_price * transaction_qty)) as Total_Sales
FROM coffee_shop_sales
WHERE MONTH(transaction_date) = 5; -- for month of (CM-May)

SELECT
    MONTH(transaction_date) AS month,
    ROUND(SUM(unit_price * transaction_qty)) AS total_sales,
    (SUM(unit_price * transaction_qty) - LAG(SUM(unit_price * transaction_qty), 1)
        OVER (ORDER BY MONTH(transaction_date))) / LAG(SUM(unit_price * transaction_qty), 1)
        OVER (ORDER BY MONTH(transaction_date)) * 100 AS mom_increase_percentage
FROM
    coffee_shop_sales
WHERE
    MONTH(transaction_date) IN (4, 5) -- for months of April and May
GROUP BY
    MONTH(transaction_date)
ORDER BY
    MONTH(transaction_date);
```

### 3. Total Orders (+ MoM Growth)

```sql
SELECT COUNT(transaction_id) as Total_Orders
FROM coffee_shop_sales
WHERE MONTH(transaction_date) = 5; -- for month of (CM-May)

SELECT
    MONTH(transaction_date) AS month,
    ROUND(COUNT(transaction_id)) AS total_orders,
    (COUNT(transaction_id) - LAG(COUNT(transaction_id), 1)
        OVER (ORDER BY MONTH(transaction_date))) / LAG(COUNT(transaction_id), 1)
        OVER (ORDER BY MONTH(transaction_date)) * 100 AS mom_increase_percentage
FROM
    coffee_shop_sales
WHERE
    MONTH(transaction_date) IN (4, 5) -- for April and May
GROUP BY
    MONTH(transaction_date)
ORDER BY
    MONTH(transaction_date);
```

### 4. Total Quantity Sold (+ MoM Growth)

```sql
SELECT SUM(transaction_qty) as Total_Quantity_Sold
FROM coffee_shop_sales
WHERE MONTH(transaction_date) = 5; -- for month of (CM-May)

SELECT
    MONTH(transaction_date) AS month,
    ROUND(SUM(transaction_qty)) AS total_quantity_sold,
    (SUM(transaction_qty) - LAG(SUM(transaction_qty), 1)
        OVER (ORDER BY MONTH(transaction_date))) / LAG(SUM(transaction_qty), 1)
        OVER (ORDER BY MONTH(transaction_date)) * 100 AS mom_increase_percentage
FROM
    coffee_shop_sales
WHERE
    MONTH(transaction_date) IN (4, 5) -- for April and May
GROUP BY
    MONTH(transaction_date)
ORDER BY
    MONTH(transaction_date);
```

### 5. Calendar Table — Daily Sales, Quantity & Orders

```sql
SELECT
    SUM(unit_price * transaction_qty) AS total_sales,
    SUM(transaction_qty) AS total_quantity_sold,
    COUNT(transaction_id) AS total_orders
FROM
    coffee_shop_sales
WHERE
    transaction_date = '2023-05-18'; -- For 18 May 2023

-- Same query, values rounded and displayed in "K" (thousands) format
SELECT
    CONCAT(ROUND(SUM(unit_price * transaction_qty) / 1000, 1),'K') AS total_sales,
    CONCAT(ROUND(COUNT(transaction_id) / 1000, 1),'K') AS total_orders,
    CONCAT(ROUND(SUM(transaction_qty) / 1000, 1),'K') AS total_quantity_sold
FROM
    coffee_shop_sales
WHERE
    transaction_date = '2023-05-18';
```

### 6. Sales Trend Over the Period (Average Daily Sales)

```sql
SELECT AVG(total_sales) AS average_sales
FROM (
    SELECT
        SUM(unit_price * transaction_qty) AS total_sales
    FROM
        coffee_shop_sales
    WHERE
        MONTH(transaction_date) = 5 -- Filter for May
    GROUP BY
        transaction_date
) AS internal_query;
```

### 7. Daily Sales for the Selected Month

```sql
SELECT
    DAY(transaction_date) AS day_of_month,
    ROUND(SUM(unit_price * transaction_qty),1) AS total_sales
FROM
    coffee_shop_sales
WHERE
    MONTH(transaction_date) = 5 -- Filter for May
GROUP BY
    DAY(transaction_date)
ORDER BY
    DAY(transaction_date);
```

### 8. Daily Sales vs. Average Sales (Above/Below Average Flag)

```sql
SELECT
    day_of_month,
    CASE
        WHEN total_sales > avg_sales THEN 'Above Average'
        WHEN total_sales < avg_sales THEN 'Below Average'
        ELSE 'Average'
    END AS sales_status,
    total_sales
FROM (
    SELECT
        DAY(transaction_date) AS day_of_month,
        SUM(unit_price * transaction_qty) AS total_sales,
        AVG(SUM(unit_price * transaction_qty)) OVER () AS avg_sales
    FROM
        coffee_shop_sales
    WHERE
        MONTH(transaction_date) = 5 -- Filter for May
    GROUP BY
        DAY(transaction_date)
) AS sales_data
ORDER BY
    day_of_month;
```

### 9. Sales by Weekday / Weekend

```sql
SELECT
    CASE
        WHEN DAYOFWEEK(transaction_date) IN (1, 7) THEN 'Weekends'
        ELSE 'Weekdays'
    END AS day_type,
    ROUND(SUM(unit_price * transaction_qty),2) AS total_sales
FROM
    coffee_shop_sales
WHERE
    MONTH(transaction_date) = 5 -- Filter for May
GROUP BY
    CASE
        WHEN DAYOFWEEK(transaction_date) IN (1, 7) THEN 'Weekends'
        ELSE 'Weekdays'
    END;
```

### 10. Sales by Store Location

```sql
SELECT
    store_location,
    SUM(unit_price * transaction_qty) as Total_Sales
FROM coffee_shop_sales
WHERE
    MONTH(transaction_date) = 5
GROUP BY store_location
ORDER BY SUM(unit_price * transaction_qty) DESC;
```

### 11. Sales by Product Category

```sql
SELECT
    product_category,
    ROUND(SUM(unit_price * transaction_qty),1) as Total_Sales
FROM coffee_shop_sales
WHERE
    MONTH(transaction_date) = 5
GROUP BY product_category
ORDER BY SUM(unit_price * transaction_qty) DESC;
```

### 12. Top 10 Products by Sales

```sql
SELECT
    product_type,
    ROUND(SUM(unit_price * transaction_qty),1) as Total_Sales
FROM coffee_shop_sales
WHERE
    MONTH(transaction_date) = 5
GROUP BY product_type
ORDER BY SUM(unit_price * transaction_qty) DESC
LIMIT 10;
```

### 13. Sales by Specific Day + Hour

```sql
SELECT
    ROUND(SUM(unit_price * transaction_qty)) AS Total_Sales,
    SUM(transaction_qty) AS Total_Quantity,
    COUNT(*) AS Total_Orders
FROM
    coffee_shop_sales
WHERE
    DAYOFWEEK(transaction_date) = 3 -- Tuesday (1 = Sunday ... 7 = Saturday)
    AND HOUR(transaction_time) = 8 -- 8 AM hour
    AND MONTH(transaction_date) = 5; -- May
```

### 14. Sales — Monday Through Sunday (May)

```sql
SELECT
    CASE
        WHEN DAYOFWEEK(transaction_date) = 2 THEN 'Monday'
        WHEN DAYOFWEEK(transaction_date) = 3 THEN 'Tuesday'
        WHEN DAYOFWEEK(transaction_date) = 4 THEN 'Wednesday'
        WHEN DAYOFWEEK(transaction_date) = 5 THEN 'Thursday'
        WHEN DAYOFWEEK(transaction_date) = 6 THEN 'Friday'
        WHEN DAYOFWEEK(transaction_date) = 7 THEN 'Saturday'
        ELSE 'Sunday'
    END AS Day_of_Week,
    ROUND(SUM(unit_price * transaction_qty)) AS Total_Sales
FROM
    coffee_shop_sales
WHERE
    MONTH(transaction_date) = 5 -- Filter for May
GROUP BY
    CASE
        WHEN DAYOFWEEK(transaction_date) = 2 THEN 'Monday'
        WHEN DAYOFWEEK(transaction_date) = 3 THEN 'Tuesday'
        WHEN DAYOFWEEK(transaction_date) = 4 THEN 'Wednesday'
        WHEN DAYOFWEEK(transaction_date) = 5 THEN 'Thursday'
        WHEN DAYOFWEEK(transaction_date) = 6 THEN 'Friday'
        WHEN DAYOFWEEK(transaction_date) = 7 THEN 'Saturday'
        ELSE 'Sunday'
    END;
```

### 15. Sales for All Hours (May)

```sql
SELECT
    HOUR(transaction_time) AS Hour_of_Day,
    ROUND(SUM(unit_price * transaction_qty)) AS Total_Sales
FROM
    coffee_shop_sales
WHERE
    MONTH(transaction_date) = 5 -- Filter for May
GROUP BY
    HOUR(transaction_time)
ORDER BY
    HOUR(transaction_time);
```

---

## 📊 Power BI — Data Model & DAX

The `Coffee_Shop_Sales.pbix` file's tabular model (`Transactions` fact table + `Date Table`) uses the following measures to power the dashboard. 

<img width="1205" height="671" alt="image" src="https://github.com/user-attachments/assets/12d17581-2745-4c31-82ce-9ec467369cf0" />

<img width="1205" height="675" alt="image" src="https://github.com/user-attachments/assets/e7b30cff-65d7-4b50-9cce-970298f268de" />

```dax
Total Sales =
SUM ( Transactions[unit_price] * Transactions[transaction_qty] )

Total Orders =
DISTINCTCOUNT ( Transactions[transaction_id] )

Total Quantity Sold =
SUM ( Transactions[transaction_qty] )

Daily Avg Sales =
AVERAGEX (
    VALUES ( Transactions[transaction_date] ),
    [Total Sales]
)

MoM Growth & Diff Sales =
VAR CurrentSales = [Total Sales]
VAR PreviousMonthSales =
    CALCULATE ( [Total Sales], DATEADD ( 'Date Table'[Date], -1, MONTH ) )
VAR Diff = CurrentSales - PreviousMonthSales
VAR PctGrowth = DIVIDE ( Diff, PreviousMonthSales )
RETURN
    Diff & " | " & FORMAT ( PctGrowth, "0.0%" )

MoM Growth & Diff Orders =
VAR CurrentOrders = [Total Orders]
VAR PreviousMonthOrders =
    CALCULATE ( [Total Orders], DATEADD ( 'Date Table'[Date], -1, MONTH ) )
VAR Diff = CurrentOrders - PreviousMonthOrders
VAR PctGrowth = DIVIDE ( Diff, PreviousMonthOrders )
RETURN
    Diff & " | " & FORMAT ( PctGrowth, "0.0%" )

MoM Growth & Diff Quantity =
VAR CurrentQty = [Total Quantity Sold]
VAR PreviousMonthQty =
    CALCULATE ( [Total Quantity Sold], DATEADD ( 'Date Table'[Date], -1, MONTH ) )
VAR Diff = CurrentQty - PreviousMonthQty
VAR PctGrowth = DIVIDE ( Diff, PreviousMonthQty )
RETURN
    Diff & " | " & FORMAT ( PctGrowth, "0.0%" )

Weekday / Weekend =
IF ( WEEKDAY ( 'Date Table'[Date], 2 ) >= 6, "Weekend", "Weekday" )

Colour For Bars =
IF ( [Total Sales] >= [Daily Avg Sales], "#E8A87C", "#4A2C2A" )
```

Supporting text/tooltip measures referenced in the report (used for dynamic titles, labels, and tooltips):
`Label for store location`, `Label for product category`, `Label for product type`, `New MoM Label`, `Foot Note`, `Placeholder`, `Hour`, `TT For Hour`.

**Date Table columns:** `Date`, `Month Year`, `Day Name`, `Day Number`, `Week Number`, `Weekday / Weekend`.

---

## 🖥️ Dashboard

Coffee Shop Sales Dashboard
<img width="1207" height="675" alt="image" src="https://github.com/user-attachments/assets/2f686d0d-0581-48c2-949b-7359007d5608" />

**Report elements:**
- Month slicer + calendar heat map (top-left) for date navigation
- KPI cards for Total Sales, Total Orders, Total Quantity Sold with MoM sparklines and % change vs. last month
- Daily sales trend chart with an average-sales reference line
- Weekday vs. Weekend revenue split (donut)
- Sales by store location
- Sales by product category
- Top 10 products by sales
- Day × Hour sales heat map

---

## 💡 Key Insights (May 2023)

- **Total Sales:** $157K — up **+31.8%** MoM ($37.8K increase vs. April)
- **Total Orders:** 33,527 — up **+32.3%** MoM (+8.2K orders vs. April)
- **Total Quantity Sold:** 48,233 units — up **+32.3%** MoM (+11.8K units vs. April)
- **Weekdays drive the majority of revenue:** $116.6K (74.4%) vs. $40.1K (25.6%) on weekends
- **Coffee** is the top-selling product category ($60.4K), followed by **Tea** ($44.5K) and **Bakery** ($18.6K)
- **Hell's Kitchen** is the top-performing store location, narrowly ahead of Astoria and Lower Manhattan
- Sales peak in the **morning hours (7–10 AM)**, consistent with a commuter coffee-run pattern
- **Barista Espresso** is the single best-selling product

---

## 📁 Repository Structure

```
coffee-shop-sales/
├── README.md
├── assets/
│   └── dashboard_overview.png
├── sql/
│   └── coffee_shop_sales_queries.sql
└── powerbi/
    └── Coffee_Shop_Sales.pbix
```


