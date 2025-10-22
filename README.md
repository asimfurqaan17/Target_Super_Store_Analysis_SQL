
LINK TO THE DATASET: https://drive.google.com/drive/folders/1K2uTG0kpPqZ65PhVhQNTFgyfEqES_dWt



# Target_Super_Store_Analysis_SQL
Answered Industry Level Questions Regarding Target's Data. 

# Q1) Import the dataset and do usual exploratory analysis steps like checking the structure & characteristics of the dataset:
 #1. Data type of all columns in the "customers" table.
 #2. Get the time range between which the orders were placed.
 #3. Count the Cities & States of customers who ordered during the given period.

# 1. To view the data types all columns in the customers table: 
SELECT * FROM `target-475623.TARGERT_SQL.customers`
limit 10;

# 2. To get the time range between the orders which were placed.
SELECT 
min(order_purchase_timestamp) as start_time,
max(order_purchase_timestamp) as end_time
FROM `target-475623.TARGERT_SQL.orders`;

# 3. To Count the Cities & States of customers who ordered during the given period.
WITH first_quarter_2018 as ( 
SELECT
c.customer_city,c.customer_state
FROM `target-475623.TARGERT_SQL.orders`  as o
JOIN `target-475623.TARGERT_SQL.customers`as c
ON o.customer_id=c.customer_id
WHERE EXTRACT(YEAR FROM o.order_purchase_timestamp) = 2018
AND EXTRACT(MONTH FROM order_purchase_timestamp) BETWEEN 1 AND 3 
)

SELECT
customer_city,
COUNT (*) as city_count
FROM first_quarter_2018
GROUP BY customer_city
ORDER BY city_count DESC; 


WITH first_quarter_2018 as ( 
SELECT
c.customer_city,c.customer_state
FROM `target-475623.TARGERT_SQL.orders`  as o
JOIN `target-475623.TARGERT_SQL.customers`as c
ON o.customer_id=c.customer_id
WHERE EXTRACT(YEAR FROM o.order_purchase_timestamp) = 2018
AND EXTRACT(MONTH FROM order_purchase_timestamp) BETWEEN 1 AND 3 
)

SELECT
customer_state,
COUNT (*) as state_count
FROM first_quarter_2018
GROUP BY customer_state
ORDER BY state_count DESC;

# Q2) In-depth Exploration:

#1. Is there a growing trend in the no. of orders placed over the past years & can we see some kind of monthly seasonality in terms of the no. of
#orders being placed?
SELECT
EXTRACT(month from order_purchase_timestamp) as month,
COUNT(order_id) as order_number
FROM `TARGERT_SQL.orders`
GROUP BY month
ORDER BY order_number DESC;


# Q2) Evolution of E-commerce orders in the Brazil region:

#1. Finding month on month number of orders
SELECT
EXTRACT(MONTH FROM order_purchase_timestamp) as month,
EXTRACT(YEAR FROM order_purchase_timestamp) as year,
COUNT (*) as num_orders
FROM `TARGERT_SQL.orders`
GROUP BY month, year
ORDER BY month, year;

# 2. How are customers distibuted across the states. 
SELECT customer_city, customer_state,
COUNT(DISTINCT customer_id) as customer_count
FROM `TARGERT_SQL.customers`
GROUP BY customer_city, customer_state
ORDER by customer_count DESC;


#Q4) Impact on Economy: Analyze the money movement by e-commerce by looking
#at order prices, freight and others.

# 1.  Get the % increase in the cost of orders from year 2017 to 2018
#(include months between Jan to Aug only).
#You can use the "payment_value" column in the payments table to get
#the cost of orders.

#STEP 1: Calculating the total payments per year
WITH yearly_totals as(
SELECT 
EXTRACT(YEAR from o.order_purchase_timestamp) as year,
SUM(p.payment_value) as total_payment
FROM `TARGERT_SQL.payments` as p
JOIN TARGERT_SQL.orders as o
ON p.order_id = o.order_id
WHERE EXTRACT(YEAR from o.order_purchase_timestamp) in (2017,2018) 
 and EXTRACT(MONTH from o.order_purchase_timestamp) in (1,8)
GROUP BY year
ORDER BY year
),

#STEP 2: Use Lead function to compare each years payments with the previous years
yearly_total_comparison AS (
 SELECT
 year,
 total_payment,
 LEAD(total_payment) over (ORDER BY year DESC) as prev_year_payments
 FROM yearly_totals
)

