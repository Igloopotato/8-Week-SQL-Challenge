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

- The sales table captures all customer_id level purchases with an corresponding order_date and product_id information for when and what menu items were ordered.
- The menu table maps the product_id to the actual product_name and price of each menu item.
- The final members table captures the join_date when a customer_id joined the beta version of the Danny’s Diner loyalty program.

## Entity Relationship Diagram
![ERD](https://github.com/marswanttobeanalyst/8-Week-SQL-Challenge/assets/141108687/8a65c7f6-eac8-4731-86d4-a5135d728581)

## Questions and Answers

**1. What is the total amount each customer spent at the restaurant?**

INPUT:
```sql
SELECT 
    customer_id,
    SUM(price) AS total_price
FROM  dannys_diner.sales
JOIN  dannys_diner.menu ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
GROUP BY sales.customer_id;
```

OUTPUT:
| customer_id | total_price |
| ----------- | ----------- |
| B           | 74          |
| C           | 36          |
| A           | 76          |


 ___

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


___

**3. What was the first item from the menu purchased by each customer?**

INPUT:
```sql
WITH sales_cte AS (
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

SELECT 
  customer_id, 
  STRING_AGG(DISTINCT product_name, ',') AS first_buy 
FROM sales_cte
WHERE rank_food = 1
GROUP BY customer_id;
```

OUTPUT:
| customer_id | first_buy    |
| ----------- | --------     |
| A           | sushi,curry  |
| B           | curry        |
| C           | ramen        | 


___

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**

INPUT:
```sql
SELECT
  product_name,
  COUNT(dannys_diner.sales.product_id) AS number_purchased
FROM dannys_diner.sales INNER
JOIN dannys_diner.menu
  ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
GROUP BY product_name
ORDER by number_purchased DESC
LIMIT 1;
```

OUTPUT:
| product_name | number_purchased |
| ------------ | ---------------- |
| ramen        | 8                |


___

**5. Which item was the most popular for each customer?**

INPUT:
```sql
WITH sales_cte AS 
	(
     SELECT
	customer_id, product_name, COUNT(dannys_diner.sales.product_id) AS number_purchased, 
        DENSE_RANK() OVER (PARTITION BY sales.customer_id ORDER BY COUNT(dannys_diner.sales.product_id) DESC) AS rank
     FROM dannys_diner.sales
     INNER JOIN dannys_diner.menu
        ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
     GROUP BY customer_id, product_name
     ORDER BY customer_id, number_purchased
    )

SELECT
  customer_id,
  STRING_AGG(DISTINCT product_name, ',') AS most_buy,
  MAX(number_purchased) as most_purchased
FROM sales_cte
WHERE rank = 1
GROUP BY customer_id;
```

OUTPUT:
| customer_id | most_buy          | most_purchased |
| ----------- | ----------------- | -------------- |
| A           | ramen             | 3              |
| B           | curry,ramen,sushi | 2              |
| C           | ramen             | 3              |


___

**6. Which item was purchased first by the customer after they became a member?**

INPUT:
```sql
WITH members_cte AS
 (
 SELECT
   members.customer_id, sales.product_id, order_date,
   ROW_NUMBER() OVER (PARTITION BY members.customer_id ORDER BY sales.order_date) AS row_num
 FROM dannys_diner.members
 INNER JOIN dannys_diner.sales
   ON members.customer_id = sales.customer_id
   AND sales.order_date >= members.join_date
  )

SELECT
  customer_id, menu.product_name, order_date
FROM members_cte
INNER JOIN dannys_diner.menu
  ON new_table.product_id = menu.product_id
WHERE row_num = 1
ORDER BY customer_id;
```

OUTPUT:
| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| B           | sushi        |


___

**7. Which item was purchased just before the customer became a member?**

INPUT:
```sql
WITH members_cte AS
 (
 SELECT
   members.customer_id, sales.product_id, order_date,
   DENSE_RANK() OVER (PARTITION BY members.customer_id ORDER BY sales.order_date DESC)  AS row_num
 FROM dannys_diner.members
 INNER JOIN dannys_diner.sales
   ON members.customer_id = sales.customer_id
   AND sales.order_date < members.join_date
 )

SELECT
  customer_id, order_date,
  STRING_AGG(product_name, ',') AS product_purchased
FROM members_cte
INNER JOIN dannys_diner.menu
  ON new_table.product_id = menu.product_id
WHERE row_num = 1
GROUP BY customer_id, order_date
ORDER BY customer_id;
```

OUTPUT:
| customer_id | order_date               | product_purchased |
| ----------- | ------------------------ | ----------------- |
| A           | 2021-01-01T00:00:00.000Z | sushi,curry       |
| B           | 2021-01-04T00:00:00.000Z | sushi             |


___

**8. What is the total items and amount spent for each member before they became a member?**

INPUT:
```sql
WITH members_cte AS
 (
 SELECT
   members.customer_id, sales.product_id, order_date,
   ROW_NUMBER() OVER (PARTITION BY members.customer_id ORDER BY sales.order_date) AS row_num
 FROM dannys_diner.members
 INNER JOIN dannys_diner.sales
   ON members.customer_id = sales.customer_id
   AND sales.order_date < members.join_date
 )

SELECT
  new_table.customer_id,
  COUNT(new_table.product_id) AS total_items,
  SUM(price) AS total_spent
FROM members_cte
INNER JOIN dannys_diner.menu
  ON new_table.product_id = menu.product_id
GROUP BY new_table.customer_id
ORDER BY new_table.customer_id;
```

OUTPUT:

| customer_id | total_items | total_spent |
| ----------- | ----------- | ----------- |
| A           | 2           | 25          |
| B           | 3           | 40          |


___

**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

There are two scenario that I will think of for this where first scenario where points counted even when they not the member yet and second scenario where point counted only after they become member.

+ ___A) SCENARIO 1 (WHEN WE CONSIDER THAT THE POINTS WILL BE COUNTED FROM THE VERY FIRST PURCHASE INSTEAD OF WHEN THEY STARTED TO JOIN THE MEMBERSHIP)___

