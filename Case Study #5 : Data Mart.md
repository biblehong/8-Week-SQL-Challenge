# Case Study #5 : Data Mart

<img src="https://github.com/user-attachments/assets/698c8ff2-1326-4c4d-b0c5-9f494fed32a5" alt="Case Study #5: Data Mart" width="300" height="300">

> [!NOTE]
> Dataset and images are all sourced from [#8WeekSQLChallenge](https://8weeksqlchallenge.com/)

## Table of Contents
- [Introduction](#introduction)
- [Table Relationship](#table-relationship)
- [Business Questions](#business-questions)
  - [Data Cleansing Steps](#data-cleansing-steps)
  - [Data Exploration](#data-exploration)
  - [Before & After Analysis](#before--after-analysis)
- [Conclusion](#conclusion)

## Introduction
Data Mart is Danny's latest venture and after running international operations for his online supermarket, Danny is asking for support to analyze his sales performance. 

There was a large scale supply change made at Data Mart in June 2020. All Data Mart products now use sustainable packaging methods in every single step from the farm all the way to the customer.

Danny needs your help to quantify the impact of this change on the sales performance for Data Mart and it’s separate business areas.

## Table Relationship

<img src="https://github.com/user-attachments/assets/0b5848bc-b812-4eb0-99bf-87f5a2ae44eb" alt="Case Study #5: Data Mart" width="230" height="250">

## Business Questions
### Data Cleansing Steps
In a single query, perform the following operations and generate a new table in the data_mart schema named clean_weekly_sales:
- Convert the week_date to a DATE format
- Add a week_number as the second column for each week_date value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
- Add a month_number with the calendar month for each week_date value as the 3rd column
- Add a calendar_year column as the 4th column containing either 2018, 2019 or 2020 values
- Add a new column called age_band after the original segment column using the following mapping on the number inside the segment value

  |segment|age_band    |
  |-------|------------|
  |1   	  |Young Adults|
  |2	    |Middle Aged |
  |3 or 4	|Retirees    |
  
- Add a new demographic column using the following mapping for the first letter in the segment values:

  |segment|demographic|
  |-------|-----------|
  |C	    |Couples    |
  |F	    |Families   |

- Ensure all null string values with an "unknown" string value in the original segment column as well as the new age_band and demographic columns
- Generate a new avg_transaction column as the sales value divided by transactions rounded to 2 decimal places for each record
  ```sql
  CREATE TABLE clean_weekly_sales AS
  SELECT 
	  TO_DATE(week_date,'DD/MM/YY') week_date, 
	  CEIL(DATE_PART('day',TO_DATE(week_date,'DD/MM/YY'))/7) week_number,
	  DATE_PART('month',TO_DATE(week_date,'DD/MM/YY')) month_number, 
	  DATE_PART('year',TO_DATE(week_date,'DD/MM/YY')) calendar_year, 
	  region,
	  platform,
	  CASE WHEN segment = 'null' THEN 'Unknown'
      ELSE segment
      END as segment,
	  CASE WHEN segment = 'null' THEN 'Unknown'
		  WHEN RIGHT(segment,1) = '1' THEN 'Young Adults'
		  WHEN RIGHT(segment,1) = '2' THEN 'Middle Aged'
		  WHEN RIGHT(segment,1) IN ('3','4') THEN 'Retirees'
		  END as age_band,
	  CASE WHEN segment = 'null' THEN 'Unknown'
		  WHEN LEFT(segment,1) = 'C' THEN 'Couples'
		  WHEN LEFT(segment,1) = 'F' THEN 'Family'
		  END as demographics,
	  customer_type,
	  transactions,
	  sales,
	  ROUND(sales::decimal(10,2)/transactions,2) as avg_transactions
  FROM data_mart.weekly_sales
  ```

  **Result**
  
  <img src="https://github.com/user-attachments/assets/3ce180f5-1ce4-43b3-bd64-6ef754722235" alt="Case Study #5: Data Mart" width="2000" height="260">

### Data Exploration
1. What day of the week is used for each week_date value?
   ```sql
   SELECT DISTINCT TO_CHAR(week_date,'Day') weekday FROM clean_weekly_sales
   ```

   **Result**
   <img src="https://github.com/user-attachments/assets/3ce180f5-1ce4-43b3-bd64-6ef754722235" alt="Case Study #5: Data Mart" width="2000" height="260">
   ![image](https://github.com/user-attachments/assets/87c0686a-2404-482d-beed-2585163af35f)

   - Monday is used as week_date

3. What range of week numbers are missing from the dataset?
   ```sql
   WITH dates AS (
   SELECT GENERATE_SERIES(1,52,1) dates
   )
   SELECT DISTINCT d.dates
   FROM dates d
   LEFT JOIN clean_weekly_sales c ON d.dates = c.week_number
   WHERE c.week_number IS NULL;
   ```

   **Result**
   <img src="https://github.com/user-attachments/assets/3ce180f5-1ce4-43b3-bd64-6ef754722235" alt="Case Study #5: Data Mart" width="2000" height="260">
   ![image](https://github.com/user-attachments/assets/94c35ee7-2307-46c8-9561-f4500b3dec88)

5. How many total transactions were there for each year in the dataset?
   ```sql
   SELECT calendar_year, SUM(transactions) txn_cnt FROM clean_weekly_sales GROUP BY calendar_year ORDER BY calendar_year
   ```

   **Result**
   <img src="https://github.com/user-attachments/assets/3ce180f5-1ce4-43b3-bd64-6ef754722235" alt="Case Study #5: Data Mart" width="2000" height="260">
   ![image](https://github.com/user-attachments/assets/d035d4f0-2bb8-401e-8ab5-83cf8f0fb754)


7. What is the total sales for each region for each month?
   ```sql
   SELECT region, month_number, SUM(sales) sales FROM clean_weekly_sales GROUP BY region, month_number ORDER BY region, month_number
   ```

   **Result**
   <img src="https://github.com/user-attachments/assets/3ce180f5-1ce4-43b3-bd64-6ef754722235" alt="Case Study #5: Data Mart" width="2000" height="260">
   ![image](https://github.com/user-attachments/assets/6d9b854f-97d7-4bcd-addf-12603f66609c)

9. What is the total count of transactions for each platform
   ```sql
   SELECT platform, SUM(transactions) txn_cnt FROM clean_weekly_sales GROUP BY platform ORDER BY platform
   ```

   **Result**
   <img src="https://github.com/user-attachments/assets/3ce180f5-1ce4-43b3-bd64-6ef754722235" alt="Case Study #5: Data Mart" width="2000" height="260">
   ![image](https://github.com/user-attachments/assets/263aa294-30d3-417b-a779-9f3c1b29d553)


11. What is the percentage of sales for Retail vs Shopify for each month?
   ```sql
   SELECT month_number, platform, ROUND(SUM(sales)/SUM(SUM(sales)) OVER(PARTITION BY month_number)*100,2) sales_percentage 
    FROM clean_weekly_sales GROUP BY month_number, platform ORDER BY month_number, platform
   ```

   **Result**
   <img src="https://github.com/user-attachments/assets/3ce180f5-1ce4-43b3-bd64-6ef754722235" alt="Case Study #5: Data Mart" width="2000" height="260">
   ![image](https://github.com/user-attachments/assets/b1d36805-8eb5-43b8-8f43-248e445e078b)

11. What is the percentage of sales by demographic for each year in the dataset?
   ```sql
   SELECT calendar_year,demographics, ROUND(SUM(sales)/SUM(SUM(sales)) OVER(PARTITION BY calendar_year)*100,2) sales_percentage 
    FROM clean_weekly_sales GROUP BY calendar_year,demographics ORDER BY calendar_year,demographics
   ```

   **Result**
   <img src="https://github.com/user-attachments/assets/3ce180f5-1ce4-43b3-bd64-6ef754722235" alt="Case Study #5: Data Mart" width="2000" height="260">
   ![image](https://github.com/user-attachments/assets/2f23b395-eef1-45ec-b384-98cfbf22c046)

11. Which age_band and demographic values contribute the most to Retail sales?
   ```sql
   SELECT age_band,demographics, SUM(sales) sales--/SUM(SUM(sales)) OVER(PARTITION BY calendar_year)*100,2) sales_percentage 
    FROM clean_weekly_sales WHERE platform = 'Retail' GROUP BY age_band,demographics ORDER BY sales desc
   ```

   **Result**
   <img src="https://github.com/user-attachments/assets/3ce180f5-1ce4-43b3-bd64-6ef754722235" alt="Case Study #5: Data Mart" width="2000" height="260">
   ![image](https://github.com/user-attachments/assets/12357d25-01b1-48c4-bc5f-36f2f188cca7)

11. Can we use the avg_transaction column to find the average transaction size for each year for Retail vs Shopify? If not - how would you calculate it instead?
    ```sql
    SELECT calendar_year, platform,ROUND(AVG(avg_transactions),0) avg_trans_row, SUM(sales)/SUM(transactions) as avg_trans_table
    FROM clean_weekly_sales
    GROUP BY calendar_year, platform
    ```

    **Result**
   <img src="https://github.com/user-attachments/assets/3ce180f5-1ce4-43b3-bd64-6ef754722235" alt="Case Study #5: Data Mart" width="2000" height="260">
    ![image](https://github.com/user-attachments/assets/9b791857-63fe-4978-81ce-9516ba221c3f)

### Before & After Analysis
This technique is usually used when we inspect an important event and want to inspect the impact before and after a certain point in time.

Taking the week_date value of 2020-06-15 as the baseline week where the Data Mart sustainable packaging changes came into effect.

We would include all week_date values for 2020-06-15 as the start of the period after the change and the previous week_date values would be before

Using this analysis approach - answer the following questions:

What is the total sales for the 4 weeks before and after 2020-06-15? What is the growth or reduction rate in actual values and percentage of sales?
What about the entire 12 weeks before and after?
How do the sale metrics for these 2 periods before and after compare with the previous years in 2018 and 2019?
