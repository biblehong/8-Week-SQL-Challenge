# Case Study #8 : Fresh Segments

<img src="https://github.com/user-attachments/assets/f8730253-aac0-4280-859c-f01e34ae7291" alt="Case Study #8: Fresh Segments" width="300" height="300">

> [!NOTE]
> Dataset and images are all sourced from [#8WeekSQLChallenge](https://8weeksqlchallenge.com/)

## Table of Contents
- [Introduction](#introduction)
- [Table Relationship](#table-relationship)
- [Business Questions](#business-questions)
  - [Data Exploration and Cleansing](#data-exploration-and-cleansing)
  - [Interest Analysis](#interest-analysis)
  - [Segment Analysis](#segment-analysis)
  - [Index Analysis](#index-analysis)
- [Conclusion](#conclusion)

## Introduction

Danny's newest digital marketing agency, Fresh Segments, helps other companies analyze trends in online ad click behaviour for this unique customer base.

Clients share their customer lists with the Fresh Segments team who then aggregate interest metrics and generate a single dataset worth of metrics for further analysis.

In particular - the composition and rankings for different interests are provided for each client showing the proportion of their customer list who interacted with online assets related to each interest for each month.

Danny has asked for your assistance to analyse aggregated metrics for an example client and provide some high level insights about the customer list and their interests.

## Table Relationship

<img src="https://github.com/user-attachments/assets/70f807ff-82ff-466d-994d-280a0fee76f7" alt="Case Study #8: Fresh Segments" width="500" height="400">

## Business Questions

### Data Exploration and Cleansing
1. Update the fresh_segments.interest_metrics table by modifying the month_year column to be a date data type with the start of the month
What is count of records in the fresh_segments.interest_metrics for each month_year value sorted in chronological order (earliest to latest) with the null values appearing first?
```sql
ALTER TABLE fresh_segments.interest_metrics
ALTER COLUMN month_year TYPE date USING TO_DATE(RIGHT(month_year,4) || '-' || LEFT(month_year,2) || '-01','YYYY-MM-DD');

SELECT
  month_year,
  COUNT(*) cnt_records 
FROM fresh_segments.interest_metrics
GROUP BY month_year
ORDER BY month_year NULLS FIRST;
```

**Result**

![image](https://github.com/user-attachments/assets/d4a627ba-bc4a-407e-bd4d-e0e5813da6a1)

2. What do you think we should do with these null values in the fresh_segments.interest_metrics
- These entries have null value on _month, _year, month_year and interest_id columns. As this study is related to customer interests aggregated on a monthly basis, the availability of these columns are critical. I would perform the following steps:
  - The null entries comprised 8.36% of the entire dataset. This takes up a large portion of the entire dataset and may portentially affect the results of the analysis.

    ![image](https://github.com/user-attachments/assets/729ec085-2616-43ec-9b75-ee62d9490d2b)

  - I would contact the client seeking clarity on these entries with null values, how to interpret them, understand the calculation behind the other fields (e.g. composition, index_value) and gaining knowledge on how these entries will affect the overall analysis.
  - Based on the importance of these rows, I would then set client expectations on the impact of cleansing these entries either through removal of these entries from the dataset or imputing with default values to allocate them to a certain interest category. (Note: allocating to a certain category only if it is relevant to the study. Otherwise, deleting the entries is the suggested method once aligned with the client)

- For the purpose of this study, I will choose the method of deleting the entries from the table.
  ```sql
  DELETE
  FROM fresh_segments.interest_metrics
  WHERE month_year IS NULL
  ```

  **Result**
  
  ![image](https://github.com/user-attachments/assets/a180eb4b-63c7-469b-8674-5e2a203865be)

3. How many interest_id values exist in the fresh_segments.interest_metrics table but not in the fresh_segments.interest_map table? What about the other way around?
- Exists in interest_metrics but missing in interest_map
  ```sql
  SELECT
    COUNT(DISTINCT interest_id) cnt_missing
	FROM fresh_segments.interest_metrics
	WHERE interest_id::integer NOT IN (
                        SELECT id
                        FROM fresh_segments.interest_map
                        )
  ```

  **Result**

  ![image](https://github.com/user-attachments/assets/224bc7a7-c9b5-4e92-b95a-d5cd0696f127)

- Exists in interest_map but missing in interest_metrics
  ```sql
  SELECT
    COUNT(DISTINCT id) cnt_missing
	FROM fresh_segments.interest_map
	WHERE id NOT IN (
          SELECT interest_id::integer
          FROM fresh_segments.interest_metrics
          )
  ```

  **Result**

  ![image](https://github.com/user-attachments/assets/864847a0-5149-4942-8aff-3a786f64db09)

- 7 entries exist in interest_map but are not in interest_metrics while all interest_ids from interest_metrics are correctly configured in interest_map
  
4. Summarise the id values in the fresh_segments.interest_map by its total record count in this table
   ```sql
   SELECT
     m.id,
     m.interest_name,
     COUNT(me.interest_id) cnt_id
   FROM fresh_segments.interest_map m
   JOIN fresh_segments.interest_metrics me ON me.interest_id::integer = m.id
   GROUP BY m.interest_name, m.id
   ORDER BY cnt_id DESC, m.id
   ```

   **Result**
   
   ![image](https://github.com/user-attachments/assets/60e7023d-5471-4cd8-8e56-98a7d2b5a62d)

   - Assuming "total record count in this table" means record in the interest_metrics table, the highest record number based on id is 14.

5. What sort of table join should we perform for our analysis and why? Check your logic by checking the rows where interest_id = 21246 in your joined output and include all columns from fresh_segments.interest_metrics and all columns from fresh_segments.interest_map except from the id column.
- We should use INNER JOIN. The goal of the study is to observe the ad click behaviour of the client's customer base, which means that interest_metrics is the primary source of the analysis. The table interest_map, in this scenario, is used as a reference table containing all the valid values to maintain referential integrity within the relational database.
  
6. Are there any records in your joined table where the month_year value is before the created_at value from the fresh_segments.interest_map table? Do you think these values are valid and why?
   ```sql
   SELECT *
   FROM fresh_segments.interest_metrics me
   LEFT JOIN fresh_segments.interest_map m ON me.interest_id::integer = m.id
   WHERE m.created_at > month_year
   ```

   **Result**
   
   ![image](https://github.com/user-attachments/assets/d2875789-f5e9-4f1e-9475-c735fa14f60f)

   - Yes, there were records where month_year is before the created_at date. However, we should not ignore the fact that month_year is always recorded as the first day of the month. We can't accurately say that these are invalid records since the data is aggregated on a monthly level. To be precise, we can instead write the where clause as below which does not return any record:
     
     ```sql
     SELECT *
     FROM fresh_segments.interest_metrics me
     LEFT JOIN fresh_segments.interest_map m ON me.interest_id::integer = m.id
     WHERE DATE_TRUNC('month',m.created_at) > month_year
     ```

### Interest Analysis
1. Which interests have been present in all month_year dates in our dataset?
   ```sql
   SELECT
     interest_id,
     COUNT(DISTINCT month_year) cnt_month_year
   FROM fresh_segments.interest_metrics me
   GROUP BY interest_id
   HAVING COUNT(DISTINCT month_year) = (
                     SELECT
                       COUNT(DISTINCT month_year)
                     FROM fresh_segments.interest_metrics me
                     )
   ```

   **Result**
   
   ![image](https://github.com/user-attachments/assets/e52ba925-67ac-48a3-9cae-d16194cb9741)

2. Using this same total_months measure - calculate the cumulative percentage of all records starting at 14 months - which total_months value passes the 90% cumulative percentage value?
3. If we were to remove all interest_id values which are lower than the total_months value we found in the previous question - how many total data points would we be removing?
4. Does this decision make sense to remove these data points from a business perspective? Use an example where there are all 14 months present to a removed interest example for your arguments - think about what it means to have less months present from a segment perspective.
5. After removing these interests - how many unique interests are there for each month?

### Segment Analysis
1. Using our filtered dataset by removing the interests with less than 6 months worth of data, which are the top 10 and bottom 10 interests which have the largest composition values in any month_year? Only use the maximum composition value for each interest but you must keep the corresponding month_year
   ```sql
   /* replacing ORDER BY composition DESC with ORDER BY composition ASC will display the bottom 10*/ 
   WITH base AS (
   SELECT
     me.interest_id,
     COUNT(DISTINCT me.month_year) cnt_month_year
   FROM fresh_segments.interest_metrics me
   GROUP BY interest_id
   HAVING COUNT(DISTINCT month_year) >= 6
   ),
   comp AS (
   SELECT
     interest_id,
     interest_name,
     composition,
     month_year,
     ROW_NUMBER() OVER(PARTITION BY interest_id ORDER BY composition DESC) rnk
   FROM fresh_segments.interest_metrics me
   JOIN fresh_segments.interest_map m ON me.interest_id::integer = m.id
   WHERE interest_id IN (SELECT interest_id FROM base)
   )
   SELECT
     interest_id,
     interest_name,
     composition,
     month_year
   FROM comp WHERE rnk = 1
   ORDER BY composition DESC
   LIMIT 10;
   ```

   **Result**
   - Top 10
     ![image](https://github.com/user-attachments/assets/afcf8e5f-222a-4451-a8cb-71eeed687806)
     
   - Bottom 10
     ![image](https://github.com/user-attachments/assets/e2a5bf8f-0916-4891-885b-3d1daf6492cf)

2. Which 5 interests had the lowest average ranking value?
   ```sql
   SELECT interest_id, interest_name, ROUND(AVG(ranking),0) avg_ranking
   FROM fresh_segments.interest_metrics me
   JOIN fresh_segments.interest_map m ON me.interest_id::integer = m.id
   GROUP BY interest_id, interest_name
   ORDER BY avg_ranking DESC
   LIMIT 5
   ```

   **Result**
   ![image](https://github.com/user-attachments/assets/97cc640c-2815-4ff6-9340-a7e01eba7be7)

3. Which 5 interests had the largest standard deviation in their percentile_ranking value?
   ```sql
   SELECT
     interest_id,
     interest_name,
     COALESCE(stddev(percentile_ranking),MAX(percentile_ranking)) stddev
   FROM fresh_segments.interest_metrics me
   JOIN fresh_segments.interest_map m ON me.interest_id::integer = m.id
   GROUP BY interest_id, interest_name
   ORDER BY stddev DESC
   LIMIT 5
   ```

   **Result**
   
   ![image](https://github.com/user-attachments/assets/8d9a5c01-ad1c-4cee-9fed-626bc1b6038f)

4. For the 5 interests found in the previous question - what was minimum and maximum percentile_ranking values for each interest and its corresponding year_month value? Can you describe what is happening for these 5 interests?
   ```sql
   WITH base AS (
   SELECT
     interest_id,
     interest_name,
     COALESCE(stddev(percentile_ranking),MAX(percentile_ranking)) stddev
   FROM fresh_segments.interest_metrics me
   JOIN fresh_segments.interest_map m ON me.interest_id::integer = m.id
   GROUP BY interest_id, interest_name
   ORDER BY stddev DESC
   LIMIT 5
   ),
   percentile AS (
   SELECT
     interest_id,
     FIRST_VALUE(month_year) OVER(PARTITION BY interest_id ORDER BY percentile_ranking) AS min_month_year,
     FIRST_VALUE(percentile_ranking) OVER(PARTITION BY interest_id ORDER BY percentile_ranking) AS min_percentile_ranking,
     FIRST_VALUE(month_year) OVER(PARTITION BY interest_id ORDER BY percentile_ranking DESC) AS max_month_year,
     FIRST_VALUE(percentile_ranking) OVER(PARTITION BY interest_id ORDER BY percentile_ranking DESC) AS max_percentile_ranking
   FROM fresh_segments.interest_metrics
   WHERE interest_id IN (SELECT interest_id FROM base)
   )
   SELECT DISTINCT *
   FROM percentile mi
   ```

   **Result**
   
   ![image](https://github.com/user-attachments/assets/af6882bd-f96f-4881-b848-8499c0d6fffb)

   - Interest ids 10011, 10365 and 5999 only had 1 occurence each in the entire dataset therefore showing no difference between minimum and maximum percntile rankings. Ids 131 (Android Fans) and 6260 (Blockbuster Movie Fans) on the other hand, fell in ranking between 2018 to 2019 which meant that customers were losing interest in these ads. This could be impacted by several factors such as topic seasonality (e.g. no new Android products released or no new movies starring popular actors), or introduction of new ads making customer lose interest in the existing ones, or no new updates on the thread. 

5. How would you describe our customers in this segment based off their composition and ranking values? What sort of products or services should we show to these customers and what should we avoid?
   ```sql
   SELECT m.interest_name, me.interest_id, MAX(composition) composition
   FROM fresh_segments.interest_metrics me
   JOIN fresh_segments.interest_map m ON me.interest_id::integer = m.id
   GROUP BY m.interest_name, me.interest_id
   ORDER BY composition DESC, interest_id
   ```

   **Result**
   ![image](https://github.com/user-attachments/assets/faa6def4-4df3-4308-a871-bd161849fab2)

   **Result**
   - `composition` is the percentage of the client's customer base that interacted with the ad. Based on the top 10 compositions, it appears the customers' interests revolve around topics in gym equipment owners, online shopping (e.g. furniture, fashion, bedroom items, cosmetics etc.) and hotel booking. The client should focus on showing ads related to these areas while showing related materials on the same page such as gym equipment on gym equipment owners' page, or jewelries on luxury pages, or hotel deals on hotel booking pages.

   ```sql
   SELECT
     m.interest_name,
     m.interest_summary,
     me.interest_id,
     COUNT(*) cnt_ranking
   FROM fresh_segments.interest_metrics me
   JOIN fresh_segments.interest_map m ON me.interest_id::integer = m.id
   WHERE ranking BETWEEN 1 AND 5
   GROUP BY m.interest_name, m.interest_summary, me.interest_id
   ORDER BY cnt_ranking DESC, interest_id
   ```

   **Result**

   ![image](https://github.com/user-attachments/assets/7312440b-852f-452c-9f83-e45202650cfe)

   - `ranking` is the measure by how many times the `index_value` ranked higher than the rest of interests. Taking ranks 1-5 as sample size, we can observe a pattern on the customers' interests in sports clothing. Showing related materials on the same page such as gym equipment, shoes or sports gadgets will potentially help boost sales.

### Index Analysis
The index_value is a measure which can be used to reverse calculate the average composition for Fresh Segments’ clients.

Average composition can be calculated by dividing the composition column by the index_value column rounded to 2 decimal places.

1. What is the top 10 interests by the average composition for each month?
   ```sql
   WITH avg_com AS (
     SELECT
       me.month_year,
       me.interest_id,
       m.interest_name,
       ROUND(me.composition::numeric/me.index_value::numeric,2) as avg_composition,
       ROW_NUMBER() OVER(PARTITION BY me.month_year ORDER BY ROUND(me.composition::numeric/me.index_value::numeric,2) DESC) rnk
   FROM fresh_segments.interest_metrics me
   JOIN fresh_segments.interest_map m ON me.interest_id::integer = m.id
   )
   SELECT *
   FROM avg_com
   WHERE rnk <= 10
   ```

   **Result**

   ![image](https://github.com/user-attachments/assets/1564390d-fee1-43f7-8703-25792a60bbca)

2. For all of these top 10 interests - which interest appears the most often?
   ```sql
   WITH avg_com AS (
   SELECT
     me.month_year,
     me.interest_id,
     m.interest_name,
     ROUND(me.composition::numeric/me.index_value::numeric,2) as avg_composition,
     ROW_NUMBER() OVER(PARTITION BY me.month_year ORDER BY ROUND(me.composition::numeric/me.index_value::numeric,2) DESC) rnk
   FROM fresh_segments.interest_metrics me
   JOIN fresh_segments.interest_map m ON me.interest_id::integer = m.id
   )
   SELECT interest_id, interest_name, COUNT(*) cnt
   FROM avg_com
   WHERE rnk <= 10
   GROUP BY interest_id, interest_name
   ORDER BY cnt DESC
   ```

   **Result**

   ![image](https://github.com/user-attachments/assets/768fd4dd-a3e7-448d-acaf-3ff41aa57352)

   - 3 interests, Alabama Trip Planner, Luxury Bedding Shoppers and Solar Energy Researchers, have tied being the most often top 10 average composition each month. There is no clear pattern as to which specific area is most popular among customers.

3. What is the average of the average composition for the top 10 interests for each month?
   ```sql
   WITH avg_com AS (
   SELECT
     me.month_year,
     me.interest_id,
     m.interest_name,
     ROUND(me.composition::numeric/me.index_value::numeric,2) as avg_composition,
     ROW_NUMBER() OVER(PARTITION BY me.month_year ORDER BY ROUND(me.composition::numeric/me.index_value::numeric,2) DESC) rnk
   FROM fresh_segments.interest_metrics me
   JOIN fresh_segments.interest_map m ON me.interest_id::integer = m.id
   )
   SELECT month_year, ROUND(AVG(avg_composition),2) avg
   FROM avg_com
   WHERE rnk <= 10
   GROUP BY month_year
   ORDER BY month_year
   ```

   **Result**

   ![image](https://github.com/user-attachments/assets/d98f053e-0557-43c9-adc5-47257eb73b28)

4. What is the 3 month rolling average of the max average composition value from September 2018 to August 2019 and include the previous top ranking interests in the same output shown below.
   ```sql
   WITH avg_com AS (
   SELECT
     me.month_year,
     me.interest_id,
     m.interest_name,
     ROUND(me.composition::numeric/me.index_value::numeric,2) as avg_composition,
     ROW_NUMBER() OVER(PARTITION BY me.month_year ORDER BY ROUND(me.composition::numeric/me.index_value::numeric,2) DESC) rnk
   FROM fresh_segments.interest_metrics me
   JOIN fresh_segments.interest_map m ON me.interest_id::integer = m.id
   ),
   moving_avg AS (
   SELECT
     month_year,
     interest_name,
     avg_composition as max_index_composition,
     ROUND(AVG(avg_composition) OVER(ORDER BY month_year ROWS BETWEEN 2 PRECEDING AND CURRENT ROW),2) "3_month_moving_avg",
     LAG(interest_name,1,NULL) OVER(ORDER BY month_year) || ': ' || LAG(avg_composition,1,0) OVER(ORDER BY month_year) "1_month_ago",
     LAG(interest_name,2,NULL) OVER(ORDER BY month_year) || ': ' || LAG(avg_composition,2,0) OVER(ORDER BY month_year) "2_months_ago"
   FROM avg_com
   WHERE rnk = 1
   )
   SELECT *
   FROM moving_avg
   WHERE month_year BETWEEN '2018-09-01' AND '2019-08-01'
   ORDER BY month_year
   ```

   **Result**

   ![image](https://github.com/user-attachments/assets/da40b65c-b373-4df3-bce3-0213b9b2ceba)

5. Provide a possible reason why the max average composition might change from month to month? Could it signal something is not quite right with the overall business model for Fresh Segments?
- A change in ranking on the top average composition does not signal a problem in the business model but could be influenced by external factors. For instance, a new iPhone release could trigger more customer interaction on composition related to news and features of iPhone releases, or a tri-athlon event could trigger more views on sports clothing, flight tickets and hotel bookings.

  However, if we observe the max average composition trend as a whole from Sep 2018 to Aug 2019, we can observe a significant decline on this measure. This trend is what I would consider as a problem in the overall business model as it could mean that customers are slowly losing interest in the client's online assets and not generating enough new topics to keep things interesting for customers.

## Conclusion
As the client data is an aggregated version sent over to Fresh Segments analytics team, data needed to be analyzed, reviewed and understood before being cleansed. On a normal case, I would consult with clients on how to interpret and handle the records with null values but for the purpose of the study, cleansing was achieved by removing entries with null _month, _year, month_year and interest_id values.

Based on composition, we could see that popular assets are focused around luxury items, hotel bookings and online shopping. The client could boost sales on other assets by creating ads similar to these top assets such as deals on hotel bookings, flight tickets, and sale on online shopping items. 

We can see a decline in the overall trend of max avg composition further supported by the moving average to flatten the graph. This could indicate a potential problem in the business model. It is recommended to have the client review the customer interaction on a transaction level and add/update their assets with the newest contents to make them more engaging and interesting to customers. 
