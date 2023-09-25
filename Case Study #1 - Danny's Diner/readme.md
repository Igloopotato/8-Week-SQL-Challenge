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

1. What is the total amount each customer spent at the restaurant?
2. ow many days has each customer visited the restaurant?
3. What was the first item from the menu purchased by each customer?
4. What is the most purchased item on the menu and how many times was it purchased by all customers?
5. Which item was the most popular for each customer?
6. Which item was purchased first by the customer after they became a member?
7. Which item was purchased just before the customer became a member?
8. What is the total items and amount spent for each member before they became a member?
9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
