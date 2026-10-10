# Retail Sales Analysis SQL Project
Exploratory data analysis and business query optimization on retail transaction data using MySQL Workbench (CTEs, Window Functions, &amp; Data Aggregation).

## Project Overview
This project demonstrates core data analysis techniques using SQL to explore, clean, and analyze retail transaction data. The analysis covers end-to-end database creation, data inspection, exploratory data analysis (EDA), and answering targeted business queries regarding customer behavior, product category performance, and sales trends.

## Project Objective
To clean raw retail data, run exploratory analysis with SQL, and find the key sales trends like top product categories, peak shopping hours, and customer demographics that help to drive smarter business decisions.
 
 
## Database Engine: MySQL Workbench (MySQL 8.0+)

# Dataset Summary
**Total Records**: 2,000 transaction rows

**Unique Customers**: 155 customers

**Product Categories**: Beauty, Clothing, Electronics

**Total Net Sales**: $911,720.00

**Date Range**: 2022–2023

## Database & Table Schema

```sql
CREATE DATABASE Retail_sales_DB;

CREATE TABLE retail_sales (
    transactions_id INT PRIMARY KEY,
    sale_date DATE,
    sale_time TIME,
    customer_id INT,
    gender VARCHAR(15),
    age INT,
    category VARCHAR(20),
    quantity INT,
    price_per_unit FLOAT,
    cogs FLOAT,
    total_sale FLOAT);
```

# Key Business Questions & SQL Queries

## 1. Retrieve all sales made on a specific date (2022-11-05)
```sql
SELECT *
FROM retail_sales
WHERE sale_date = '2022-11-05';
```
## 2. Retrieve 'Clothing' transactions with quantity ≥ 4 in November 2022
```sql
SELECT *
FROM retail_sales
WHERE category = 'Clothing'
  AND DATE_FORMAT(sale_date, '%Y-%m') = '2022-11'
  AND quantity >= 4;
```
## 3. Calculate total revenue and total orders by product category
```sql
SELECT 
    category,
    SUM(total_sale) AS net_sale,
    COUNT(*) AS total_orders
FROM retail_sales
GROUP BY category;
```
## 4. Find the average age of customers purchasing from the 'Beauty' category
```sql
SELECT
    ROUND(AVG(age), 2) AS avg_age
FROM retail_sales
WHERE category = 'Beauty';
```

## 5. Find all high-value transactions where total sale exceeds 1000
```sql
SELECT * 
FROM retail_sales
WHERE total_sale > 1000;
```
## 6. Analyze transaction volume by category and gender
```sql
SELECT 
    category,
    gender,
    COUNT(*) AS total_transactions
FROM retail_sales
GROUP BY category, gender
ORDER BY category;
```

## 7. Find the best-selling month in each year based on average total sales
```sql
WITH monthly_sales AS (
    SELECT 
        YEAR(sale_date) AS year,
        MONTH(sale_date) AS month,
        AVG(total_sale) AS avg_sale,
        DENSE_RANK() OVER (
            PARTITION BY YEAR(sale_date) 
            ORDER BY AVG(total_sale) DESC
        ) AS rank_num
    FROM retail_sales
    GROUP BY YEAR(sale_date), MONTH(sale_date)
)
SELECT 
    year,
    month,
    avg_sale
FROM monthly_sales
WHERE rank_num = 1;
```

## 8. Identify the top 5 customers based on total spend
```sql
SELECT 
    customer_id,
    SUM(total_sale) AS total_sales
FROM retail_sales
GROUP BY customer_id
ORDER BY total_sales DESC
LIMIT 5;
```
## 9. Count unique customers who purchased from each product category
```sql
SELECT 
    category,    
    COUNT(DISTINCT customer_id) AS unique_customers
FROM retail_sales
GROUP BY category;
```
## 10. Categorize orders into time shifts (Morning, Afternoon, Evening)
```sql
WITH shift_data AS (
    SELECT 
        quantity,
        CASE 
            WHEN HOUR(sale_time) < 12 THEN 'Morning'
            WHEN HOUR(sale_time) BETWEEN 12 AND 17 THEN 'Afternoon'
            ELSE 'Evening'
        END AS shift
    FROM retail_sales
)
SELECT 
    shift,
    COUNT(*) AS total_orders
FROM shift_data
GROUP BY shift
ORDER BY total_orders DESC;
```

# Key Insights & Findings
Category Performance: Electronics generated the highest overall net revenue ($313,810), while Clothing had the highest transaction volume (702 orders).

Customer Demographics: Customer engagement across categories is balanced across genders, with the average customer age for Beauty products standing at 40.4 years.

Peak Sales Periods: July was the top-performing month in 2022 (avg sale: $541.34), while February led 2023 (avg sale: $535.53).

Shift Analysis: Evening shifts (> 5:00 PM) drive the vast majority of orders (1,062 orders), indicating peak shopping activity occurs during end-of-day hours.
