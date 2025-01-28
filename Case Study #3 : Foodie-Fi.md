# Case Study #3 : Foodie-Fi

<img src="https://github.com/user-attachments/assets/9ffb9718-167b-40d6-a78f-4d225b6178a9" alt="Case Study #3: Foodie-Fi" width="300" height="300">

> [!NOTE]
> Dataset and images are all sourced from [#8WeekSQLChallenge](https://8weeksqlchallenge.com/)

## Table of Contents
- [Introduction](#introduction)
- [Table Relationship](#table-relationship)
- [Business Questions](#business-questions)
  - [Customer Journey](#customer-journey)
  - [Data Analysis Questions](#data-analysis-questions)
  - [Challenge Payment Question](#challenge-payment-question)
- [Conclusion](#conclusion)

## Introduction
Subscription based businesses are super popular and Danny wants to launch a new streaming service with only food-related content. Think Netflix but only cooking shows! 

He started selling monthly and annual subscriptions, giving customers unlimited access to food videos around the world. Danny wants to use this case study to understand the data and influence future decisions on investment and new features. 

## Table Relationship

<img src="https://github.com/user-attachments/assets/59bb5661-0265-428e-9aff-cf5278e97d59" alt="Case Study #3: Foodie-Fi" width="500" height="170">

## Business Questions
### Customer Journey
Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

#### Sample subscriptions table
<img src="https://github.com/user-attachments/assets/e52facd1-58d1-480c-bd20-1e6bd2a84049" alt="Case Study #3: Foodie-Fi" width="200" height="500">

```sql
SELECT s.customer_id,
  s.plan_id,
  s.start_date,
  p.plan_name
FROM subscriptions s 
JOIN plans p ON s.plan_id = p.plan_id
WHERE customer_id IN (1,2,11,13,15,16,18,19)
ORDER BY customer_id, start_date;
```

**Result**

<img src="https://github.com/user-attachments/assets/7af39f54-5138-44d2-b0c1-2b0f749bb676" alt="Case Study #3: Foodie-Fi" width="380" height="450">

- 1 out of 8 customers (Customer 11) or 12.5% of the total sample subscriptions decided not to continue the subscription after the trial period.
- 3 out of 8 customers (Customer 1, 13, 16) or 37.5% of the total sample subscriptions decided to downgrade and start with the basic monthly plan. 2 out of 3 customers were successfully converted to a pro plan. However, the start dates of these pro plans were way past the monthly renewal date therefore it is important to note that as per the upgrade policy, higher plans will automatically take effect which means that the customers paid for the basic plan and the pro plan in 1 month.

  <img src="https://github.com/user-attachments/assets/c52233fa-c31b-47eb-9722-48aae3e5eaff" alt="Case Study #3: Foodie-Fi" width="400" height="170">

- 3 out of 8 customers (Customer 2, 18, 19) or 37.5% of the total sample subscriptions were automatically entered in the pro monthly plan after their trial period. 1 out of 3 customers converted to an annual plan on the same day of the pro monthly plan's renewal date.
- 1 out of 8 customers (Customer 2) immediately upgraded his plan to pro annual after the trial period.
 
### Data Analysis Questions
1. How many customers has Foodie-Fi ever had?
   ```sql
   SELECT
     COUNT(DISTINCT customer_id) customer_cnt
   FROM foodie_fi.subscriptions
   ```

   **Result**

   <img src="https://github.com/user-attachments/assets/a301742c-849c-4485-955b-024424aa94c2" alt="Case Study #3: Foodie-Fi" width="130" height="60">

   - Foodie-Fi has a total of 1,000 customers since it started.

2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value
   ```sql
   SELECT
     DATE_TRUNC('month',start_date) month_start,
     COUNT(DISTINCT customer_id) customer_cnt
   FROM foodie_fi.subscriptions
   WHERE plan_id = 0
   GROUP BY month_start
   ORDER BY month_start
   ```

   **Result**
   
   <img src="https://github.com/user-attachments/assets/7a755cb3-83a5-4569-acb0-02bce9a13718" alt="Case Study #3: Foodie-Fi" width="250" height="280">

   - March had the highest number of trial subscriptions with February being the lowest (less number of days in February potentially affected this number)
     
3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name
   ```sql
   SELECT
     s.plan_id,
     p.plan_name,
     count(s.customer_id) customer_cnt
   FROM foodie_fi.subscriptions s
   JOIN foodie_fi.plans p ON s.plan_id = p.plan_id
   WHERE s.start_date >= '2021-01-01'
   GROUP BY s.plan_id, p.plan_name
   ORDER BY plan_id
   ```

   **Result**

   <img src="https://github.com/user-attachments/assets/9f9f8f32-36ec-4761-a979-3a3ab1ec1473" alt="Case Study #3: Foodie-Fi" width="280" height="100">
   
   - As observed in the result, there is a high churn count of customers after 2021. 

4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
   ```sql
   SELECT SUM(CASE WHEN p.plan_name = 'churn' THEN 1 ELSE 0 END) churn_customers,
   ROUND(
       SUM(
         CASE WHEN p.plan_name = 'churn' THEN 1
         ELSE 0 END)::decimal(10,2)
   /COUNT(DISTINCT customer_id)*100,1) as churn_rate
   FROM foodie_fi.subscriptions s
   JOIN foodie_fi.plans p ON s.plan_id = p.plan_id
   ```

   **Result**

   <img src="https://github.com/user-attachments/assets/c7995923-03f5-4ded-ae53-9e75fa73f0c9" alt="Case Study #3: Foodie-Fi" width="300" height="80">

   - 307 customers churned which is 30.7% of Foodie-Fi's customer database.
   
5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
    ```sql
    WITH churn AS(
    SELECT
      s.customer_id,
      p.plan_name,
      LEAD(p.plan_name) OVER(PARTITION BY customer_id ORDER BY start_date) next_plan
    FROM foodie_fi.subscriptions s
    JOIN foodie_fi.plans p ON s.plan_id = p.plan_id
    )
    SELECT
      SUM(
        CASE WHEN plan_name = 'trial' AND next_plan = 'churn' THEN 1
        ELSE 0 END) churn_after_trial,
    ROUND(
        SUM(
          CASE WHEN plan_name = 'trial' AND next_plan = 'churn'
          THEN 1 ELSE 0 END)::decimal(10,2)
      /COUNT(DISTINCT customer_id)*100,0) as churn_after_trial_percentage
    FROM churn
    ```

   **Result**

   <img src="https://github.com/user-attachments/assets/63fb9560-ad09-4ac7-a239-5edfdf5896dd" alt="Case Study #3: Foodie-Fi" width="300" height="60">

   - 92 customers churned right after their trial which is 9% of Foodie-Fi's customer database.
   
6. What is the number and percentage of customer plans after their initial free trial?
    ```sql
    WITH first_paid_plan AS (
    SELECT
      s.customer_id,
      p.plan_id,
      p.plan_name,
      ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY s.start_date) rownum
    FROM foodie_fi.subscriptions s
    JOIN foodie_fi.plans p ON s.plan_id = p.plan_id
    WHERE p.plan_name <> 'trial'
    )
    SELECT
      plan_id,
      plan_name,
      COUNT(customer_id) plan_cnt, 
    	ROUND(
        COUNT(customer_id)::decimal(10,2)/
        (
        SELECT
          COUNT(DISTINCT customer_id)
        FROM foodie_fi.subscriptions
        )
      *100,2) plan_percentage
    FROM first_paid_plan
    WHERE rownum = 1
    GROUP BY plan_id, plan_name
    ```

    **Result**

    <img src="https://github.com/user-attachments/assets/4419e8be-269e-4055-be91-4996476c7cdd" alt="Case Study #3: Foodie-Fi" width="380" height="110">

    - more than half of the customers (54.6%) chose to the basic plan for their first paid plan. This could indicate that customers are testing whether the basic plan can cover enough navigation on the streaming site before opting for a much higher subscription.

7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
    ```sql
    WITH plan AS (
    SELECT
      s.customer_id,
      p.plan_id,
      p.plan_name,
      ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY s.start_date DESC) rownum
    FROM foodie_fi.subscriptions s
    JOIN foodie_fi.plans p ON s.plan_id = p.plan_id
    WHERE start_date <= '2020-12-31'
    )
    SELECT
      plan_id,
      plan_name,
      COUNT(customer_id) plan_cnt, 
    	ROUND(
        COUNT(customer_id)::decimal(10,2)/
        (
        SELECT
          COUNT(DISTINCT customer_id)
        FROM foodie_fi.subscriptions
        )
      *100,2) plan_percentage
    FROM plan
    WHERE rownum = 1
    GROUP BY plan_id, plan_name
    ```

    **Result**

    <img src="https://github.com/user-attachments/assets/bf3509b3-abe4-4bec-a941-92a5354654cd" alt="Case Study #3: Foodie-Fi" width="380" height="130">

    - As of Dec 31, 2020, we can see a more even distribution on the subscription plans but with an alarming high churn rate of 23.6%

8. How many customers have upgraded to an annual plan in 2020?
    ```sql
    WITH annual_plan AS (
    SELECT
      s.customer_id,
      p.plan_id,
      p.plan_name,
      ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY s.start_date DESC) rownum
    FROM foodie_fi.subscriptions s
    JOIN foodie_fi.plans p ON s.plan_id = p.plan_id
    WHERE start_date <= '2020-12-31'
    )
    SELECT
      COUNT(customer_id) plan_cnt
    FROM annual_plan
    WHERE rownum = 1
    AND plan_name = 'pro annual'
    GROUP BY plan_id, plan_name
    ```

    **Result**
    
    <img src="https://github.com/user-attachments/assets/004d34b1-9b21-4b15-9c77-420d4da5e821" alt="Case Study #3: Foodie-Fi" width="150" height="80">

     - 195 customers upgraded to an annual plan in 2020. 

9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
    ```sql
    WITH avg_upgrade AS (
    SELECT
      s.customer_id,
      p.plan_id,
      p.plan_name,
      start_date,
      LEAD(s.start_date) OVER(PARTITION BY s.customer_id ORDER BY s.start_date) next_plan_start
    FROM foodie_fi.subscriptions s
    JOIN foodie_fi.plans p ON s.plan_id = p.plan_id
    WHERE p.plan_name IN ('trial','pro annual')
    ),
    filter AS ( 
    SELECT *
    FROM avg_upgrade
    WHERE next_plan_start IS NOT NULL
    )
    SELECT
      ROUND(AVG(next_plan_start - start_date),0) average_upgrade
    FROM filter
    ```

    **Result**
    
    <img src="https://github.com/user-attachments/assets/8d9a6379-977b-422d-8a3c-0e9553c55d3e" alt="Case Study #3: Foodie-Fi" width="180" height="80">

    - it takes an average of 105 days before a customer upgrades to an annual plan.
    
10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
    ```sql
    WITH avg_upgrade AS (
    SELECT
      s.customer_id,
      p.plan_id,
      p.plan_name,
      start_date,
      LEAD(s.start_date) OVER(PARTITION BY s.customer_id ORDER BY s.start_date) next_plan_start
    FROM foodie_fi.subscriptions s
    JOIN foodie_fi.plans p ON s.plan_id = p.plan_id
    WHERE p.plan_name IN ('trial','pro annual')
    ),
    bucket as (
    SELECT
      *,
      next_plan_start - start_date days_upgrade,
      WIDTH_BUCKET(next_plan_start - start_date,1,360,12) bucket_no
      FROM avg_upgrade
    WHERE next_plan_start IS NOT NULL
    )
    SELECT
      bucket_no,
      (bucket_no-1)*30+1 || '-' || bucket_no*30 || ' days' as bucket,
      ROUND(AVG(next_plan_start - start_date),0) average_days
    FROM bucket
    GROUP BY bucket_no, bucket
    ORDER BY bucket_no
    ```

    **Result**

    <img src="https://github.com/user-attachments/assets/c21853b4-aa2e-43a9-84c5-2281e2c30918" alt="Case Study #3: Foodie-Fi" width="280" height="300">

11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
    ```sql
    WITH downgrade AS (
    SELECT
      s.customer_id,
      p.plan_id,
      p.plan_name,
      LEAD(p.plan_name) OVER(PARTITION BY s.customer_id ORDER BY s.start_date) downgrade
    FROM foodie_fi.subscriptions s
    JOIN foodie_fi.plans p ON s.plan_id = p.plan_id
    WHERE p.plan_name IN ('pro monthly','basic monthly')
    AND date_part('year',start_date) = 2020
    ORDER BY s.customer_id
    )
    SELECT COUNT(customer_id)
    	FROM downgrade
    WHERE downgrade = 'basic monthly'
    ```

    **Result**
    - in 2020, there were no customers who downgraded from pro monthly to basic monthly

### Challenge Payment Question
The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:

- monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
- upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
- upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
- once a customer churns they will no longer make payments
  ```sql
  WITH recursive payment AS (
  SELECT
    s.customer_id,
    p.plan_id,
    p.plan_name,
    p.price,
    s.start_date as payment_date,
    CASE WHEN lead(s.start_date) OVER(PARTITION BY customer_id ORDER by start_date) IS NULL THEN '2020-12-31'
	  ELSE lead(s.start_date) OVER(PARTITION BY customer_id ORDER by start_date)
    END as lead_start_date
  FROM subscriptions s
	JOIN plans p ON s.plan_id = p.plan_id
	WHERE s.start_date <= '2020-12-31'
	UNION ALL
	SELECT
    customer_id,
    plan_id,
    plan_name,
	  price,
    CASE WHEN plan_name like '%monthly' THEN CAST(payment_date+interval '1 month' AS DATE)
         WHEN plan_name = 'pro annual' THEN CAST(payment_date + interval '1 year' as DATE)
  END payment_date,
	lead_start_date
	FROM payment
	WHERE CAST(payment_date+interval '1 month' AS DATE) <= lead_start_date
	AND plan_name NOT IN ('trial','churn')
  )
  SELECT
    customer_id,
    plan_id,
    payment_date,
    price
  FROM payment a
  ORDER BY customer_id, plan_id, payment_date
  ```

  **Result**

  <img src="https://github.com/user-attachments/assets/e8b1ce3b-4f31-4a11-9257-9b9e341b3a04" alt="Case Study #3: Foodie-Fi" width="280" height="500">