#STEP 3: Finding Percentage increase
SELECT ((total_payment - prev_year_payments) / prev_year_payments) * 100 
FROM yearly_total_comparison;

#2. Calculate the Total & Average value of order price and order freight for each state.
SELECT 
c.customer_state,c.customer_city,
ROUND(AVG(price), 2) as avg_price,
ROUND(SUM(price), 2) as sum_price,
ROUND(AVG(freight_value), 2) as avg_freight,
ROUND(SUM(freight_value), 2) as sum_freight
FROM `TARGERT_SQL.orders` as o
JOIN TARGERT_SQL.order_items as oi
ON o.order_id=oi.order_id
JOIN `TARGERT_SQL.customers`as c
ON c.customer_id = o.customer_id
GROUP BY c.customer_state,c.customer_city;

#Q5) Analysis based on sales, freight and delivery time.

#1. Find the no. of days taken to deliver each order from the order’s
#purchase date as delivery time. Also, calculate the difference (in days) between the estimated & actual
#delivery date of an order.
SELECT order_id,
DATE_DIFF(DATE(order_delivered_customer_date), DATE(order_purchase_timestamp), DAY) as days_to_delivery,
DATE_DIFF(DATE(order_delivered_customer_date), DATE(order_estimated_delivery_date), DAY) as diff_estimated_delivery
FROM `TARGERT_SQL.orders`;

# Another Approach

SELECT
  order_id,
  DATE(order_delivered_customer_date) - DATE(order_purchase_timestamp) AS days_to_delivery,
  DATE(order_delivered_customer_date) - DATE(order_estimated_delivery_date) AS diff_estimated_delivery
FROM `TARGERT_SQL.orders`;


# 2. Find out the top 5 states with the highest & lowest average freight value.

SELECT c.customer_state, c.customer_city,
ROUND(AVG(freight_value), 2) as avg_frieght
FROM `TARGERT_SQL.orders` as o
JOIN TARGERT_SQL.order_items as oi
ON o.order_id = oi.order_id
JOIN `TARGERT_SQL.customers`as c
ON c.customer_id=o.customer_id
GROUP BY c.customer_state, c.customer_city
ORDER BY avg_frieght DESC
LIMIT 5;

# 3. Find out the top 5 states with the highest & lowest average delivery time.
SELECT c.customer_state, c.customer_city,
ROUND(AVG(DATE_DIFF(DATE(o.order_delivered_customer_date), DATE(o.order_purchase_timestamp), DAY)), 2) AS avg_time_to_delivery_days
FROM `TARGERT_SQL.orders`as o
JOIN `TARGERT_SQL.order_items`as oi
ON o.order_id = oi.order_id
JOIN `TARGERT_SQL.customers`as c
ON c.customer_id =o.customer_id
GROUP BY c.customer_state, c.customer_city
ORDER BY avg_time_to_delivery_days DESC
LIMIT 5;

#4. Find out the top 5 states where the order delivery is really fast as compared to the estimated date of delivery.
SELECT c.customer_state, c.customer_city,
ROUND(AVG(DATE_DIFF(DATE(o.order_delivered_customer_date), DATE(o.order_purchase_timestamp), DAY)), 2) AS avg_time_to_delivery_days
FROM `TARGERT_SQL.orders`as o
JOIN `TARGERT_SQL.order_items`as oi
ON o.order_id = oi.order_id
JOIN `TARGERT_SQL.customers`as c
ON c.customer_id =o.customer_id
GROUP BY c.customer_state, c.customer_city
ORDER BY avg_time_to_delivery_days ASC
LIMIT 5;

#Q6) Analysis based on the payments:

#1. Find the month on month no. of orders placed using different payment types.
SELECT payment_type,
EXTRACT(YEAR FROM o.order_purchase_timestamp) as year,
EXTRACT(MONTH FROM o.order_purchase_timestamp) as month,
FROM `TARGERT_SQL.orders` as o
JOIN TARGERT_SQL.payments as p
ON o.order_id=p.order_id
GROUP BY payment_type, year, month
ORDER BY payment_type, year, month;

#2. Find the no. of orders placed on the basis of the payment installments that have been paid.
SELECT payment_installments,
COUNT(DISTINCT order_id) as num_orders
FROM `TARGERT_SQL.payments`
GROUP BY payment_installments
