## 1. Provide the list of markets in which customer "Atliq Exclusive" operates its business in the APAC region

SELECT 
  DISTINCT market 
FROM dim_customer
WHERE region = "APAC" and customer = "Atliq Exclusive";

## 2. What is the percentage of unique product increase in 2021 vs. 2020?

SELECT
uq_20.unique_products_2020,
uq_21.unique_products_2021,
ROUND((unique_products_2021 - unique_products_2020)*100 / unique_products_2020, 2) AS pct_change
FROM
(
(SELECT COUNT(DISTINCT product_code) AS unique_products_2020
FROM fact_sales_monthly
WHERE fiscal_year = 2020) uq_20,
  
(SELECT COUNT(DISTINCT product_code) AS unique_products_2021
FROM fact_sales_monthly
WHERE fiscal_year = 2021) uq_21
);
    
## 3. Provide a report with all the unique product counts for each segment and sort them in descending order of product counts

SELECT
segment,
COUNT(DISTINCT product_code) AS product_count
FROM dim_product
GROUP BY segment
ORDER BY product_count DESC;

## 4. Follow-up: Which segment had the most increase in unique products in 2021 vs 2020?

with cte1 AS (
SELECT
p.segment AS sg1,
COUNT(DISTINCT s.product_code) AS product_count_2020
FROM dim_product p
JOIN fact_sales_monthly s ON p.product_code = s.product_code
JOIN dim_date d ON s.fiscal_year = d.fiscal_year
WHERE s.fiscal_year = 2020
GROUP BY segment), 
cte2 AS (
SELECT
p.segment AS sg2,
COUNT(DISTINCT s.product_code) AS product_count_2021
FROM dim_product p
JOIN fact_sales_monthly s ON p.product_code = s.product_code
JOIN dim_date d ON s.fiscal_year = d.fiscal_year
WHERE s.fiscal_year = 2021
GROUP BY segment)

SELECT cte1.sg1, cte1.product_count_2020, cte2.product_count_2021, (cte2.product_count_2021 - cte1.product_count_2020) AS difference
FROM cte1
JOIN cte2
ON cte1.sg1=cte2.sg2
ORDER BY difference DESC;

## 5. Get the products that have the highest and lowest manufacturing costs.

(SELECT
fm.product_code,
p.product,
fm.manufacturing_cost
FROM dim_product p, fact_manufacturing_cost fm
WHERE p.product_code = fm.product_code
ORDER BY manufacturing_cost
LIMIT 1)
UNION
(SELECT
fm.product_code,
p.product,
fm.manufacturing_cost
FROM dim_product p, fact_manufacturing_cost fm
WHERE p.product_code = fm.product_code
ORDER BY manufacturing_cost DESC
LIMIT 1);

## 6. Generate a report which contains the top 5 customers who received an average high pre_invoice_discount_pct for the fiscal year 2021 and in the 
Indian market. 

SELECT
pre.customer_code,
c.customer,
pre.pre_invoice_discount_pct AS average_discount_percentage
FROM dim_customer c
JOIN fact_pre_invoice_deductions pre ON c.customer_code = pre.customer_code
WHERE pre.fiscal_year=2021 AND c.market="India" AND pre.pre_invoice_discount_pct > (SELECT AVG(pre_invoice_discount_pct) FROM fact_pre_invoice_deductions)
GROUP BY c.customer_code
ORDER BY average_discount_percentage DESC
LIMIT 5;

## 7. Get the complete report of the Gross sales amount for the customer “Atliq Exclusive” for each month. 
-- This analysis helps to get an idea of low and high-performing months and take strategic decisions.

SELECT
CONCAT(MONTHNAME(s.date), '(', YEAR(s.date), ')') AS month,
s.fiscal_year,
ROUND(SUM(g.gross_price * s.sold_quantity), 2) as gross_sales_amt
FROM fact_sales_monthly s
JOIN dim_customer c ON s.customer_code = c.customer_code
JOIN fact_gross_price g ON g.product_code = s.product_code
WHERE c.customer = "Atliq Exclusive"
GROUP BY month, s.fiscal_year
ORDER BY s.fiscal_year;

## 8. In which quarter of 2020, got the maximum total_sold_quantity?

-- creating a function which will return fiscal quarters

DROP FUNCTION IF EXISTS get_fiscal_quarter;
DELIMITER //

CREATE FUNCTION `get_fiscal_quarter`(calendar_date date) RETURNS CHAR(2) DETERMINISTIC
BEGIN
DECLARE m TINYINT;
DECLARE qtr CHAR(2);
SET m = MONTH(calendar_date);
    
CASE
WHEN m IN (9,10,11) THEN SET qtr = "Q1";
WHEN m IN (12,1,2) THEN SET qtr = "Q2";
WHEN m IN (3,4,5) THEN SET qtr = "Q3";
WHEN m IN (6,7,8) THEN SET qtr = "Q4";
END CASE;
    
return qtr;
        
END

//
DELIMITER ;

SELECT 
get_fiscal_quarter(date) AS Quarter,
SUM(sold_quantity) AS total_sold_quantity
FROM fact_sales_monthly
WHERE fiscal_year=2020
GROUP BY Quarter;

## 9. Which channel helped to bring more gross sales in the fiscal year 2021 and the percentage of contribution?

WITH cte1 AS (
SELECT 
c.channel, 
SUM(s.sold_quantity*g.gross_price) AS total_sales
FROM fact_sales_monthly s
JOIN dim_customer c ON s.customer_code=c.customer_code
JOIN fact_gross_price g ON s.product_code=g.product_code
WHERE s.fiscal_year=2021
GROUP BY c.channel
ORDER BY total_sales DESC)

SELECT channel, ROUND(total_sales/1000000,2) AS gross_sales_mln,
ROUND(total_sales/(SUM(total_sales) OVER())*100,2) AS percentage
FROM cte1; 

## 10. Get the Top 3 products in each division that have a high total_sold_quantity in the fiscal_year 2021?

WITH cte1 AS (
SELECT
p.division, s.product_code, p.product,
SUM(s.sold_quantity) as total_sold_quantity
FROM fact_sales_monthly s
JOIN dim_product P ON s.product_code=p.product_code
WHERE s.fiscal_year = 2021 
GROUP BY s.product_code, division, P.product),
cte2 AS (
SELECT division, product_code, product, total_sold_quantity,
RANK() OVER(PARTITION BY division ORDER BY total_sold_quantity DESC) AS 'rank_order'
FROM cte1)

SELECT cte1.division, cte1.product_code, cte1.product, cte2.total_sold_quantity, cte2.rank_order
FROM cte1 
JOIN cte2 
ON cte1.product_code=cte2.product_code
WHERE rank_order <= 3;
