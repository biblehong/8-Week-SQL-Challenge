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
   WHERE ph.product_id IS NOT NULL
   GROUP BY ph.product_category
   ```

   **Result**
   
   <img src="https://github.com/user-attachments/assets/034d2a73-79fa-43cd-9b32-caba35ae21e5" alt="Case Study #6: Clique Bait" width="400" height="120">

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
    AND ph.product_id IS NOT NULL
    ),
    purchase AS (
    SELECT
      e.visit_id,
      1 cnt_purchase
    FROM clique_bait.events e
    JOIN clique_bait.event_identifier ei ON e.event_type = ei.event_type
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

- How many times was each product viewed?
- How many times was each product added to cart?
- How many times was each product added to a cart but not purchased (abandoned)?
- How many times was each product purchased?
  ```sql
  WITH activity AS (
  SELECT
    e.visit_id,
    ph.page_name,
    ei.event_name,
    CASE WHEN ei.event_name = 'Page View' THEN 1 ELSE 0 END AS cnt_view,
    CASE WHEN ei.event_name = 'Add to Cart' THEN 1 ELSE 0 END AS cnt_addtocart
  FROM clique_bait.events e
  JOIN clique_bait.event_identifier ei ON e.event_type = ei.event_type
  JOIN clique_bait.page_hierarchy ph ON e.page_id = ph.page_id
  WHERE ei.event_name IN ('Page View','Add to Cart')
  AND ph.product_id IS NOT NULL
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
    a.page_name AS product,
    SUM(a.cnt_view) AS cnt_view,
    SUM(a.cnt_addtocart) AS cnt_addtocart,
    SUM(CASE WHEN a.cnt_addtocart = 1 AND p.cnt_purchase IS NULL THEN 1 ELSE 0 END) AS cnt_abandoned,
    SUM(COALESCE(p.cnt_purchase,0)) AS cnt_purchased_item
  FROM activity a
  LEFT JOIN purchase p ON a.visit_id = p.visit_id AND a.event_name = 'Add to Cart'
  GROUP BY product
  ORDER BY product
  ```

  **Result**

  <img src="https://github.com/user-attachments/assets/099da856-419b-457d-a114-7dd077cb75e9" alt="Case Study #6: Clique Bait" width="600" height="250">

Additionally, create another table which further aggregates the data for the above points but this time for each product category instead of individual products.
```sql
WITH activity AS (
SELECT
  e.visit_id,
  ph.product_category,
  ei.event_name,
  CASE WHEN ei.event_name = 'Page View' THEN 1 ELSE 0 END AS cnt_view,
  CASE WHEN ei.event_name = 'Add to Cart' THEN 1 ELSE 0 END AS cnt_addtocart
FROM clique_bait.events e
JOIN clique_bait.event_identifier ei ON e.event_type = ei.event_type
JOIN clique_bait.page_hierarchy ph ON e.page_id = ph.page_id
WHERE ei.event_name IN ('Page View','Add to Cart')
AND ph.product_id IS NOT NULL
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
  a.product_category,
  SUM(a.cnt_view) AS cnt_view,
  SUM(a.cnt_addtocart) AS cnt_addtocart,
  SUM(CASE WHEN a.cnt_addtocart = 1 AND p.cnt_purchase IS NULL THEN 1 ELSE 0 END) AS cnt_abandoned,
  SUM(COALESCE(p.cnt_purchase,0)) AS cnt_purchased_item
FROM activity a
LEFT JOIN purchase p ON a.visit_id = p.visit_id AND a.event_name = 'Add to Cart'
GROUP BY product_category
ORDER BY product_category
```

**Result**

<img src="https://github.com/user-attachments/assets/13459a38-343b-463a-874e-35069259d522" alt="Case Study #6: Clique Bait" width="600" height="120">

Use your 2 new output tables - answer the following questions:

1. Which product had the most views, cart adds and purchases?

- Oyster was the most viewed item while Lobster was the most added to cart and purchased product.
  
2. Which product was most likely to be abandoned?

- Russian Caviar was the most abandoned product.
  
3. Which product had the highest view to purchase percentage?
   ```sql
   /*using the same activity and purchase CTEs above, add the following to get the final output*/
   SELECT
     a.page_name as product,
     ROUND(SUM(COALESCE(p.cnt_purchase,0))::decimal(10,2)/SUM(a.cnt_view)*100,2) as view_to_purchase
   FROM activity a
   LEFT JOIN purchase p ON a.visit_id = p.visit_id AND a.event_name = 'Add to Cart'
   GROUP BY product
   ORDER BY view_to_purchase DESC
   LIMIT 1
   ```

   **Result**

   <img src="https://github.com/user-attachments/assets/27196a60-1dc9-4b3a-8bc6-be78638a4d4a" alt="Case Study #6: Clique Bait" width="350" height="80">
 
4. What is the average conversion rate from view to cart add?
   ```sql
   /*using the same activity CTE above, add the following to get the final output*/
   conversion_rate AS (
   SELECT
     a.page_name as product,
     SUM(a.cnt_addtocart)::decimal(10,2)/SUM(a.cnt_view)*100 as view_to_addtocart
   FROM activity a
   GROUP BY product
   )
   SELECT
   ROUND(AVG(view_to_addtocart),2) avg_conversion_rate
   FROM conversion_rate
   ```

   **Result**

   <img src="https://github.com/user-attachments/assets/0d3255f1-788e-474a-aee4-24f74aa0d9a3" alt="Case Study #6: Clique Bait" width="200" height="80">
 
5. What is the average conversion rate from cart add to purchase?
    ```sql
   /*using the same activity and purchase CTEs above, add the following to get the final output*/
    conversion_rate AS (
    SELECT
      a.page_name as product,
      SUM(p.cnt_purchase)::decimal(10,2)/SUM(a.cnt_addtocart)*100 as addtocart_to_purchase
    FROM activity a
    LEFT JOIN purchase p ON a.visit_id = p.visit_id AND a.event_name = 'Add to Cart'
    GROUP BY product
    )
    SELECT
    ROUND(AVG(addtocart_to_purchase),2) avg_conversion_rate
    FROM conversion_rate
    ```

    **Result**

   <img src="https://github.com/user-attachments/assets/7c324d38-e924-4556-b235-6ad7ac65bb22" alt="Case Study #6: Clique Bait" width="200" height="80">
   
### Campaigns Analysis
Generate a table that has 1 single row for every unique visit_id record and has the following columns:

- `user_id`
- `visit_id`
- `visit_start_time`: the earliest `event_time` for each visit
- `page_views`: count of page views for each visit
- `cart_adds`: count of product cart add events for each visit
- `purchase`: 1/0 flag if a purchase event exists for each visit
- `campaign_name`: map the visit to a campaign if the `visit_start_time` falls between the `start_date` and `end_date`
- `impression`: count of ad impressions for each visit
- `click`: count of ad clicks for each visit
- (Optional column) `cart_products`: a comma separated text value with products added to the cart sorted by the order they were added to the cart (hint: use the `sequence_number`)
  ```sql
  SELECT
    u.user_id,
	  e.visit_id,
	  MIN(e.event_time) as visit_start_time,
	  SUM(CASE WHEN ei.event_name = 'Page View' THEN 1 ELSE 0 END) page_views,
	  SUM(CASE WHEN ei.event_name = 'Add to Cart' THEN 1 ELSE 0 END) cart_adds,
	  SUM(CASE WHEN ei.event_name = 'Purchase' THEN 1 ELSE 0 END) purchase,
	  COALESCE(
      (
      SELECT campaign_name
      FROM clique_bait.campaign_identifier
      WHERE start_date <= MIN(e.event_time)
      AND end_date >= MIN(e.event_time)
      )
    ,'N/A') campaign_name,
	  SUM(CASE WHEN ei.event_name = 'Ad Impression' THEN 1 ELSE 0 END) impression,
	  SUM(CASE WHEN ei.event_name = 'Ad Click' THEN 1 ELSE 0 END) click,
	  COALESCE(
      STRING_AGG(
        CASE WHEN ei.event_name = 'Add to Cart' THEN ph.page_name
        ELSE NULL
        END
      ,', ' ORDER BY e.sequence_number)
    ,'N/A') cart_products
  FROM clique_bait.events e
  JOIN clique_bait.users u ON e.cookie_id = u.cookie_id
  JOIN clique_bait.event_identifier ei ON e.event_type = ei.event_type
  JOIN clique_bait.page_hierarchy ph ON e.page_id = ph.page_id
  GROUP BY u.user_id, e.visit_id
  ORDER BY e.visit_id
  ```

  **Result**
  
  <img src="https://github.com/user-attachments/assets/7051d36a-1707-443a-a775-334a6a60622e" alt="Case Study #6: Clique Bait" width="1200" height="230">

## Conclusion

February had the most number of visits in the history of Clique Bait online store. About 49.86% of the total visits resulted into a purchase, which was quite low and may need some digging up to see if there's any area Danny can improve to increase sales. 

Shellfish was the most popular category that customers viewed, added to cart and purchased, with Lobster being the most bought item among all the products. Russian Caviar was the most abandoned item despite having a high view and add to cart numbers. This could mean that customers were changing their mind mid-purchase which can be due to item price or item quantity. Danny should start thinking about running campaigns on least favorite items to boost their sales. 

About 61% of those who viewed items on the website subsequently added the items to the cart and about 76% of these were converted into a sale. 

Additional Insights:

<img src="https://github.com/user-attachments/assets/d5996d1c-1914-45ad-98f3-7e3abd64221e" alt="Case Study #6: Clique Bait" width="400" height="180">

The image shows that about 75% of the total visits did not start from an ad impression. 29% of the total visits that resulted in a purchase also did not start from an ad impression. This could mean that majority of repeated customers, referrals or new customers were going directly to the website. However, if we observe only those who have received an ad impression, they had a higher chance of converting into a sale, especially those who clicked the ad. This could prove that the ads were effectively attracting new customers based on the ad marketing content. 

In summary, the visits that were from impressions were likely to convert into a sale rather than those that were going directly to the website. Marketing team has done a good job attracting new customers with their marketing ad, resulting in a strong conversion rate in this area.

<img src="https://github.com/user-attachments/assets/49789251-125b-4d2e-ae0b-def703f8643e" alt="Case Study #6: Clique Bait" width="400" height="220">

The half off campaign on shellfish saw the most visits that ended in a sale in all the campaign periods, proving once again that shellfish is the most popular category in the online store. However, if we look closely at each campaign's conversion rate, the 3 campaigns displayed relatively close rates with the 25% off campaign taking the win at 50%. A 50% conversion rate during a campaign period seems rather low seeing that the no campaign period resulted in a 52% conversion rate. 

Marketing team may have done a good job at attracting customers with their marketing ad but on campaign period perspective, it seems there is a need to revisit the terms and other aspects such as system issue that may result to payment failure, or purchase terms such as 25% off only on selected items etc. in order to improve the campaign strategy of each category. In addition, a campaign that would promote cross selling and leverage the popularity of shellfish may boost the sales of other items in other categories (e.g. Buy 1 Salmon, Get 1 Oyster for 20% Off).
