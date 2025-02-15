# Case Study #7 : Balanced Tree Clothing Co.

<img src="https://github.com/user-attachments/assets/f8730253-aac0-4280-859c-f01e34ae7291" alt="Case Study #7: Balanced Tree Clothing Co." width="300" height="300">

> [!NOTE]
> Dataset and images are all sourced from [#8WeekSQLChallenge](https://8weeksqlchallenge.com/)

## Table of Contents
- [Introduction](#introduction)
- [Table Relationship](#table-relationship)
- [Business Questions](#business-questions)
  - [High Level Sales Analysis](#high-level-sales-analysis)
  - [Transaction Analysis](#transaction-analysis)
  - [Product Analysis](#product-analysis)
- [Conclusion](#conclusion)

## Introduction

Danny, Balanced Tree Clothing's CEO, is requesting assistance for their merchandising team to analyze their sales performance and generate a basic financial report to share to a wider business.

## Table Relationship

<img src="https://github.com/user-attachments/assets/beab9bb1-7f44-48a9-a777-ad8789f2f96b" alt="Case Study #7: Balanced Tree Clothing Co." width="1050" height="400">

## Business Questions
### High Level Sales Analysis
1. What was the total quantity sold for all products?
   ```sql
   SELECT
     SUM(qty) total_qty
   FROM balanced_tree.sales
   ```

   **Result**
   
   ![image](https://github.com/user-attachments/assets/53c92d88-3eb6-4d65-8270-0ddf350aeab9)

2. What is the total generated revenue for all products before discounts?
   ```sql
   SELECT
     SUM(s.price*s.qty) total_revenue
   FROM balanced_tree.sales s
   ```

   **Result**
   
   ![image](https://github.com/user-attachments/assets/bfa3fa84-7d8f-477f-af41-0230c5aee2fc)

3. What was the total discount amount for all products?
   ```sql
   SELECT
     SUM(s.price*s.qty*discount/100) total_discount
   FROM balanced_tree.sales s
   ```

   **Result**
   
   ![image](https://github.com/user-attachments/assets/613a32c0-664b-49ae-b68e-2afd7e042de6)

### Transaction Analysis
1. How many unique transactions were there?
   ```sql
   SELECT
     COUNT(DISTINCT txn_id) cnt_txn
   FROM balanced_tree.sales s
   ```

   **Result**

   ![image](https://github.com/user-attachments/assets/23e5b1ee-5c53-4035-a603-fe351398ce94)

2. What is the average unique products purchased in each transaction?
   ```sql
   WITH txn AS (
   SELECT
     txn_id,
     COUNT(DISTINCT prod_id) unique_prod
   FROM balanced_tree.sales s
   GROUP BY txn_id
   )
   SELECT
     ROUND(AVG(unique_prod),0) avg_product
   FROM txn
   ```

   **Result**
   
   ![image](https://github.com/user-attachments/assets/6c455ae2-3a75-4bc7-a671-7268ebfc05fd)


3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?
   ```sql
   WITH txn AS (
   SELECT
     txn_id,
     SUM(s.qty*s.price) as revenue
   FROM balanced_tree.sales s
   GROUP BY txn_id
   )
   SELECT
     PERCENTILE_CONT(0.25) WITHIN GROUP(ORDER BY revenue) "25th_percentile",
     PERCENTILE_CONT(0.50) WITHIN GROUP(ORDER BY revenue) "50th_percentile",
     PERCENTILE_CONT(0.75) WITHIN GROUP(ORDER BY revenue) "75th_percentile"
   FROM txn
   ```

   **Result**

   ![image](https://github.com/user-attachments/assets/3e1bda7a-9294-48fd-81b8-a7e56aaf1bad)

4. What is the average discount value per transaction?
   ```sql
   SELECT
     txn_id,
     ROUND(AVG(s.qty*s.price*s.discount/100),2) as avg_discount
   FROM balanced_tree.sales s
   GROUP BY txn_id
   ```

   **Result**
   
   ![image](https://github.com/user-attachments/assets/10937db4-fa61-4351-b851-593786349fcc)


5. What is the percentage split of all transactions for members vs non-members?
   ```sql
   SELECT
     member,
     COUNT(DISTINCT txn_id) cnt_txn,
     ROUND(COUNT(DISTINCT txn_id)/SUM(COUNT(DISTINCT txn_id)) OVER()*100,2) AS percentage_split
   FROM balanced_tree.sales s
   GROUP BY member
   ```

   **Result**
   
   ![image](https://github.com/user-attachments/assets/1f8892f3-9770-4a0e-8357-74df0420c69b)

6. What is the average revenue for member transactions and non-member transactions?
   ```sql
   WITH txn AS (
   SELECT
     member,
     txn_id,
     ROUND(SUM(s.qty*s.price),2) revenue
   FROM balanced_tree.sales s
   GROUP BY member, txn_id
   )
   SELECT
     member,
     ROUND(AVG(revenue),2) avg_revenue
   FROM txn
   GROUP BY member
   ```

   **Result**
   
   ![image](https://github.com/user-attachments/assets/933a1f79-3060-4240-ba29-63f16eb3a648)

### Product Analysis
1. What are the top 3 products by total revenue before discount?
   ```sql
   SELECT
     p.product_name,
     SUM(s.qty*s.price) total_revenue
   FROM balanced_tree.sales s
   JOIN balanced_tree.product_details p ON s.prod_id = p.product_id
   GROUP BY p.product_name
   ORDER BY total_revenue DESC
   LIMIT 3
   ```

   **Result**

   ![image](https://github.com/user-attachments/assets/246f428b-e06c-4f2b-b9c1-d34f1333a75e)

2. What is the total quantity, revenue and discount for each segment?
   ```sql
   SELECT
     p.segment_name,
     SUM(qty) total_qty,
     SUM(s.qty*s.price) total_revenue,
     SUM(s.qty*s.price*s.discount/100) total_discount
   FROM balanced_tree.sales s
   JOIN balanced_tree.product_details p ON s.prod_id = p.product_id
   GROUP BY p.segment_name
   ```

   **Result**

   ![image](https://github.com/user-attachments/assets/88324266-68b0-43be-a6c5-c3af03a62789)
   
3. What is the top selling product for each segment?
   ```sql
   WITH segment AS (
     SELECT
       p.segment_name,
       p.product_name,
       SUM(s.qty*s.price) total_revenue,
       ROW_NUMBER() OVER(PARTITION BY p.segment_name ORDER BY SUM(s.qty*s.price) DESC) rnk
     FROM balanced_tree.sales s
     JOIN balanced_tree.product_details p ON s.prod_id = p.product_id
     GROUP BY p.segment_name, p.product_name
   )
   SELECT
     segment_name,
     product_name,
     total_revenue
   FROM segment
   WHERE rnk = 1
   ```

   **Result**
   
   ![image](https://github.com/user-attachments/assets/72defb34-1063-4b45-bc30-1e26faf0d592)

4. What is the total quantity, revenue and discount for each category?
   ```sql
   SELECT
     p.category_name,
     SUM(qty) total_qty,
     SUM(s.qty*s.price) total_revenue,
     SUM(s.qty*s.price*s.discount/100) total_discount
   FROM balanced_tree.sales s
   JOIN balanced_tree.product_details p ON s.prod_id = p.product_id
   GROUP BY p.category_name
   ```

   **Result**
   
   ![image](https://github.com/user-attachments/assets/d9abe79a-0926-4c4e-9165-7203daba4890)

5. What is the top selling product for each category?
   ```sql
   WITH category AS (
     SELECT
       p.category_name,
       p.product_name,
       SUM(s.qty*s.price) total_revenue,
       ROW_NUMBER() OVER(PARTITION BY p.category_name ORDER BY SUM(s.qty*s.price) DESC) rnk
     FROM balanced_tree.sales s
     JOIN balanced_tree.product_details p ON s.prod_id = p.product_id
     GROUP BY p.category_name, p.product_name
   )
   SELECT
     category_name,
     product_name,
     total_revenue
   FROM category
   WHERE rnk = 1
   ```

   **Result**
   
   ![image](https://github.com/user-attachments/assets/f45cd431-b75e-4a0a-bb74-dfca33321629)

6. What is the percentage split of revenue by product for each segment?
   ```sql
   SELECT
     p.segment_name,
	   p.product_name,
     SUM(s.qty*s.price) total_revenue,
     ROUND(
       SUM(s.qty*s.price)/
       SUM(SUM(s.qty*s.price)) OVER(PARTITION BY p.segment_name)*100
     ,2) AS percentage_split
   FROM balanced_tree.sales s
	JOIN balanced_tree.product_details p ON s.prod_id = p.product_id
   GROUP BY p.segment_name, p.product_name
   ```

   **Result**
   
   ![image](https://github.com/user-attachments/assets/7bb97910-ed54-47aa-b3c8-ccb15c655958)

7. What is the percentage split of revenue by segment for each category?
   ```sql
   SELECT
     p.category_name,
     p.segment_name,
     SUM(s.qty*s.price) total_revenue,
     ROUND(
       SUM(s.qty*s.price)/
       SUM(SUM(s.qty*s.price)) OVER(PARTITION BY p.category_name)*100
     ,2) AS percentage_split
   FROM balanced_tree.sales s
	JOIN balanced_tree.product_details p ON s.prod_id = p.product_id
   GROUP BY p.category_name, p.segment_name
   ```

   **Result**
   
   ![image](https://github.com/user-attachments/assets/e44ee99b-db53-41ee-b4df-4f5b3aa85cd3)

8. What is the percentage split of total revenue by category?
   ```sql
   SELECT
     p.category_name,
     SUM(s.qty*s.price) total_revenue,
     ROUND(
       SUM(s.qty*s.price)/
       SUM(SUM(s.qty*s.price)) OVER()*100
     ,2) AS percentage_split
   FROM balanced_tree.sales s
	JOIN balanced_tree.product_details p ON s.prod_id = p.product_id
   GROUP BY p.category_name
   ```

   **Result**

   ![image](https://github.com/user-attachments/assets/1f6e7893-db2b-444b-8f98-58b6b7dbf565)

9. What is the total transaction “penetration” for each product? (hint: penetration = number of transactions where at least 1 quantity of a product was purchased divided by total number of transactions)
    ```sql
    SELECT
      p.product_name,
      COUNT(DISTINCT txn_id) cnt_txn,
      ROUND(
        COUNT(DISTINCT txn_id)::decimal(10,2)/
        (SELECT COUNT(DISTINCT txn_id) FROM balanced_tree.sales)*100
      ,2) penetration
    FROM balanced_tree.sales s
    JOIN balanced_tree.product_details p ON s.prod_id = p.product_id
    GROUP BY p.product_name
    ```

    **Result**
   
   ![image](https://github.com/user-attachments/assets/28a20d93-0b88-4636-b1a7-408cdb7b6fa5)
   
10. What is the most common combination of at least 1 quantity of any 3 products in a 1 single transaction?
    ```sql
    WITH txn AS (
    SELECT
      s.txn_id,
      STRING_AGG(p.product_name,', ' ORDER BY p.product_id) AS "3_prod_sales"
    FROM balanced_tree.sales s
    JOIN balanced_tree.product_details p ON s.prod_id = p.product_id
    GROUP by txn_id
    HAVING COUNT(DISTINCT p.product_id) = 3
    )
    SELECT "3_prod_sales", COUNT(txn_id) cnt_txn
    FROM txn
    GROUP BY "3_prod_sales"
    ORDER BY cnt_txn DESC
    ```

    **Result**

    ![image](https://github.com/user-attachments/assets/509e54c3-35c2-4f40-91af-1a2f234a32c6)

### Bonus Challenge
Use a single SQL query to transform the product_hierarchy and product_prices datasets to the product_details table.

Hint: you may want to consider using a recursive CTE to solve this problem!
