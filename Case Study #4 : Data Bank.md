# Case Study #4 : Data Bank

<img src="https://github.com/user-attachments/assets/8c93087d-08b6-4c31-9eba-6447f97e48e7" alt="Case Study #4: Data Bank" width="300" height="300">

> [!NOTE]
> Dataset and images are all sourced from [#8WeekSQLChallenge](https://8weeksqlchallenge.com/)

## Table of Contents
- [Introduction](#introduction)
- [Table Relationship](#table-relationship)
- [Business Questions](#business-questions)
  - [Customer Nodes Exploration](#customer-nodes-exploration)
  - [Customer Transactions](#customer-transactions)
- [Conclusion](#conclusion)

## Introduction
Danny is launching this new initiative to create a digital bank that goes beyond just banking activities. He wants to build a digital bank that has the world's most secure distributed data storage platform. Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts.  Thus, Data Bank was born!

Danny needs help needs help understanding the data, how to increase their customer database but also monitoring how much storage is needed. This case study is all about calculating metrics, growth and helping the business analyse their data in a smart way to better forecast and plan for their future developments!

## Table Relationship

<img src="https://github.com/user-attachments/assets/a66e78c6-de1e-4b0a-83a3-3a57da1b3a14" alt="Case Study #4: Data Bank" width="450" height="200">

## Business Questions
### Customer Nodes Exploration
1. How many unique nodes are there on the Data Bank system?
   ```sql
   SELECT COUNT(DISTINCT node_id) node_cnt
   FROM customer_nodes
   ```

   **Result**
   
   <img src="https://github.com/user-attachments/assets/e1f831ca-eef4-45cb-bd36-776921bb5388" alt="Case Study #4: Data Bank" width="150" height="80">
   
2. What is the number of nodes per region?
   ```sql
   SELECT
     r.region_name,
     COUNT(DISTINCT node_id) node_cnt
   FROM customer_nodes n
   JOIN regions r ON n.region_id = r.region_id
   GROUP by r.region_name
   ```

   **Result**
   
   <img src="https://github.com/user-attachments/assets/76bd072d-23c5-4076-b614-c9d97f9bbf93" alt="Case Study #4: Data Bank" width="250" height="150">

3. How many customers are allocated to each region?
   ```sql
   SELECT
     r.region_name,
     COUNT(DISTINCT customer_Id) customer_cnt
   FROM customer_nodes n
   JOIN regions r ON n.region_id = r.region_id
   GROUP by r.region_name
   ORDER BY r.region_name
   ```

   **Result**
   
   <img src="https://github.com/user-attachments/assets/9731d6a9-8306-4207-bf96-16e2b7a03daa" alt="Case Study #4: Data Bank" width="250" height="150">

4. How many days on average are customers reallocated to a different node?
   ```sql
   SELECT
     ROUND(AVG(end_date - start_date),2) AS node_chng
   FROM customer_nodes
   WHERE end_date <> '9999-12-31'
   ORDER BY node_chng desc
   ```

   **Result**
   
   <img src="https://github.com/user-attachments/assets/4af6355a-19c7-44c4-a746-9a11302431a9" alt="Case Study #4: Data Bank" width="150" height="80">
  
5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
   ```sql
   SELECT
     r.region_name,
     PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY end_date - start_date) AS median,
     PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY end_date - start_date) as "80th",
     PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY end_date - start_date) as "95th"
   FROM customer_nodes n
   JOIN regions r ON n.region_id = r.region_id
   WHERE end_date <> '9999-12-31'
   GROUP BY r.region_name
   ```

    **Result**
   
   <img src="https://github.com/user-attachments/assets/b03f407a-f08e-4562-8b66-bcd998f4f45a" alt="Case Study #4: Data Bank" width="450" height="150">

### Customer Transactions
1. What is the unique count and total amount for each transaction type?
   ```sql
   SELECT
     txn_type,
     COUNT(customer_id) txn_cnt,
     SUM(txn_amount) total_amt
   FROM data_bank.customer_transactions
   GROUP BY txn_type
   ```

   **Result**
   
   <img src="https://github.com/user-attachments/assets/f6275169-a3d6-421b-bc67-03ee51364650" alt="Case Study #4: Data Bank" width="330" height="100">

3. What is the average total historical deposit counts and amounts for all customers?
   ```sql
   WITH deposit AS(
   SELECT customer_id, COUNT(customer_id) txn_cnt, AVG(txn_amount) total_amt
   FROM data_bank.customer_transactions
   WHERE txn_type = 'deposit'
   GROUP BY customer_id
   )
   SELECT ROUND(AVG(txn_cnt),0) avg_txn, ROUND(AVG(total_amt),2) as avg_amt
   FROM deposit
   ```

   **Result**
   
    <img src="https://github.com/user-attachments/assets/a9e95e86-8a6a-4cfd-a933-fe6647ada19d" alt="Case Study #4: Data Bank" width="200" height="80">

5. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
   ```sql
    WITH txn AS (
    SELECT customer_id, 
    	DATE_TRUNC('month',txn_date)::date AS month,
    	SUM(CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END) deposit, 
    	SUM(CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END) purchase,
    	SUM(CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END) withdrawal
    	FROM data_bank.customer_transactions 
    	GROUP BY customer_id, month
    )
    SELECT month, COUNT(DISTINCT customer_id) customer_cnt
    FROM txn
    WHERE deposit > 1
    AND (purchase >= 1 OR withdrawal >=1)
    GROUP BY month
    ORDER BY month
   ```

   **Result**

   <img src="https://github.com/user-attachments/assets/8e4af283-1d4c-4430-802c-d1bdd2d3627f" alt="Case Study #4: Data Bank" width="250" height="150">

7. What is the closing balance for each customer at the end of the month?
   ```sql
   WITH txn AS (
   SELECT
     customer_id,
     (DATE_TRUNC('month',txn_date)+interval '1 month'-interval '1 day')::date as monthend_date,
     CASE WHEN txn_type = 'deposit' THEN txn_amount
       ELSE -txn_amount
       END as agg
   FROM data_bank.customer_transactions
   ),
   dates AS (
   SELECT * FROM
   (SELECT DISTINCT monthend_date	FROM txn) CROSS JOIN
   (SELECT DISTINCT customer_id FROM txn)
   )
   SELECT
     d.monthend_date,
     d.customer_id,
     SUM(COALESCE(t.agg,0)) AS total_agg,
     SUM(SUM(COALESCE(t.agg,0))) OVER (PARTITION BY d.customer_id ORDER BY d.monthend_date) AS balance
   FROM dates d
   LEFT JOIN txn t ON d.monthend_date = t.monthend_date AND d.customer_id = t.customer_id
   GROUP BY d.monthend_date, d.customer_id
   ORDER BY d.customer_id, d.monthend_date
   ```

   **Result**

   <img src="https://github.com/user-attachments/assets/db58f649-c334-4ed6-ad31-7697bdd5fb6a" alt="Case Study #4: Data Bank" width="380" height="450">

9. What is the percentage of customers who increase their closing balance by more than 5%?
    ```sql
    WITH txn AS (
    SELECT
      customer_id,
      (DATE_TRUNC('month',txn_date)+interval '1 month'-interval '1 day')::date as monthend_date,
      CASE WHEN txn_type = 'deposit' THEN txn_amount ELSE -txn_amount END as agg
    FROM data_bank.customer_transactions
    ),
    balance AS (
    SELECT
      t.monthend_date,
      t.customer_id,
      SUM(COALESCE(t.agg,0)) AS total_agg,
      SUM(
        SUM(COALESCE(t.agg,0))
      ) OVER (PARTITION BY t.customer_id ORDER BY t.monthend_date) AS ending_balance,
      ROW_NUMBER() OVER(PARTITION BY t.customer_id ORDER BY t.monthend_date) rnk1,
      ROW_NUMBER() OVER(PARTITION BY t.customer_id ORDER BY t.monthend_date DESC) rnk2
    FROM txn t
    GROUP BY t.monthend_date, t.customer_id
    ),
    opening_ending AS (
    SELECT
      *,
      LAG(ending_balance) OVER(PARTITION BY customer_id ORDER BY monthend_date) opening_balance
    FROM balance WHERE rnk1= 1 OR rnk2 = 1
    ),
    growth AS (
    SELECT
      customer_id,
      opening_balance,
      ending_balance,
      ROUND((ending_balance-opening_balance)/ABS(opening_balance)*100,2) as growth
    FROM opening_ending
    WHERE opening_balance IS NOT NULL
    )
    SELECT
      ROUND(
        SUM(
          CASE WHEN growth > 5 THEN 1
          ELSE 0
          END
        )::decimal(10,2)/
      COUNT(DISTINCT customer_id)*100,2) as percentage_cust
    FROM growth
    ```

    **Result**

   <img src="https://github.com/user-attachments/assets/61061466-8ba7-4b23-838f-43ef64936b51" alt="Case Study #4: Data Bank" width="150" height="70">

### Conclusion
There are 5 nodes that customers are being transferred to every 15 days on average. On average, customers make 5 transactions and deposits to the bank amounting to $508.61. About 33.4% of the whole customer base increased their initial deposit by 5% during their ending balance. This means that 66.6% have either made less than 5% or have gone more into debt (negatives!). Danny should review these customers to avoid hurtinig the bank's reserves any further.  
