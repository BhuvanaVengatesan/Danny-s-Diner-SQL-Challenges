# Case Study #1 - Danny's Diner

<img src="https://github.com/BhuvanaVengatesan/Danny-s-Diner-SQL-Challenges/assets/172362151/e31bcf7f-71cf-4963-8bfa-41a170d259bc" width="300">

View the case study [here](https://8weeksqlchallenge.com/case-study-1/)

# Entity Relationship Diagram

<img src="https://user-images.githubusercontent.com/81607668/127271130-dca9aedd-4ca9-4ed8-b6ec-1e1920dca4a8.png" width="600">

# Introduction

Danny seriously loves Japanese food so in the beginning of 2021, he decides to embark upon a risky venture and opens up a cute little restaurant that sells his 3 favourite foods: sushi, curry and ramen.

Danny’s Diner is in need of your assistance to help the restaurant stay afloat - the restaurant has captured some very basic data from their few months of operation but have no idea how to use their data to help them run the business.

# Datasets used
* sales: The sales table captures all customer_id level purchases with an corresponding order_date and product_id information for when and what menu items were ordered.
* menu: The menu table maps the product_id to the actual product_name and price of each menu item.
* members: The members table captures the join_date when a customer_id joined the beta version of the Danny’s Diner loyalty program.

# Case Study Questions
1. What is the total amount each customer spent at the restaurant?
2. How many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What is the total items and amount spent for each member before they became a member?
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

# Solution
## 1. What is the total amount each customer spent at the restaurant?

```
SELECT s.customer_id , CONCAT('$', (SUM(m.price))) AS total_sales
FROM sales s INNER JOIN menu m
ON s.product_id=m.product_id
GROUP BY customer_id;
```

![image](https://github.com/BhuvanaVengatesan/Danny-s-Diner-SQL-Challenges/assets/172362151/e2a265b4-5172-478b-8edf-72740cd883d3)

## 2. How many days has each customer visited the restaurant?
```
SELECT customer_id, COUNT(DISTINCT(order_date)) AS days_visited FROM sales
GROUP BY customer_id;
```
![image](https://github.com/BhuvanaVengatesan/Danny-s-Diner-SQL-Challenges/assets/172362151/7afb7197-bc62-4910-9448-687f82cf02d7)

## 3. What was the first item from the menu purchased by each customer?
```
WITH cte AS(
SELECT s.customer_id, m.product_name, s.order_date, 
DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS rank1 FROM sales s INNER JOIN menu m
ON s.product_id=m.product_id
GROUP BY  s.customer_id, m.product_name, s.order_date
)
SELECT customer_id, product_name FROM cte
WHERE rank1=1;
```
![image](https://github.com/BhuvanaVengatesan/Danny-s-Diner-SQL-Challenges/assets/172362151/dcf2cd3e-a3f7-4c1d-b670-b03dbb64a811)

## 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```
SELECT m.product_name, COUNT(s.product_id) AS ordered FROM sales AS s 
INNER JOIN menu AS m
ON  s.product_id=m.product_id
GROUP BY m.product_name
ORDER BY COUNT(s.product_id) DESC
LIMIT 1;
```
![image](https://github.com/BhuvanaVengatesan/Danny-s-Diner-SQL-Challenges/assets/172362151/adbc6ea0-0075-46fe-ada8-7f24b6c5a741)

## 5. Which item was the most popular for each customer?

```
WITH fav_item AS (
SELECT 
s.customer_id, 
m.product_name, 
COUNT(s.product_id) AS order_count,
DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY COUNT(s.product_id) DESC) AS Ranking
FROM sales AS s 
INNER JOIN menu AS m
ON  s.product_id=m.product_id
GROUP BY s.customer_id, m.product_name
)
SELECT customer_id, product_name, order_count FROM fav_item
WHERE ranking=1;

```
![image](https://github.com/BhuvanaVengatesan/Danny-s-Diner-SQL-Challenges/assets/172362151/6b1d7024-7da2-4767-beaa-0dcefb885ef6)

## 6. Which item was purchased first by the customer after they became a member?
```
WITH FirstItems_AfterMember AS
( 
SELECT s.customer_id, 
m.product_name, 
DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS ranking 
FROM sales s JOIN menu m
ON s.product_id=m.product_id 
JOIN members a 
ON a.customer_id=s.customer_id 
WHERE s.order_date >= a.join_date)

SELECT customer_id, product_name FROM FirstItems_AfterMember
WHERE ranking=1;
```
![image](https://github.com/BhuvanaVengatesan/Danny-s-Diner-SQL-Challenges/assets/172362151/68a30af4-411f-415c-bdc4-a31d07bbe39d)

## 7. Which item was purchased just before the customer became a member?
```
WITH cte as (
SELECT  s.customer_id, m.product_name, 
DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS Rank1
FROM sales AS s 
INNER JOIN menu AS m 
ON s.product_id = m.product_id
INNER JOIN members AS b
ON b.customer_id = s.customer_id
WHERE s.order_date < b.join_date
)
SELECT customer_id, product_name from cte
WHERE rank1=1;
```
![image](https://github.com/BhuvanaVengatesan/Danny-s-Diner-SQL-Challenges/assets/172362151/917344e6-a71c-4e45-b6ca-35091aef36ba)


## 8.  What is the total items and amount spent for each member before they became a member?

```
SELECT  s.customer_id, COUNT(s.product_id) AS total_items, CONCAT('$', SUM((m.price))) AS amount_spent
FROM sales AS s 
INNER JOIN menu AS m 
ON s.product_id = m.product_id
INNER JOIN members AS b
ON b.customer_id = s.customer_id
WHERE s.order_date < b.join_date
GROUP BY s.customer_id;
```
![image](https://github.com/BhuvanaVengatesan/Danny-s-Diner-SQL-Challenges/assets/172362151/fdc5238b-7873-4ce2-94c5-2477776ed99a)

## 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```
WITH cte AS (
SELECT s.customer_id, m.product_name,
CASE WHEN m.product_name='sushi' THEN m.price*20
ELSE m.price*10 END AS points
FROM sales AS s 
INNER JOIN menu AS m
ON s.product_id=m.product_id
)
SELECT customer_id, sum(points) AS total_points
FROM cte
GROUP BY customer_id
ORDER BY customer_id;
```
![image](https://github.com/BhuvanaVengatesan/Danny-s-Diner-SQL-Challenges/assets/172362151/1ed45a74-6a8c-46a3-bab2-5c46780cc77d)

## 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```
WITH cte AS
  (SELECT join_date,
          DATE_ADD(join_date, INTERVAL 6 DAY) AS program_date,
          customer_id
   FROM members)
SELECT s.customer_id,
       SUM(CASE
               WHEN s.order_date BETWEEN b.join_date AND b.program_date THEN m.price*20
               WHEN s.order_date NOT BETWEEN b.join_date AND b.program_date
                    AND m.product_name = 'sushi' THEN m.price*20
               WHEN s.order_date NOT BETWEEN b.join_date AND b.program_date
                    AND m.product_name != 'sushi' THEN m.price*10
           END) AS customer_points
FROM menu AS m
INNER JOIN sales AS s ON m.product_id = s.product_id
INNER JOIN cte AS b ON b.customer_id = s.customer_id
AND s.order_date <='2021-01-31'
AND s.order_date >=join_date
GROUP BY s.customer_id
ORDER BY s.customer_id;
```
![image](https://github.com/BhuvanaVengatesan/Danny-s-Diner-SQL-Challenges/assets/172362151/405bff05-1961-4bc1-8b09-6e25b52112de)

