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
  - [Data Allocation Challenge](#data-allocation-challenge)
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
2. What is the average total historical deposit counts and amounts for all customers?
3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
4. What is the closing balance for each customer at the end of the month?
5. What is the percentage of customers who increase their closing balance by more than 5%?

### Data Allocation Challenge
To test out a few different hypotheses - the Data Bank team wants to run an experiment where different groups of customers would be allocated data using 3 different options:

Option 1: data is allocated based off the amount of money at the end of the previous month
Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days
Option 3: data is updated real-time
For this multi-part challenge question - you have been requested to generate the following data elements to help the Data Bank team estimate how much data will need to be provisioned for each option:

running customer balance column that includes the impact each transaction
customer balance at the end of each month
minimum, average and maximum values of the running balance for each customer
Using all of the data available - how much data would have been required for each option on a monthly basis?
