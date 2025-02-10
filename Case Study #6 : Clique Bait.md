# Case Study #6 : Clique Bait

<img src="https://github.com/user-attachments/assets/c7db1eca-bcd3-46eb-8c8d-cb21262f3148" alt="Case Study #6: Clique Bait" width="300" height="300">

> [!NOTE]
> Dataset and images are all sourced from [#8WeekSQLChallenge](https://8weeksqlchallenge.com/)

## Table of Contents
- [Introduction](#introduction)
- [Table Relationship](#table-relationship)
- [Business Questions](#business-questions)
  - [Digital Analysis](#digital-analysis)
  - [Product Funnel Analysis](#product-funnel-analysis)
  - [Campaigns Analysis](#campaigns-analysis)
- [Conclusion](#conclusion)

## Introduction

Clique Bait's CEO Danny wants to expand his knowledge in the seafood industry and is looking for some help to analyze the dataset of his online seafood store. Help Danny uncover interesting insights including coming up with creative solutions to calculate funnel fallout rates for his store.

## Table Relationship

<img src="https://github.com/user-attachments/assets/7635dddd-3dc5-4c56-9f0e-3153235a71e8" alt="Case Study #6: Clique Bait" width="650" height="400">

## Business Questions
### Digital Analysis
Using the available datasets - answer the following questions using a single query for each one:

1. How many users are there?
   ```sql
   SELECT
     COUNT(*) AS cnt_users
   FROM clique_bait.users
   ```

   **Result**

   <img src="https://github.com/user-attachments/assets/f77f5fc1-6b7d-4a4c-a478-c0a442e0e074" alt="Case Study #6: Clique Bait" width="150" height="80">

2. How many cookies does each user have on average?
   ```sql
   WITH cookies AS (
   SELECT
     user_id,
     COUNT(*) total_cookies
   FROM clique_bait.users
   GROUP BY user_id
   )
   SELECT
     ROUND(AVG(total_cookies),0) as avg_cookies
   FROM cookies
   ```

   **Result**
   
   <img src="https://github.com/user-attachments/assets/a4565bb5-3c2d-4c83-80f4-11ebf1b2db67" alt="Case Study #6: Clique Bait" width="150" height="80">

3. What is the unique number of visits by all users per month?
   ```sql
   SELECT
     DATE_PART('month',event_time) month_no,
     COUNT(DISTINCT visit_id) unique_visit
   FROM clique_bait.events
   GROUP BY month_no
   ```

   **Result**

   <img src="https://github.com/user-attachments/assets/bc05ea45-123b-4773-8d59-169aed12fe8c" alt="Case Study #6: Clique Bait" width="300" height="200">

4. What is the number of events for each event type?
   ```sql
   SELECT event_type,
   COUNT(*) cnt_event
   FROM clique_bait.events
   GROUP BY event_type
   ORDER BY event_type
   ```

   **Result**

   <img src="https://github.com/user-attachments/assets/405976b9-1a85-40e4-b588-c316cb4493f5" alt="Case Study #6: Clique Bait" width="300" height="200">

5. What is the percentage of visits which have a purchase event?
   ```sql
   SELECT
     ei.event_name,
     ROUND(
       COUNT(DISTINCT visit_id)::decimal(10,2)/
       (SELECT COUNT(DISTINCT visit_id) FROM clique_bait.events)*100
     ,2) as purchase_percentage
   FROM clique_bait.events e
   JOIN clique_bait.event_identifier ei ON e.event_type = ei.event_type
   WHERE event_name = 'Purchase'
   GROUP BY ei.event_name
   ```

   **Result**
   
   <img src="https://github.com/user-attachments/assets/2a96528d-7ffe-4319-ae46-cb36790937c9" alt="Case Study #6: Clique Bait" width="350" height="80">

6. What is the percentage of visits which view the checkout page but do not have a purchase event?
   ```sql
   WITH no_purchase as (
   SELECT
     e.visit_id,
     SUM(CASE WHEN ei.event_name = 'Purchase' THEN 1 ELSE 0 END) purchase,
     SUM(CASE WHEN ph.page_name = 'Checkout' THEN 1 ELSE 0 END) checkout
   FROM clique_bait.events e
   JOIN clique_bait.event_identifier ei ON e.event_type = ei.event_type
   JOIN clique_bait.page_hierarchy ph ON ph.page_id = e.page_id
   GROUP BY e.visit_id
   HAVING SUM(CASE WHEN ei.event_name = 'Purchase' THEN 1 ELSE 0 END) = 0
   AND SUM(CASE WHEN ph.page_name = 'Checkout' THEN 1 ELSE 0 END) > 0
   )
   SELECT
     ROUND(
       COUNT(*)::decimal(10,2)/
       (SELECT COUNT(DISTINCT visit_id) FROM clique_bait.events)*100
     ,2) as no_purchase_perc
   FROM no_purchase
   ```

   **Result**
   
   <img src="https://github.com/user-attachments/assets/c27981cc-b7c5-4105-85c8-5c61cd225ad5" alt="Case Study #6: Clique Bait" width="200" height="80">

7. What are the top 3 pages by number of views?
   ```sql
   SELECT
     ph.page_name,
     COUNT(*) cnt_view
   FROM clique_bait.events e
   JOIN clique_bait.event_identifier ei ON e.event_type = ei.event_type
   JOIN clique_bait.page_hierarchy ph ON e.page_id = ph.page_id
   GROUP BY ph.page_name
   ORDER BY cnt_view DESC
   LIMIT 3
   ```

   **Result**
   
   <img src="https://github.com/user-attachments/assets/f1b0d498-b584-4f49-beb3-6de340ea9090" alt="Case Study #6: Clique Bait" width="300" height="120">

8. What is the number of views and cart adds for each product category?
   ```sql
   SELECT
     ph.product_category,
     SUM(CASE WHEN ei.event_name = 'Page View' THEN 1 ELSE 0 END) cnt_view,
     SUM(CASE WHEN ei.event_name = 'Add to Cart' THEN 1 ELSE 0 END) cnt_addtocart
   FROM clique_bait.events e
   JOIN clique_bait.event_identifier ei ON e.event_type = ei.event_type
   JOIN clique_bait.page_hierarchy ph ON e.page_id = ph.page_id
   GROUP BY ph.product_category
   ```

   **Result**
   
   <img src="https://github.com/user-attachments/assets/9b0fb7f8-7a52-4ee8-94b5-817e22418b82" alt="Case Study #6: Clique Bait" width="400" height="150">

9. What are the top 3 products by purchases?
    ```sql
    WITH addtocart AS (
    SELECT
      e.visit_id,
      ph.page_name,
      1 cnt_addtocart
    FROM clique_bait.events e
    JOIN clique_bait.event_identifier ei ON e.event_type = ei.event_type
    JOIN clique_bait.page_hierarchy ph ON e.page_id = ph.page_id
    WHERE ei.event_name = 'Add to Cart'
    ),
    purchase AS (
    SELECT
      e.visit_id,
      1 cnt_purchase
    FROM clique_bait.events e
    JOIN clique_bait.event_identifier ei ON e.event_type = ei.event_type
    JOIN clique_bait.page_hierarchy ph ON e.page_id = ph.page_id
    WHERE ei.event_name = 'Purchase'
    )
    SELECT
      page_name,
      COUNT(*) cnt_purchased_item
    FROM addtocart atc
    JOIN purchase p ON atc.visit_id = p.visit_id
    GROUP BY page_name
    ORDER BY cnt_purchased_item DESC
    LIMIT 3
    ```

    **Result**

   <img src="https://github.com/user-attachments/assets/28048fe7-3532-489d-9d76-0958212d75f7" alt="Case Study #6: Clique Bait" width="400" height="150">

### Product Funnel Analysis
Using a single SQL query - create a new output table which has the following details:

How many times was each product viewed?
How many times was each product added to cart?
How many times was each product added to a cart but not purchased (abandoned)?
How many times was each product purchased?
Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.

Use your 2 new output tables - answer the following questions:

Which product had the most views, cart adds and purchases?
Which product was most likely to be abandoned?
Which product had the highest view to purchase percentage?
What is the average conversion rate from view to cart add?
What is the average conversion rate from cart add to purchase?

### Campaigns Analysis
Generate a table that has 1 single row for every unique visit_id record and has the following columns:

user_id
visit_id
visit_start_time: the earliest event_time for each visit
page_views: count of page views for each visit
cart_adds: count of product cart add events for each visit
purchase: 1/0 flag if a purchase event exists for each visit
campaign_name: map the visit to a campaign if the visit_start_time falls between the start_date and end_date
impression: count of ad impressions for each visit
click: count of ad clicks for each visit
(Optional column) cart_products: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the sequence_number)
Use the subsequent dataset to generate at least 5 insights for the Clique Bait team - bonus: prepare a single A4 infographic that the team can use for their management reporting sessions, be sure to emphasise the most important points from your findings.

Some ideas you might want to investigate further include:

Identifying users who have received impressions during each campaign period and comparing each metric with other users who did not have an impression event
Does clicking on an impression lead to higher purchase rates?
What is the uplift in purchase rate when comparing users who click on a campaign impression versus users who do not receive an impression? What if we compare them with users who just an impression but do not click?
What metrics can you use to quantify the success or failure of each campaign compared to eachother?
