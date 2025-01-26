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
   
3. How many unique customer orders were made?
4. How many successful orders were delivered by each runner?
5. How many of each type of pizza was delivered?
6. How many Vegetarian and Meatlovers were ordered by each customer?
7. What was the maximum number of pizzas delivered in a single order?
8. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
9. How many pizzas were delivered that had both exclusions and extras?
10. What was the total volume of pizzas ordered for each hour of the day?
11. What was the volume of orders for each day of the week?
