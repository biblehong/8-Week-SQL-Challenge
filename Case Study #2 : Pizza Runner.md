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

<img src="https://github.com/user-attachments/assets/18b6b3e3-135c-47ac-a5b4-fdb76fd4849d" alt="Case Study #2: Pizza Runner" width="550" height="300">
   
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

<img src="https://github.com/user-attachments/assets/e25d679b-77c9-43f4-8c5e-afe6e65ed7b0" alt="Case Study #2: Pizza Runner" width="700" height="300">

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

   - A total of 14 pizzas were ordered.
     
2. How many unique customer orders were made?
   ```sql
   SELECT
     COUNT(DISTINCT order_id) as order_cnt
   FROM customer_orders_temp
   ```
   
   **Result**
   
   <img src="https://github.com/user-attachments/assets/3754e323-a77e-4245-bb70-eae01fd3166b" alt="Case Study #2: Pizza Runner" width="130" height="70">

   - 10 unique orders were made

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

   - runner 1 made the most delivery with 4 completed orders followed by runner 2 with 3 and runner 3 with 1

4. How many of each type of pizza was delivered?
   ```sql
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

   - 9 Meatlovers were ordered while 3 for Vegetarian pizzas.  

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

    - Most pizza ordered in a single order was with order id 4

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

    - There were equal number of pizzas ordered with change and no change among the customers
    
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

    - Only 1 pizza order had both exclusion and extra

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

    - Pizza order peaked during the afternoon between 1pm - 6pm and late night between 9pm - 11pm 

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

    - Best pizza sales were during Wednesday and Friday.

### Runner and Customer Experience
1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
   ```sql
   SELECT
     CEIL(DATE_PART('day',registration_date)/7) week,
     COUNT(runner_id)
   FROM runners
   GROUP BY week
   ORDER BY week
   ```
   
   **Result**
   
   <img src="https://github.com/user-attachments/assets/08680508-6951-4f11-afac-a42d1df2d593" alt="Case Study #2: Pizza Runner" width="200" height="100">

2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
   ```sql
   WITH average as (
   SELECT DISTINCT
     r.runner_id,
     c.order_id,
     (r.pickup_time - c.order_time) as interval_mins
   FROM customer_orders_temp c
   JOIN runner_orders_temp r ON c.order_id = r.order_id
   WHERE pickup_time IS NOT NULL
   )
   SELECT DISTINCT
     runner_id,
     AVG(interval_mins) average_mins
   FROM average
   GROUP BY runner_id
   ```
   **Result**

   <img src="https://github.com/user-attachments/assets/a721ceae-506a-4768-8a27-3effe00c23a5" alt="Case Study #2: Pizza Runner" width="200" height="100">

   - Runner 2 took the most time to arrive to the HQ for pickup while runner 3 was the quickest.

3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
   ```sql
   WITH prep AS (
   SELECT
     c.order_id,
     COUNT(pizza_id) pizza_cnt,
     MAX(r.pickup_time - c.order_time) prep_time
   FROM customer_orders_temp c
   JOIN runner_orders_temp r ON c.order_id = r.order_id
   WHERE r.cancellation IS NULL
   GROUP BY c.order_id
   )
   SELECT
     pr.pizza_cnt,
     AVG(pr.prep_time)
   FROM prep pr
   GROUP BY pr.pizza_cnt
   ```
   **Result**

   <img src="https://github.com/user-attachments/assets/1d74de23-c648-4015-9a29-ba81489a8dea" alt="Case Study #2: Pizza Runner" width="200" height="100">

   - Pizza bought in bundles of 3 took the longest to prepare. The average timing increases as the number of pizzas ordered increases likely indicating that Danny's HQ only has limited ovens to cater for bigger orders.

4. What was the average distance travelled for each customer?
   ```sql
   WITH average as (
   SELECT DISTINCT
     c.customer_id,
     c.order_id,
     r.distance
   FROM customer_orders_temp c
   JOIN runner_orders_temp r ON c.order_id = r.order_id
   WHERE distance IS NOT NULL
   ORDER BY c.customer_id, c.order_id
   )
   SELECT DISTINCT
     customer_id,
     ROUND(AVG(distance),2) average_km
   FROM average
   GROUP BY customer_id
   ```
   
   **Result**

   <img src="https://github.com/user-attachments/assets/ab2b3656-9495-44ec-a345-5206a5e1ef9f" alt="Case Study #2: Pizza Runner" width="180" height="130">

   - Customer 105 was the farthest customer by distance while Customer 104 was the closest.

5. What was the difference between the longest and shortest delivery times for all orders?
   ```sql
   SELECT MAX(duration) - MIN(duration) as max_min
   FROM runner_orders_temp
   ```
   
   **Result**

   <img src="https://github.com/user-attachments/assets/c9a43dc7-fcb8-4468-9ec7-d52a068c6fb3" alt="Case Study #2: Pizza Runner" width="150" height="80">

6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
   ```sql
   SELECT
     runner_id,
     ROUND(avg(duration),2) avg_duration
   FROM runner_orders_temp
   WHERE duration IS NOT NULL
   GROUP BY runner_id
   ORDER BY runner_id
   ```
   
   **Result**
   
    <img src="https://github.com/user-attachments/assets/96e0a922-25fa-4611-9579-005445ad48c2" alt="Case Study #2: Pizza Runner" width="200" height="100">

    - Runner 3 was the fastest among all the runners, whether it was in terms of delivery time or arrival time to the HQ to pickup the order. Danny should consider looking at runner 2's performance as he took the longest in both areas. 

7. What is the successful delivery percentage for each runner?
   ```sql
   SELECT
     runner_id,
     ROUND(
       SUM(
         CAST(
           CASE WHEN cancellation IS NULL THEN 1
           ELSE 0
           END
         AS DECIMAL(10,2)
           )
         )/COUNT(order_id)*100,2)
   FROM runner_orders_temp
   GROUP BY runner_id
   ```

   **Result**
   
   <img src="https://github.com/user-attachments/assets/a4d4ee7f-f55b-464b-9254-e1ce844eb8fc" alt="Case Study #2: Pizza Runner" width="180" height="100">

   - Runner 1 was able to deliver 100% of his deliveries. At this point, it seems like runner 1 is showing a promising result in different delivery KPIs.

### Ingredient Optimization
1. What are the standard ingredients for each pizza?
   ```sql
   WITH recipes AS (
   SELECT
     pr.pizza_id,
     TRIM(string_to_table(toppings,','))::int as topping_id
   FROM pizza_recipes pr
   )
   SELECT
   pt.topping_name,
   COUNT(DISTINCT r.pizza_id) AS common_ingredient
   FROM recipes r
   JOIN pizza_toppings pt ON r.topping_id = pt.topping_id
   GROUP BY pt.topping_name
   HAVING COUNT(DISTINCT r.pizza_id) > 1
   ```

   **Result**

   <img src="https://github.com/user-attachments/assets/a74886ba-645f-4fde-9c97-2cbd86b6ba93" alt="Case Study #2: Pizza Runner" width="250" height="80">

   - Cheese and Mushrooms were always included in both types of pizza.

2. What was the most commonly added extra?
   ```sql
   WITH extra_toppings AS (
   SELECT
     pizza_id,
     TRIM(string_to_table(extras ,','))::int AS topping_id
   FROM customer_orders_temp c
   )
   SELECT
     pt.topping_name,
     COUNT(ex.pizza_id) extras_cnt
   FROM extra_toppings ex
   JOIN pizza_toppings pt on ex.topping_id = pt.topping_id
   GROUP BY pt.topping_name
   ORDER BY extras_cnt DESC
   LIMIT 1
   ```

   **Result**

   <img src="https://github.com/user-attachments/assets/ee6ec959-3681-436f-a105-932bde9853f4" alt="Case Study #2: Pizza Runner" width="270" height="80">

   - Bacon was the most requested extra topping in several orders.

3. What was the most common exclusion?
   ```sql
   WITH excl_toppings AS (
   SELECT
     pizza_id,
     TRIM(string_to_table(exclusions ,','))::int AS topping_id
   FROM customer_orders_temp c
   )
   SELECT
     pt.topping_name,
     COUNT(ex.pizza_id) excl_cnt
   FROM excl_toppings ex
   JOIN pizza_toppings pt on ex.topping_id = pt.topping_id
   GROUP BY pt.topping_name
   ORDER BY excl_cnt DESC
   LIMIT 1
   ```

   **Result**
   
   <img src="https://github.com/user-attachments/assets/30ecfd4e-ec4a-4e59-ac86-3d727aef7ae0" alt="Case Study #2: Pizza Runner" width="270" height="80">

   - Cheese was the most requested exclusion in several orders.

4. Generate an order item for each record in the customers_orders table in the format of one of the following:
   - Meat Lovers
   - Meat Lovers - Exclude Beef
   - Meat Lovers - Extra Bacon
   - Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

   ```sql
   WITH pk as (
   SELECT
     c.pizza_id,
     c.order_id,
     c.exclusions,
     c.extras,
     ROW_NUMBER() OVER(ORDER BY order_id, pizza_id) pk
   FROM customer_orders_temp c
   ),
   excl_split AS (
   SELECT
     pk,
     TRIM(STRING_TO_TABLE(exclusions,', '))::int AS excl
   FROM pk
   ),
   exclusions AS (
   SELECT
     ex.pk,
     'Exclude ' || STRING_AGG(pt.topping_name,', ' ORDER BY pt.topping_name) as excluded
   FROM excl_split ex
   JOIN pizza_toppings pt ON pt.topping_id = ex.excl
   GROUP by ex.pk
   ),
   ext_split AS (
   SELECT
     pk,
     TRIM(STRING_TO_TABLE(extras,','))::int AS extr
   FROM pk
   ),
   extras AS (
   SELECT
     ext.pk,
     'Extra ' || STRING_AGG(pt.topping_name,', ' ORDER BY pt.topping_name) as extras
   FROM ext_split ext
   JOIN pizza_toppings pt ON pt.topping_id = ext.extr
   GROUP by ext.pk
   )
   SELECT
     pk.order_id,
     pn.pizza_name ||
       CASE WHEN ex.excluded IS NOT NULL THEN ' - ' || ex.excluded
       ELSE ''
       END ||
         CASE WHEN ext.extras IS NOT NULL THEN ' - ' || ext.extras
         ELSE ''
         END order_list
   FROM pk
   JOIN pizza_names pn on pk.pizza_id = pn.pizza_id
   LEFT JOIN exclusions ex ON pk.pk = ex.pk
   LEFT JOIN extras ext ON pk.pk = ext.pk
   ORDER BY order_id
   ```

   **Result**

   <img src="https://github.com/user-attachments/assets/0f89818b-02b5-4f36-977c-c4a9e32021e5" alt="Case Study #2: Pizza Runner" width="360" height="300">

5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
   - For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
   ```sql
   WITH pk as (
   SELECT
     c.pizza_id,
     c.order_id,
     c.exclusions,
     c.extras,
     ROW_NUMBER() OVER(ORDER BY order_id, pizza_id) pk
   FROM customer_orders_temp c
   ),
   ingr AS(
   SELECT
     pk,
     TRIM(STRING_TO_TABLE(pr.toppings,','))::int AS ingr
   FROM pk
   JOIN pizza_recipes pr on pr.pizza_id = pk.pizza_id
   ),
   ext_split AS (
   SELECT
     pk,
     TRIM(STRING_TO_TABLE(extras,','))::int AS ingr
   FROM pk
   ),
   excl_split AS (
   SELECT
     pk,
     TRIM(STRING_TO_TABLE(exclusions,', '))::int AS ingr
   FROM pk
   ),
   combined_ingr AS (
   SELECT
     pk,
     ingr,
     COUNT(ingr) as total_ingr
   FROM (
       SELECT * FROM ingr
         UNION ALL
       SELECT * FROM ext_split
         EXCEPT ALL
       SELECT * FROM excl_split
     )
   GROUP BY pk, ingr
   ),
   concat_ingr as (
   SELECT
     ing.pk,
     STRING_AGG(
       CASE WHEN total_ingr > 1 THEN total_ingr || 'x '
       ELSE '' END ||
       pt.topping_name, ', '
     ORDER BY pt.topping_name) as ingredient_list
   FROM combined_ingr ing
   JOIN pizza_toppings pt on pt.topping_id = ing.ingr
   GROUP BY ing.pk
   ORDER BY pk
   )
   SELECT
     pk.order_id,
     pn.pizza_name || ' : ' || ci.ingredient_list as order_list
   FROM pk
   JOIN pizza_names pn on pk.pizza_id = pn.pizza_id
   JOIN concat_ingr ci ON pk.pk = ci.pk
   ORDER BY order_id
   ```

   **Result**

   <img src="https://github.com/user-attachments/assets/90a79e4f-ed00-4154-b800-19357d279a10" alt="Case Study #2: Pizza Runner" width="430" height="300">

6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
   ```sql
   WITH ingr AS(
   SELECT
     order_id,
     TRIM(STRING_TO_TABLE(pr.toppings,','))::int AS ingr
   FROM customer_orders_temp c
   JOIN pizza_recipes pr on pr.pizza_id = c.pizza_id
   ),
   ext_split AS (
   SELECT
     order_id,
     TRIM(STRING_TO_TABLE(extras,','))::int AS ingr
   FROM customer_orders_temp
   ),
   excl_split AS (
   SELECT
     order_id,
     TRIM(STRING_TO_TABLE(exclusions,', '))::int AS ingr
   FROM customer_orders_temp
   )
   SELECT
     pt.topping_name,
     COUNT(ingr) as total_ingr
   FROM (
		SELECT * FROM ingr
			UNION ALL
		SELECT * FROM ext_split
		 	EXCEPT ALL
		SELECT * FROM excl_split
   ) i
   JOIN pizza_toppings pt on pt.topping_id = i.ingr
   GROUP BY pt.topping_name
   ORDER BY total_ingr DESC
   ```

   **Result**

   <img src="https://github.com/user-attachments/assets/980194b8-ce6b-4e37-a782-e228ac86b2b4" alt="Case Study #2: Pizza Runner" width="200" height="300">

   - Bacon was the most used ingredient in most of the orders, as a default ingredient or as an add-on in the pizza ordered. Danny should consider keeping an eye on this ingredient's inventory to avoid running out and losing sales in the process.

### Pricing and Ratings
1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?
   ```sql
   SELECT
     SUM(
       CASE WHEN pn.pizza_name = 'Meatlovers'
       THEN 12
       ELSE 10 END) as total_sales
   FROM customer_orders_temp c
   JOIN pizza_names pn ON pn.pizza_id = c.pizza_id
   ```

   **Result**

    <img src="https://github.com/user-attachments/assets/1744af8f-cf9a-475e-9f47-1a25dc65d225" alt="Case Study #2: Pizza Runner" width="150" height="80">

    - Total Sales is $160. 

2. What if there was an additional $1 charge for any pizza extras?
   - Add cheese is $1 extra
     ```sql
     SELECT
       SUM(
         CASE WHEN pn.pizza_name = 'Meatlovers' THEN 12
         ELSE 10
         END
         +
         CASE WHEN extras LIKE '%4%' THEN 1
         ELSE 0
         END) as total_sales
      FROM customer_orders_temp c
      JOIN pizza_names pn ON pn.pizza_id = c.pizza_id
     ```

     **Result**

     <img src="https://github.com/user-attachments/assets/12557f0b-a380-4ddb-99d6-e220475c45b9" alt="Case Study #2: Pizza Runner" width="150" height="80">

     - No significant change in the total sales as there was only 1 order has an add-on request for cheese. Charging extra for Bacon would increase the revenue. 

3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.
   ```sql
   CREATE SCHEMA runner_rating;
   SET search_path = runner_rating;

   CREATE TABLE rating (
   "order_id" INTEGER,
   "rating" INTEGER,
   "comment" VARCHAR(100)
   );

   INSERT INTO rating
   ("order_id", "rating","comment")
   VALUES
   ('1', '3','food was already cold. pretty disappointed'),
   ('2', '4',NULL),
   ('3', '3',NULL),
   ('4', '2','delivery took way too long!'),
   ('5', '4','excellent delivery time. thank you'),
   ('7', '4',NULL),
   ('8', '5',NULL),
   ('10', '5','delivery was super fast!');
   ```

   **Result**
   
   <img src="https://github.com/user-attachments/assets/b19ffe92-0683-4b4e-a4de-c5dfd9bcafc5" alt="Case Study #2: Pizza Runner" width="500" height="270">
   
4. Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries
   - customer_id
   - order_id
   - runner_id
   - rating
   - order_time
   - pickup_time
   - Time between order and pickup
   - Delivery duration
   - Average speed
   - Total number of pizzas
     ```sql
     SELECT
       c.customer_id,
       c.order_id,
       r.runner_id,
       rr.rating,
       rr.comment,
       c.order_time,
       r.pickup_time,
       r.pickup_time-c.order_time as time_between_order_and_pickup,
       r.duration as delivery_duration,
       c.total_pizza
     FROM (
       SELECT
         customer_id,
         order_id,
         order_time,
         COUNT(pizza_id) total_pizza
       FROM customer_orders_temp
       GROUP BY customer_id, order_id, order_time
     ) c
     JOIN runner_orders_temp r ON c.order_id = r.order_id
     JOIN runner_rating.rating rr ON rr.order_id = c.order_id
     WHERE r.cancellation IS NULL
     ORDER BY customer_id, order_id
     ```

     **Result**
     
     <img src="https://github.com/user-attachments/assets/8ae60328-17b8-4aaa-8cfa-2c009f1221e5" alt="Case Study #2: Pizza Runner" width="1200" height="200">
     
5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled
   - how much money does Pizza Runner have left over after these deliveries?
   ```
   WITH sales AS (
   SELECT
     order_id,
     SUM(
       CASE WHEN pn.pizza_name = 'Meatlovers' THEN 12
       ELSE 10
       END) as sales_per_order
   FROM customer_orders_temp c
   JOIN pizza_names pn ON pn.pizza_id = c.pizza_id
   GROUP BY order_id
   )
   SELECT SUM(ROUND(sales_per_order - r.distance*0.3,2)) as net_sales
   FROM sales s
   JOIN runner_orders_temp r ON s.order_id = r.order_id
   WHERE r.cancellation IS NULL
   ```

   **Result**
   
   <img src="https://github.com/user-attachments/assets/997de784-09e2-4f81-8fb0-9618941652ca" alt="Case Study #2: Pizza Runner" width="150" height="80">

   - Pizza Runner would have an ending net revenue of $94.44. Assuming that Danny does not charge for extra toppings and excluding the cost of the ingredients, his pizza restaurant would be making 59% profit margin. This will of course further reduce once he takes the ingredient costs into consideration.
  
## Conclusion
Best sales are observed during Wednesday and Friday, during afternoon 1pm-6pm and late evening 9pm - 11pm. Danny should ensure that there are enough runners to accommodate deliveries during these periods.

Runner 1 has the overall best performance in terms of pickup time, delivery time and percentage of successful delivery. Runner 3 is the fastest among all the runners but was only able to deliver 50% of the orders assigned to him. Danny should review runner 2's performance as he is lacking in all the areas. 

Bacon is the most requested add-on. Danny should keep an eye on this ingredient's inventory and consider adding an add-on charge for this ingredient to increase the revenue. 

Runner fee is too expensive for single-pizza orders delivered to a great distance location. Danny should consider the following: 
1. Narrowing down his delivery area.
2. Imposing a minumum number of pizzas in a single order for locations farther than 10km.
3. Running a campaign such as a self-collect discount to reduce operating costs.
4. Consider having runners deliver multiple orders around the same location. 
