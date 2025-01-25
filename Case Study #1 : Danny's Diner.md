# Case Study #1 : Danny's Diner

<img src="https://github.com/user-attachments/assets/d8c5c831-c5e9-45c8-b558-f5ab15c62fe1" alt="Cast Study #1 : Danny's Diner" width="300" height="300">

> [!NOTE]
> Dataset and images are all sourced from [8 Week SQL Challenge](https://8weeksqlchallenge.com/)

## Introduction
Danny has opened his new restaurant and wants to understand the data about his customers, particularly their visiting patterns, purchases, and food preferences. This analysis will help him deliver a more personalized experience to his customers and help him decide on the campaign approach he needs to use for his business. 

## Table Relationship
<img src="https://github.com/user-attachments/assets/186e4fbe-6d0a-4e61-b805-3b803fee69a3" alt="Table Relationship" width="500" height="300">

## Business Questions
1. What is the total amount each customer spent at the restaurant?
   ```
   SELECT
     customer_id,
     SUM(price) as total
   FROM sales s
   JOIN menu m on s.product_id = m.product_id
   GROUP BY customer_id
   ORDER BY customer_id
   ```
   $${\color{lightgreen}Result}$$
    | customer_id | total |
    | ----------- | ----- |
    | A           | 76    |
    | B           | 74    |
    | C           | 36    |

    $${\color{lightgreen}Quick \space Insight}$$
   - Customer A made the biggest purchase in terms of amount.
   
2. How many days has each customer visited the restaurant?
   ```
   SELECT
     customer_id,
     COUNT(DISTINCT order_date) as no_of_days_visited
   FROM sales
   GROUP BY customer_id
   ```

   $${\color{lightgreen}Result}$$
   | customer_id | no_of_days_visited |
   | ----------- | ------------------ |
   | A           | 4                  |
   | B           | 6                  |
   | C           | 2                  |

   $${\color{lightgreen}Quick \space Insight}$$
   - Customer B paid the most visits to the restaurant.

3. What was the first item from the menu purchased by each customer?
   ```
   WITH first_product as (
   SELECT
     s.customer_id,
     m.product_name,
     RANK() OVER(PARTITION BY customer_id ORDER BY order_date) rnk
   FROM sales s
   JOIN menu m on s.product_id = m.product_id
   )
   SELECT
     customer_id,
     product_name
   FROM first_product
   WHERE rnk = 1
   ```

   $${\color{lightgreen}Result}$$
   | customer_id | product_name |
   | ----------- | ------------ |
   | A           | curry        |
   | A           | sushi        |
   | B           | curry        |
   | C           | ramen        |

   $${\color{lightgreen}Quick \space Insight}$$
   - Customer A purchased 2 items on his/her first visit to the restaurant while Customers B and C purchased 1 item each.

4. What is the most purchased item on the menu and how many times was it purchased by all customers?
    ```
    SELECT 
      m.product_name,
      COUNT(s.product_id) as purchase_cnt
    FROM sales s
    JOIN menu m on s.product_id = m.product_id
    GROUP BY m.product_name
    ORDER BY purchase_cnt DESC
    LIMIT 1
    ```
    
    $${\color{lightgreen}Result}$$
   | product_name | purchase_cnt |
   | ------------ | ------------ |
   | ramen        | 8            |

   $${\color{lightgreen}Quick \space Insight}$$
   - Ramen is the most popular dish on the menu, purchased by customers 8 times.
     
5. Which item was the most popular for each customer?
   ```
   WITH popular AS (
   SELECT
     s.customer_id,
     m.product_name,
   RANK() OVER(PARTITION BY s.customer_id ORDER BY COUNT(s.product_id) DESC) as popular_menu
   FROM sales s
   JOIN menu m on s.product_id = m.product_id
   GROUP BY s.customer_id, m.product_name
   )
   SELECT customer_id,
     product_name
   FROM popular
   WHERE popular_menu = 1
   ```
   
   $${\color{lightgreen}Result}$$
   | customer_id | product_name | 
   | ----------- | ------------ | 
   | A           | ramen        | 
   | B           | ramen        | 
   | B           | curry        | 
   | B           | sushi        | 
   | C           | ramen        | 

   $${\color{lightgreen}Quick \space Insight}$$
   - Ramen seems to be the most popular dish with all the customers. 
   
6. Which item was purchased first by the customer after they became a member?
   ```
   WITH member_purchase AS (
   SELECT
     s.customer_id,
     m.product_name,
     RANK() OVER(PARTITION BY s.customer_id ORDER BY order_date ASC) rnk
   FROM sales s
   JOIN menu m on s.product_id = m.product_id
   JOIN members me on me.customer_id = s.customer_id
   WHERE order_date >= join_date
   )
   SELECT
     customer_id,
     product_name
   FROM member_purchase
   WHERE rnk = 1
   ```
   
   $${\color{lightgreen}Result}$$
   | customer_id | product_name | 
   | ----------- | ------------ | 
   | A           | curry        | 
   | B           | sushi        | 

   $${\color{lightgreen}Quick \space Insight}$$
   - Assuming purchases made on the same day as the membership date are included, curry was the first item purchased by Customer A and sushi for Customer B after they became a member.
     
7. Which item was purchased just before the customer became a member?
   ```
   WITH nonmember_purchase AS (
   SELECT
     s.customer_id,
     m.product_name,
     RANK() OVER(PARTITION BY s.customer_id ORDER BY order_date ASC) rnk
   FROM sales s
   JOIN menu m on s.product_id = m.product_id
   JOIN members me on me.customer_id = s.customer_id
   WHERE order_date < join_date
   )
   SELECT DISTINCT
   customer_id,
   product_name
   FROM nonmember_purchase
   WHERE rnk = 1
   ORDER BY customer_id, product_name
   ```
   
   $${\color{lightgreen}Result}$$
   | customer_id | product_name | 
   | ----------- | ------------ | 
   | A           | curry        | 
   | A           | sushi        |
   | B           | curry        |  

   $${\color{lightgreen}Quick \space Insight}$$
   - Customer A: curry and sushi (purchased on the same day)
   - Customer B: curry
   - These were the last items purchased by these customers right before they became members.
   
8. What is the total items and amount spent for each member before they became a member?
   ```
   WITH nonmember_purchase AS (
   SELECT
     s.customer_id,
     s.product_id,
     m.price
   FROM sales s
   JOIN menu m on s.product_id = m.product_id
   JOIN members me on me.customer_id = s.customer_id
   WHERE order_date < join_date
   )
   SELECT
     customer_id,
     COUNT(product_id) as nonmember_purchase_cnt,
     SUM(price) as nonmember_total_sales
   FROM nonmember_purchase
   GROUP BY customer_id
   ORDER BY customer_id
   ```
   $${\color{lightgreen}Result}$$
   | customer_id | nonmember_purchase_cnt | nonmember_total_sales |
   | ----------- | ---------------------- | --------------------- |
   | A           | 2                      | 25                    |
   | B           | 3                      | 40                    |

   $${\color{lightgreen}Quick \space Insight}$$
   - Customer A purchased curry and sushi (purchased on the same day) while Customer B purchased curry right before they became members.
   
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
   ```
   WITH member_purchase AS (
   SELECT
     s.customer_id,
     m.product_name,
     m.price
   FROM sales s
   JOIN menu m on s.product_id = m.product_id
   JOIN members me on me.customer_id = s.customer_id
   WHERE order_date >= join_date
   )
   SELECT
     customer_id,
     SUM(
       CASE WHEN product_name = 'sushi' THEN 20*price
       ELSE 10*price
       END
     ) as total_points
   FROM member_purchase
   GROUP BY customer_id
   ORDER BY customer_id
   ```

   $${\color{lightgreen}Result}$$
   | customer_id | total_points |
   | ----------- | ------------ |
   | A           | 510          |
   | B           | 440          |

   $${\color{lightgreen}Quick \space Insight}$$
   - Assuming points are awarded only to members and purchases made on the same day as the membership date are included, Customer A has accumulated more points as he has purchased an item on the same day he became a member.
     
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
    ```
    WITH member_purchase AS (
    SELECT
      s.customer_id,
      m.product_name,
      s.order_date,
      me.join_date,
      m.price
    FROM sales s
    JOIN menu m on s.product_id = m.product_id
    JOIN members me on me.customer_id = s.customer_id
    WHERE order_date >= join_date
    AND order_date <= '2021-01-31'
    )
    SELECT
      customer_id,
      SUM(
        CASE WHEN product_name = 'sushi' THEN 20*price
        WHEN order_date < join_date + interval '7 days' THEN 20*price 
      	ELSE 10*price 
      	END
      )
    FROM member_purchase
    GROUP BY customer_id
    ORDER BY customer_id
    ```

    $${\color{lightgreen}Result}$$
    | customer_id | total_points |
    | ----------- | ------------ |
    | A           | 1020         |
    | B           | 320          |

    $${\color{lightgreen}Quick \space Insight}$$
    - Assuming "first week" is 7 days from membership date (join date included), Customer A has accumulated more points because all his purchases were within 7 days from join date while Customer B only had 1 purchase qualified for the double points.

## Conclusion 


