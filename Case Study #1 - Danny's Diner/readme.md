# Case Study #1 - Danny's Diner
![1](https://github.com/marswanttobeanalyst/8-Week-SQL-Challenge/assets/141108687/194e7616-a7f2-4253-a820-aa52a54cdea6)

Link to the case study: [here](https://8weeksqlchallenge.com/case-study-1/)

## Introduction
Danny has a severe love for Japanese cuisine, so in the beginning of 2021 he makes the dangerous decision to build a cute little restaurant serving his three favourite dishes, sushi, curry, and ramen.
Danny's Diner is in need of your aid to stay in business. Although they have collected some very basic data over the course of their short time in company, they are unsure of how to use it.

## Problem Statement
Danny wants to use the information to get basic answers about his consumers, particularly regarding their spending habits, frequency of visits, and favourite menu items. He will be able to give his devoted consumers a better and more individualised experience thanks to this stronger relationship with them.

He intends to utilise these insights to inform his decision about whether to expand the current customer loyalty programme. He also needs assistance in creating some simple datasets so his team can quickly examine the data without the need for SQL.


## Datasets used
-The sales table captures all customer_id level purchases with an corresponding order_date and product_id information for when and what menu items were ordered.
-The menu table maps the product_id to the actual product_name and price of each menu item.
-The final members table captures the join_date when a customer_id joined the beta version of the Danny’s Diner loyalty program.

## Entity Relationship Diagram
![ERD](https://github.com/marswanttobeanalyst/8-Week-SQL-Challenge/assets/141108687/8a65c7f6-eac8-4731-86d4-a5135d728581)

## Questions and Answers

**1. What is the total amount each customer spent at the restaurant?**

INPUT:
```sql
SELECT 
	customer_id ,
    SUM(price) AS total_price 
FROM dannys_diner.sales 
JOIN dannys_diner.menu
	ON dannys_diner.sales.product_id = dannys_diner.menu.product_id 
GROUP BY sales.customer_id;
```

OUTPUT:

| customer_id | total_price |
| ----------- | ----------- |
| B           | 74          |
| C           | 36          |
| A           | 76          |


**2. How many days has each customer visited the restaurant?**

INPUT:
```sql
SELECT
	customer_id,
	COUNT(DISTINCT order_date) AS day_come
FROM dannys_diner.sales
GROUP BY customer_id;
```

OUTPUT:

| customer_id | day_come |
| ----------- | -------- |
| A           | 4        |
| B           | 6        |
| C           | 2        |

3. What was the first item from the menu purchased by each customer?
INPUT:
```sql
SELECT 
    sales.customer_id, 
    sales.order_date, 
    menu.product_name,
    DENSE_RANK() OVER (PARTITION BY sales.customer_id 
      ORDER BY sales.order_date) AS rank_food
  FROM dannys_diner.sales
  INNER JOIN dannys_diner.menu
    ON sales.product_id = menu.product_id
)

SELECT customer_id, GROUP_CONCAT(product_name) AS first_buy FROM new_table_cte
WHERE rank_food=1
GROUP BY customer_id;
```
OUTPUT:
 customer_id |   first_buy   
------------+---------------
 A          | sushi,curry
 B          | curry
 C          | ramen


5. What is the most purchased item on the menu and how many times was it purchased by all customers?
6. Which item was the most popular for each customer?
7. Which item was purchased first by the customer after they became a member?
8. Which item was purchased just before the customer became a member?
9. What is the total items and amount spent for each member before they became a member?
10. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
11. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