INPUT:
```sql
SELECT
  customer_id,
  SUM
     (CASE
     WHEN product_name = 'sushi'  THEN price*20
     ELSE price*10
     END) AS points      
  FROM dannys_diner.sales
  JOIN dannys_diner.menu
     ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
  GROUP BY customer_id
  ORDER BY customer_id
```

OUTPUT:

| customer_id | points |
| ----------- | ------ |
| A           | 860    |
| B           | 940    |
| C           | 360    |






+ ___B) SCENARIO 2 (WHEN WE CONSIDER THAT THE POINTS ONLY BE COUNTED WHEN THEY STARTED TO JOIN THE MEMBERSHIP)___

INPUT:
```sql
WITH sales_cte AS
 (
  SELECT
    members.customer_id, order_date, product_id
  FROM dannys_diner.sales
  INNER JOIN dannys_diner.members
    ON sales.customer_id = members.customer_id
    AND order_date >= join_date 
 )

SELECT
  customer_id,
  SUM
    (CASE
    WHEN product_name = 'sushi' THEN price*20
    ELSE price*10 END)
FROM sales_cte
INNER JOIN dannys_diner.menu
  ON new_table.product_id = dannys_diner.menu.product_id
GROUP BY customer_id
ORDER BY customer_id;
```

OUTPUT:
| customer_id | sum |
| ----------- | --- |
| A           | 510 |
| B           | 440 |


___

**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

I really like this assumption made by [Katie Huang](https://www.linkedin.com/in/katiehuangx/)

> -  On Day -X to Day 1 (the day a customer becomes a member), each $1 spent earns 10 points. However, for sushi, each $1 spent earns 20 points.
> -  From Day 1 to Day 7 (the first week of membership), each $1 spent for any items earns 20 points.
> -  From Day 8 to the last day of January 2021, each $1 spent earns 10 points. However, sushi continues to earn double the points at 20 points per $1 spent.

Hence, I decided to make two scenario out of this as well.

+ ___A) SCENARIO 1 (WHEN WE CONSIDER THAT THE POINTS WILL BE COUNTED FROM THE VERY FIRST PURCHASE INSTEAD OF WHEN THEY STARTED TO JOIN THE MEMBERSHIP)___

INPUT: 
```sql
WITH date_cte AS 
  (
   SELECT
     customer_id, join_date,
     join_date + INTERVAL '6 DAY' AS last_joined_date
   FROM dannys_diner.members
  )

SELECT 
 sales.customer_id,
 SUM
  (CASE 
  WHEN product_name = 'sushi'  THEN 2*10* price
  WHEN order_date BETWEEN join_date AND last_joined_date THEN 2*10*price
  ELSE 10*price END) AS points
FROM dannys_diner.sales 
INNER JOIN dannys_diner.menu
  ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
INNER JOIN date_cte
  ON dannys_diner.sales.customer_id = date_cte.customer_id
  AND order_date <='2021-01-31'
GROUP BY sales.customer_id;
```

OUTPUT:

| customer_id | points |
| ----------- | ------ |
| A           | 1370   |
| B           | 820    |






+ ___B) SCENARIO 2 (WHEN WE CONSIDER THAT THE POINTS ONLY BE COUNTED WHEN THEY STARTED TO JOIN THE MEMBERSHIP)___

INPUT: 
```sql
WITH date_cte AS 
  (
   SELECT
     customer_id, join_date,
     join_date + INTERVAL '6 DAY' AS last_joined_date
   FROM dannys_diner.members
  )

SELECT 
 sales.customer_id,
 SUM
   (CASE 
   WHEN product_name = 'sushi'  THEN 2*10* price
   WHEN order_date BETWEEN join_date AND last_joined_date THEN 2*10*price
   ELSE 10*price END) AS points
FROM dannys_diner.sales 
INNER JOIN dannys_diner.menu
  ON dannys_diner.sales.product_id = dannys_diner.menu.product_id
INNER JOIN date_cte
  ON dannys_diner.sales.customer_id = date_cte.customer_id
  AND order_date >= join_date 
  AND order_date <='2021-01-31'
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```

OUTPUT:

| customer_id | points |
| ----------- | ------ |
| A           | 1020   |
| B           | 320    |
