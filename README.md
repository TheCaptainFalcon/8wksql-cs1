# SQL Case Study #1 (Week 1) - Danny's Diner

Note: All source material and respected credit is from: https://8weeksqlchallenge.com/

Online SQL instance used to test queries: https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138

## Table of contents:
1. [Dataset Structure](#data)
2. [Entity Relationship Diagram](#diagram)
3. [Case Study Questions + Answers](#questions)
4. [Bonus Questions + Answers](#bonus)

<div id='data'/>

## Dataset Structure:

Note: The original data was built around PostgreSQL, but was swapped to fit MySQL syntax.

** **
    CREATE SCHEMA dannys_diner;
    
    CREATE TABLE sales (
        customer_id VARCHAR(1),
        order_date DATE,
        product_id INTEGER
    );
    
    INSERT INTO sales
    
    VALUES
        ('A', '2021-01-01', '1'),
        ('A', '2021-01-01', '2'),
        ('A', '2021-01-07', '2'),
        ('A', '2021-01-10', '3'),
        ('A', '2021-01-11', '3'),
        ('A', '2021-01-11', '3'),
        ('B', '2021-01-01', '2'),
        ('B', '2021-01-02', '2'),
        ('B', '2021-01-04', '1'),
        ('B', '2021-01-11', '1'),
        ('B', '2021-01-16', '3'),
        ('B', '2021-02-01', '3'),
        ('C', '2021-01-01', '3'),
        ('C', '2021-01-01', '3'),
        ('C', '2021-01-07', '3');
        
    
    CREATE TABLE menu (
        product_id INTEGER,
        product_name VARCHAR(5),
        price INTEGER
    );
    
    INSERT INTO menu
    
    VALUES
        ('1', 'sushi', '10'),
        ('2', 'curry', '15'),
        ('3', 'ramen', '12');
        
    
    CREATE TABLE members (
        customer_id VARCHAR(1),
        join_date DATE
    );
    
    INSERT INTO members
    
    VALUES
        ('A', '2021-01-07'),
        ('B', '2021-01-09');

<div id='diagram'/>

## Entity Relationship Diagram View


Original image source: 
https://dbdiagram.io/d/608d07e4b29a09603d12edbd/?utm_source=dbdiagram_embed&utm_medium=bottom_open

![image](https://github.com/TheCaptainFalcon/8wksql-cs1/blob/master/ERD-CS1.JPG)

<div id='questions'/>

## Case Study Questions:

#### 1. What is the total amount each customer spent at the restaurant?

**Query #1**

    SELECT
        s.customer_id,
        SUM(m.price) AS amount_spent
    FROM sales s
    LEFT JOIN menu m ON s.product_id = m.product_id
    GROUP BY s.customer_id
    ORDER BY s.customer_id;

| customer_id | amount_spent |
| ----------- | ------------ |
| A           | 76           |
| B           | 74           |
| C           | 36           |

---

#### 2. How many days has each customer visited the restaurant?

**Query #2**

    SELECT 
        customer_id,
        COUNT(order_date) AS visit_count
    FROM sales
    GROUP BY customer_id;

| customer_id | visit_count |
| ----------- | ----------- |
| A           | 6           |
| B           | 6           |
| C           | 3           |

---

#### 3. What was the first item from the menu purchased by each customer?

**Query #3**

    SELECT
        customer_id,
        MIN(order_date),
        product_id
    FROM sales
    GROUP BY customer_id, product_id;

| customer_id | product_id | min(order_date) |
| ----------- | ---------- | --------------- |
| A           | 1          | 2021-01-01      |
| A           | 2          | 2021-01-01      |
| A           | 3          | 2021-01-10      |
| B           | 2          | 2021-01-01      |
| B           | 1          | 2021-01-04      |
| B           | 3          | 2021-01-16      |
| C           | 3          | 2021-01-01      |

#### Answer: 
 Customer A ordered product_id 1
>
 Customer B ordered product_id 2
>
 Customer C ordered product_id 3

---

#### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

**Query #4**

    SELECT 
        m.product_name, 
        COUNT(s.product_id) AS order_count
    FROM sales s
    LEFT JOIN menu m ON m.product_id = s.product_id 
    GROUP BY product_name
    ORDER BY order_count DESC;

| product_name | order_count |
| ------------ | ----------- |
| ramen        | 8           |
| curry        | 4           |
| sushi        | 3           |

#### Answer: 
Most purchased item is ramen and it was purchased 8 times total by all customers.

---

#### 5. Which item was the most popular for each customer?

**Query #5**

    SELECT
        s.customer_id, 
        m.product_name,
        COUNT(s.product_id) AS order_count
    FROM sales s
    LEFT JOIN menu m ON s.product_id = m.product_id
    GROUP BY s.customer_id, m.product_name
    ORDER BY s.customer_id, order_count DESC;

| customer_id | product_name | order_count |
| ----------- | ------------ | ----------- |
| A           | ramen        | 3           |
| A           | curry        | 2           |
| A           | sushi        | 1           |
| B           | curry        | 2           |
| B           | sushi        | 2           |
| B           | ramen        | 2           |
| C           | ramen        | 3           |

#### Answer: 

Customer A - Ramen, 
>
Customer B - Curry, 
>
Customer C - Ramen

---

#### 6. Which item was purchased first by the customer after they became a member?

**Query #6**

Solution 1:

    SELECT
        s.customer_id,
        s.order_date,
        mm.product_name,
        m.join_date
    FROM sales s 
    LEFT JOIN members m ON s.customer_id = m.customer_id
    INNER JOIN menu mm ON s.product_id = mm.product_id 
    WHERE order_date >= join_date
    ORDER BY s.customer_id, s.order_date;

| customer_id | order_date | product_name | join_date  |
| ----------- | ---------- | ------------ | ---------- |
| A           | 2021-01-07 | curry        | 2021-01-07 |
| A           | 2021-01-10 | ramen        | 2021-01-07 |
| A           | 2021-01-11 | ramen        | 2021-01-07 |
| A           | 2021-01-11 | ramen        | 2021-01-07 |
| B           | 2021-01-11 | sushi        | 2021-01-09 |
| B           | 2021-01-16 | ramen        | 2021-01-09 |
| B           | 2021-02-01 | ramen        | 2021-01-09 |

#### Answer: 

Customer A - Curry
>
Customer B - Sushi

---

#### 7. Which item was purchased just before the customer became a member?

**Query #7**

    SELECT
        s.customer_id,
        s.order_date,
        mm.product_name,
        m.join_date
    FROM sales s 
    LEFT JOIN members m ON s.customer_id = m.customer_id
    INNER JOIN menu mm ON s.product_id = mm.product_id
    WHERE order_date < join_date
    ORDER BY s.customer_id, s.order_date DESC;

| customer_id | order_date | product_name | join_date  |
| ----------- | ---------- | ------------ | ---------- |
| A           | 2021-01-01 | sushi        | 2021-01-07 |
| A           | 2021-01-01 | curry        | 2021-01-07 |
| B           | 2021-01-04 | sushi        | 2021-01-09 |
| B           | 2021-01-02 | curry        | 2021-01-09 |
| B           | 2021-01-01 | curry        | 2021-01-09 |

#### Answer: 

Customer A - Sushi
>
Customer B - Sushi

---

#### 8. What is the total items and amount spent for each member before they became a member?

**Query #8**

    SELECT
        s.customer_id,
        COUNT(s.product_id) AS products_purchased,
        SUM(mm.price) AS total_price
    FROM sales s 
    LEFT JOIN members m ON s.customer_id = m.customer_id
    INNER JOIN menu mm ON s.product_id = mm.product_id
    WHERE s.order_date < m.join_date
    GROUP BY s.customer_id
    ORDER BY customer_id;

| customer_id | products_purchased | total_price |
| ----------- | ------------------ | ----------- |
| A           | 2                  | 25          |
| B           | 3                  | 40          |

---

#### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

Note: First applicable use in a CTE (outside of tutorials and exercises).

**Query #9**

    WITH points_table AS
    (
    SELECT
        customer_id,
        product_id,
        CASE 
        	WHEN product_id = 1 THEN 20
        	WHEN product_id = 2 THEN 15
            WHEN product_id = 3 THEN 12
        END AS points
    FROM sales
    )
    
    SELECT
        customer_id,
        SUM(points) AS total_points
    FROM points_table
    GROUP BY customer_id;

| customer_id | total_points |
| ----------- | ------------ |
| A           | 86           |
| B           | 94           |
| C           | 36           |

For reference (per occurrence):

Sushi = 10(2) points
>
Curry = 15 points
>
Ramen = 12 points

---

#### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

**Query #10**

    WITH points_table AS
    (
    SELECT
        s.customer_id,
        s.product_id,
        CASE
        	WHEN s.product_id = 1 THEN 20
            WHEN s.product_id = 2 THEN 30
            WHEN s.product_id = 3 THEN 24
        END AS points
    FROM sales s 
    LEFT JOIN members m ON s.customer_id = m.customer_id
    WHERE s.order_date >= m.join_date AND s.order_date <= (m.join_date + 7)
    )
    
    SELECT
        customer_id,
        SUM(points) AS total_points
    FROM points_table
    GROUP BY s.customer_id;

| customer_id | total_points |
| ----------- | ------------ |
| A           | 102          |
| B           | 44           |

For reference (per occurrence):

Sushi = 10(2) points
>
Curry = 15(2) points
>
Ramen = 12(2) points

---
<div id='bonus'/>

## Bonus Question 1


#### 1. Purpose was to recreate a table output by reverse-engineering through queries.

**Query #1**

    SELECT
        s.customer_id,
        s.order_date, 
        mm.product_name,
        mm.price,
        CASE
        	WHEN s.order_date < m.join_date THEN 'N'
            WHEN m.join_date IS NULL THEN 'N'
            ELSE 'Y'
        END AS member
    FROM sales s
    LEFT JOIN menu mm ON s.product_id = mm.product_id
    LEFT JOIN members m ON s.customer_id = m.customer_id
    ORDER BY s.customer_id;

| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | ------------ | ----- | ------ |
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| B           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-16 | ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-07 | ramen        | 12    | N      |

---
## Bonus Question 2 

#### 2. Purpose was to apply ranks based on customer transactions after they became a member.

Note: First applicable use in a ranking function, over(), partitioning.

Important: Needed to use dense_rank and to include member (plus the customer_id) within the partition, otherwise the nulls will count as rank 1 (starting or continuing) even though they are considered an 'N'.

**Query #2**

    WITH joined_table AS 
    (
    SELECT
        s.customer_id,
        s.order_date, 
        mm.product_name,
        mm.price,
        CASE
        	WHEN s.order_date < m.join_date THEN 'N'
            WHEN m.join_date IS NULL THEN 'N'
            ELSE 'Y'
        END AS member
    FROM sales s
    LEFT JOIN menu mm ON s.product_id = mm.product_id
    LEFT JOIN members m ON s.customer_id = m.customer_id
    ORDER BY s.customer_id
    )
    
    SELECT
        customer_id,
        order_date,
        product_name,
        price,
        member, 
        CASE
        	WHEN member = 'N' THEN 'null'
            WHEN member = 'Y' THEN DENSE_RANK() OVER (PARTITION BY customer_id, member ORDER BY order_date) 
        END AS ranking
    FROM joined_table;

| customer_id | order_date | product_name | price | member | ranking |
| ----------- | ---------- | ------------ | ----- | ------ | ------- |
| A           | 2021-01-01 | sushi        | 10    | N      | null    |
| A           | 2021-01-01 | curry        | 15    | N      | null    |
| A           | 2021-01-07 | curry        | 15    | Y      | 1       |
| A           | 2021-01-10 | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01 | curry        | 15    | N      | null    |
| B           | 2021-01-02 | curry        | 15    | N      | null    |
| B           | 2021-01-04 | sushi        | 10    | N      | null    |
| B           | 2021-01-11 | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16 | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01 | ramen        | 12    | Y      | 3       |
| C           | 2021-01-01 | ramen        | 12    | N      | null    |
| C           | 2021-01-01 | ramen        | 12    | N      | null    |
| C           | 2021-01-07 | ramen        | 12    | N      | null    |

---
