# Case Study #2 : Pizza Runner

<img src="https://github.com/user-attachments/assets/a1292c49-93df-4f1c-a31f-4870c7784c5c" alt="Case Study #2: Pizza Runner" width="300" height="300">

> [!NOTE]
> Dataset and images are all sourced from [#8WeekSQLChallenge](https://8weeksqlchallenge.com/)

## Table of Contents
- [Introduction](#introduction)
- [Table Relationship](#table-relationship)
- [Data Cleansing & Preparation](#data-cleansing--preparation)
  - [customer_orders](#1-customer_orders-table)
  - [runner orders](#2-runner_orders-table)
- [Business Questions](#business-questions)
  - [Pizza Metrics](#pizza-metrics)
  - [Runner and Customer Experience](#runner-and-customer-experience)
  - [Ingredient Optimization](#ingredient-optimization)
  - [Pricing and Ratings](#pricing-and-ratings)
- [Conclusion](#conclusion)
- [Bonus Questions](#bonus-questions)

## Introduction
Danny wants to expand his Pizza Empire so he decided to Uberize it! He then hired "runners" to deliver pizza from his headquarters and understands that data collection is crucial in expanding his business. He now needs help understanding this data and see how he can best leverage it to grow his empire.

## Table Relationship

<img src="https://github.com/user-attachments/assets/f0d5851e-9443-42ac-851d-ba7c08e7e062" alt="Case Study #2: Pizza Runner" width="600" height="300">

## Data Cleansing & Preparation
### 1. customer_orders table
<img src="https://github.com/user-attachments/assets/17322a9b-e3f4-4ad8-bece-6ab63e3c3c2c" alt="Case Study #2: Pizza Runner" width="600" height="300">

Here, we can see that 2 columns, exclusions and extras, need cleaning. To standardize, I will replace blank and 'null' values with NULL. 

```sql
CREATE TEMP TABLE customer_orders_temp AS
SELECT
  order_id,
  customer_id,
  pizza_id,
  CASE WHEN exclusions = 'null' THEN NULL
    ELSE NULLIF(exclusions,'')
    END exclusions,
  CASE WHEN extras = 'null' THEN NULL
    ELSE NULLIF(extras,'')
    END extras,
  order_time
FROM customer_orders;
```

**Result**

<img src="https://github.com/user-attachments/assets/18b6b3e3-135c-47ac-a5b4-fdb76fd4849d" alt="Case Study #2: Pizza Runner" width="500" height="300">
   
### 2. runner_orders table
<img src="https://github.com/user-attachments/assets/01916ea9-a6e2-4810-9676-c0ebd706b5ee" alt="Case Study #2: Pizza Runner" width="700" height="300">

Here, some columns not only need data cleansing but also data type correction. To standardize, we will perform the following steps:
- Replace 'null' and blank values with NULL in the pickup_time, distance, duration and cancellation columns.
- Alter the data type of pickup_time to timestamp.
- Remove 'km' value in the distance column and alter the data type to decimal.
- Remove 'minutes', 'mins', 'minute' in the duration column and alter the data type to int.

``` sql
CREATE TEMP TABLE runner_orders_temp AS
SELECT
  order_id,
  runner_id,
  CASE WHEN pickup_time = 'null' THEN NULL
    ELSE CAST(pickup_time AS timestamp)
    END pickup_time,
  CASE WHEN distance = 'null' THEN NULL
    ELSE CAST(TRIM(TRIM(distance,'km')) AS decimal(10,2))
    END distance,
  CASE WHEN duration='null' THEN NULL
    ELSE CAST(TRIM(TRIM(TRIM(TRIM(duration,'minutes'),'minute'),'mins')) AS int)
    END duration,
  CASE WHEN cancellation = 'null' OR cancellation = '' THEN NULL
    ELSE TRIM(cancellation)
    END cancellation
FROM runner_orders
```

**Result**

<img src="https://github.com/user-attachments/assets/564d37c4-dcf8-4f91-be41-991e978608b2" alt="Case Study #2: Pizza Runner" width="700" height="300">

## Business Questions
### Pizza Metrics
1. How many pizzas were ordered?
   ```sql
   SELECT
     COUNT(pizza_id) as pizza_order
   FROM customer_orders_temp
   ```

   **Result**
   
   <img src="https://github.com/user-attachments/assets/6ecae4e1-ba23-474e-9151-429a7f9e109a" alt="Case Study #2: Pizza Runner" width="130" height="70">

2. How many unique customer orders were made?
   ```sql
   SELECT
     COUNT(DISTINCT order_id) as order_cnt
   FROM customer_orders_temp
   ```
   
   **Result**
   
   <img src="https://github.com/user-attachments/assets/3754e323-a77e-4245-bb70-eae01fd3166b" alt="Case Study #2: Pizza Runner" width="130" height="70">

3. How many successful orders were delivered by each runner?
   ```sql
   SELECT
     r.runner_id,
     COUNT(r.order_id) as successful_order
   FROM runner_orders_temp r
   WHERE r.cancellation IS NULL
   GROUP BY r.runner_id
   ````

   **Result**
   
    <img src="https://github.com/user-attachments/assets/06101910-ad1d-4539-9a5f-dc17dd749dea" alt="Case Study #2: Pizza Runner" width="230" height="100">

4. How many of each type of pizza was delivered?
   ```
   SELECT
     p.pizza_name,
     COUNT(c.order_id) as delivered_cnt
   FROM customer_orders_temp c
   JOIN runner_orders_temp r ON c.order_id = r.order_id
   JOIN pizza_names p ON p.pizza_id = c.pizza_id
   WHERE cancellation IS NULL
   GROUP BY p.pizza_name
   ORDER BY pizza_name
   ```

   **Result**
   
   <img src="https://github.com/user-attachments/assets/e78f5c11-26f1-4508-a9c7-99220c2829f7" alt="Case Study #2: Pizza Runner" width="230" height="90">

5. How many Vegetarian and Meatlovers were ordered by each customer?
    ```sql
    SELECT c.customer_id,
      p.pizza_name,
      COUNT(c.order_id) as ordered_cnt
    FROM customer_orders_temp c
    JOIN runner_orders_temp r ON c.order_id = r.order_id
    JOIN pizza_names p ON p.pizza_id = c.pizza_id
    GROUP BY c.customer_id, p.pizza_name
    ORDER BY c.customer_id, pizza_name, ordered_cnt
    ```

    **Result**
   
    <img src="https://github.com/user-attachments/assets/73f3e836-a0ea-4beb-bc2a-06b7048ec040" alt="Case Study #2: Pizza Runner" width="280" height="200">

6. What was the maximum number of pizzas delivered in a single order?
    ```sql
    SELECT c.order_id,
      COUNT(c.pizza_id) as delivered_cnt
    FROM customer_orders_temp c
    JOIN runner_orders_temp r ON c.order_id = r.order_id
    WHERE r.cancellation IS NULL
    GROUP BY c.order_id
    ORDER BY delivered_cnt DESC
    LIMIT 1
    ```

    **Result**
   
    <img src="https://github.com/user-attachments/assets/7b02ac07-3b58-43bd-a9de-062744cba92c" alt="Case Study #2: Pizza Runner" width="230" height="70">

7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
    ```sql
    SELECT
      c.customer_id,
      SUM(
        CASE WHEN exclusions IS NULL AND extras IS NULL THEN 0
        ELSE 1
        END) atleast_one_change,
      SUM(
        CASE WHEN exclusions IS NULL AND extras IS NULL THEN 1
        ELSE 0
        END) no_change
    FROM customer_orders_temp c
    JOIN runner_orders_temp r ON c.order_id = r.order_id
    WHERE r.cancellation IS NULL
    GROUP BY c.customer_id
    ORDER BY c.customer_id 
    ```

    **Result**
   
    <img src="https://github.com/user-attachments/assets/054f7755-0b59-4f72-82aa-255655243bc5" alt="Case Study #2: Pizza Runner" width="330" height="150">
    
8. How many pizzas were delivered that had both exclusions and extras?
    ```sql
    SELECT
    SUM(
      CASE WHEN exclusions IS NOT NULL AND extras IS NOT NULL THEN 1
	    ELSE 0 
	    END) both_change
    FROM customer_orders_temp c
    JOIN runner_orders_temp r ON c.order_id = r.order_id
    WHERE r.cancellation IS NULL
    ```
    
    **Result**
   
    <img src="https://github.com/user-attachments/assets/3e8e0423-3ef2-4786-9110-ecf7d61bd159" alt="Case Study #2: Pizza Runner" width="130" height="70">

9. What was the total volume of pizzas ordered for each hour of the day?
    ```sql
    SELECT
      DATE_PART('hour',order_time) as hour,
      COUNT(order_id)
    FROM customer_orders_temp
    GROUP BY hour
    ORDER BY hour
    ```

    **Result**
   
    <img src="https://github.com/user-attachments/assets/c8bcf3ed-62c8-4a86-9598-97c613e30d37" alt="Case Study #2: Pizza Runner" width="200" height="150">

10. What was the volume of orders for each day of the week?
    ```sql
    SELECT
      DATE_PART('dow', order_time) dow,
      TO_CHAR(order_time,'Day') as dayoftheweek,
      COUNT(order_id) order_cnt
    FROM customer_orders_temp
    GROUP BY dow, dayoftheweek
    ORDER BY dow
    ```

    **Result**
    
    <img src="https://github.com/user-attachments/assets/111206a6-7861-4992-964b-549c8338ae09" alt="Case Study #2: Pizza Runner" width="250" height="100">

