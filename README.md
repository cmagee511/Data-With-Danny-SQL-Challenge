# Data-With-Danny-SQL-Challenge
This is a case study from the 8 Week SQL Challenge by Data with Danny.

This first case study is on Danny's Diner which can be found below.

## Case Study 1                                                                                                                                       
[Case Study 1 - Danny's Diner](https://8weeksqlchallenge.com/case-study-1/) 

## SQL Soultions  (Ranking,CTEs, Case Statements, Dates, Substrings)

[SQL Soultions](https://github.com/cmagee511/Data-With-Danny-SQL-Challenge/blob/main/SQL%20Solutions.sql)

## Problem Statement

Danny wants to use the data to answer a few simple questions about his customers, especially about their visiting patterns, how much money they’ve spent and also which menu items are their favourite. Having this deeper connection with his customers will help him deliver a better and more personalised experience for his loyal customers.

He plans on using these insights to help him decide whether he should expand the existing customer loyalty program - additionally he needs help to generate some basic datasets so his team can easily inspect the data without needing to use SQL.

Danny has provided you with a sample of his overall customer data due to privacy issues - but he hopes that these examples are enough for you to write fully functioning SQL queries to help him answer his questions!


## Case Study Questions

-- 1. What is the total amount each customer spent at the restaurant?
```sql  
SELECT s.customer_id, SUM(m.price) as total_spend
FROM sales as s
INNER JOIN menu as m 
ON s.product_id = m.product_id
GROUP BY s.customer_id;
```


-- 2. How many days has each customer visited the restaurant?
```sql 
SELECT customer_id, COUNT(DISTINCT order_date) as days_visited
FROM sales
Group by customer_id;
```


-- 3. What was the first item from the menu purchased by each customer?
```sql  
WITH table_1 AS (
SELECT s.customer_id, m.product_name, s.order_date,
RANK()  OVER(PARTITION BY s.customer_id ORDER BY s.order_date) as rnk
FROM sales as s 
INNER JOIN menu as m
ON s.product_id = m.product_id)

SELECT customer_id, product_name
FROM final_table
Where rnk = 1;
```
-- 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
SELECT m.product_name, 
COUNT(order_date) as orders
FROM sales as s
INNER JOIN menu as m 
ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY orders DESC
LIMIT 1;
```
-- 5. Which item was the most popular for each customer?
```sql
SELECT customer_id,product_name
FROM (
SELECT s.customer_id, m.product_name,
Rank() OVER(partition by s.customer_id ORDER BY COUNT(s.order_date) DESC) as rnk
FROM sales as s
INNER JOIN menu as m 
ON s.product_id = m.product_id
GROUP BY s.customer_id, m.product_name) x
WHERE rnk = 1
GROUP BY customer_id,product_name;
```

-- 6. Which item was purchased first by the customer after they became a member?
```sql
WITH TABLE_2 AS (
SELECT mem.customer_id, m.product_name, s.order_date, mem.join_date,
RANK() OVER(PARTITION BY mem.customer_id  ORDER BY mem.join_date,s.order_date) as rnk
FROM sales as s
INNER JOIN menu as m 
ON s.product_id = m.product_id
INNER JOIN members as mem
ON s.customer_id = mem.customer_id
WHERE s.order_date <= mem.join_date)

SELECT customer_id, product_name
FROM TABLE_2
WHERE rnk = 1;
```
-- 7. Which item was purchased just before the customer became a member?
```sql
WITH TABLE_3 AS (
SELECT mem.customer_id, m.product_name, s.order_date, mem.join_date,
RANK() OVER(PARTITION BY mem.customer_id  ORDER BY mem.join_date,s.order_date DESC) as rnk
FROM sales as s
INNER JOIN menu as m 
ON s.product_id = m.product_id
INNER JOIN members as mem
ON s.customer_id = mem.customer_id
WHERE s.order_date < mem.join_date)

SELECT customer_id, product_name
FROM TABLE_3 
WHERE rnk = 1;
```

-- 8. What is the total items and amount spent for each member before they became a member?
```sql
SELECT mem.customer_id, COUNT(m.product_name) as total_items, SUM(m.price) as Total_Spent
FROM sales as s
INNER JOIN menu as m 
ON s.product_id = m.product_id
INNER JOIN members as mem
ON s.customer_id = mem.customer_id
WHERE s.order_date < mem.join_date
GROUP BY mem.customer_id
ORDER BY mem.customer_id;
```

-- 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
SELECT s.customer_id,
SUM(CASE WHEN m.product_name = 'Sushi' THEN Price * 10 * 2
WHEN m.product_name = 'Curry' THEN Price * 10
WHEN m.product_name = 'Ramen' THEN Price * 10 ELSE 0 END) AS Points
FROM sales as s
INNER JOIN menu as m 
ON s.product_id = m.product_id
GROUP BY s.customer_id;
```


-- 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql
SELECT s.customer_id,
SUM(CASE 
WHEN s.order_date BETWEEN mem.join_date AND mem.join_date + INTERVAL  6 DAY THEN Price * 10 * 2
WHEN m.product_name = 'Sushi' THEN Price * 10 * 2
WHEN m.product_name = 'Curry' THEN Price * 10
WHEN m.product_name = 'Ramen' THEN Price * 10 ELSE 0 END) AS Points
FROM sales as s
INNER JOIN menu as m 
ON s.product_id = m.product_id
INNER JOIN members as mem
ON s.customer_id = mem.customer_id
WHERE EXTRACT(Month FROM s.order_date) = 1
GROUP BY s.customer_id
ORDER BY s.customer_id;
```




