# SQL Case Study #1 (Week 1) - Danny's Diner

Note: All source material and respected credit is from: https://8weeksqlchallenge.com/

Online SQL instance used to test queries: https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138

## Table of contents:
1. [Dataset Structure](#data)
2. [Entity Relationship Diagram](#diagram)
3. [Case Study Questions + Answers](#questions)
4. [Bonus Questions + Answers](#bonus)

## Dataset Structure:

<div id='data'/>

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

![image](ERD-CS1.jpg)

<div id='questions'/>

## Case Study Questions:

#### 1. What is the total amount each customer spent at the restaurant?

**Query #1**

    select
    	s.customer_id,
        sum(m.price) as amount_spent
    from sales s
    left join menu m on s.product_id = m.product_id
    group by s.customer_id
    order by s.customer_id;

| customer_id | amount_spent |
| ----------- | ------------ |
| A           | 76           |
| B           | 74           |
| C           | 36           |

---

#### 2. How many days has each customer visited the restaurant?

**Query #2**

    select 
    	customer_id,
        count(order_date) as visit_count
    from sales
    group by customer_id;

| customer_id | visit_count |
| ----------- | ----------- |
| A           | 6           |
| B           | 6           |
| C           | 3           |

---

#### 3. What was the first item from the menu purchased by each customer?

**Query #3**

    select
    	customer_id,
        min(order_date),
        product_id
    from sales
    group by customer_id, product_id;

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
Customer B ordered product_id 2
Customer C ordered product_id 3

---

#### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

**Query #4**

    select 
        m.product_name, 
        count(s.product_id) as order_count
    from sales s
    left join menu m on m.product_id = s.product_id 
    group by product_name
    order by order_count desc;

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

    select 
        s.customer_id, 
        m.product_name, 
        count(s.product_id) as order_count
    from sales s
        left join menu m on s.product_id = m.product_id
        group by s.customer_id, m.product_name
        order by s.customer_id, order_count desc;

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
Customer B - Curry, 
Customer C - Ramen

---

#### 6. Which item was purchased first by the customer after they became a member?

**Query #6**

Solution 1:

    select
    	s.customer_id,
        s.order_date,
        mm.product_name,
        m.join_date
    from sales s 
    left join members m on s.customer_id = m.customer_id
    inner join menu mm on s.product_id = mm.product_id 
    where order_date >= join_date
    order by s.customer_id, s.order_date;

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
Customer B - Sushi

---

#### 7. Which item was purchased just before the customer became a member?

**Query #7**

    select
    	s.customer_id,
        s.order_date,
        mm.product_name,
        m.join_date
    from sales s 
    left join members m on s.customer_id = m.customer_id
    inner join menu mm on s.product_id = mm.product_id
    where order_date < join_date
    order by s.customer_id, s.order_date desc;

| customer_id | order_date | product_name | join_date  |
| ----------- | ---------- | ------------ | ---------- |
| A           | 2021-01-01 | sushi        | 2021-01-07 |
| A           | 2021-01-01 | curry        | 2021-01-07 |
| B           | 2021-01-04 | sushi        | 2021-01-09 |
| B           | 2021-01-02 | curry        | 2021-01-09 |
| B           | 2021-01-01 | curry        | 2021-01-09 |

#### Answer: 

Customer A - Sushi
Customer B - Sushi

---

#### 8. What is the total items and amount spent for each member before they became a member?

**Query #8**

    select
    	s.customer_id,
        count(s.product_id) as products_purchased,
        sum(mm.price) as total_price
    from sales s 
    left join members m on s.customer_id = m.customer_id
    inner join menu mm on s.product_id = mm.product_id
    where s.order_date < m.join_date
    group by s.customer_id
    order by customer_id;

| customer_id | products_purchased | total_price |
| ----------- | ------------------ | ----------- |
| A           | 2                  | 25          |
| B           | 3                  | 40          |

---

#### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

Note: First applicable use in a CTE (outside of tutorials and exercises).

**Query #9**

    with points_table as 
    (
    select 
    	customer_id,
        product_id,
        case 
        	when product_id = 1 then 20
        	when product_id = 2 then 15
            when product_id = 3 then 12
        end as points
    from sales
    )
    
    select 
    	customer_id,
        sum(points) as total_points
    from points_table
    group by customer_id;

| customer_id | total_points |
| ----------- | ------------ |
| A           | 86           |
| B           | 94           |
| C           | 36           |

For reference (per occurrence):

Sushi = 10(2) points
Curry = 15 points
Ramen = 12 points

---

#### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

**Query #10**

    with points_table as
    (
    select
    	s.customer_id,
        s.product_id,
        case
        	when s.product_id = 1 then 20
            when s.product_id = 2 then 30
            when s.product_id = 3 then 24
        end as points
    from sales s 
    left join members m on s.customer_id = m.customer_id
    where s.order_date >= m.join_date and s.order_date <= (m.join_date + 7)
    )
    
    select 
    	customer_id,
        sum(points) as total_points
    from points_table
    group by s.customer_id;

| customer_id | total_points |
| ----------- | ------------ |
| A           | 102          |
| B           | 44           |

For reference (per occurrence):

Sushi = 10(2) points
Curry = 15(2) points
Ramen = 12(2) points

---
<div id='bonus'/>

## Bonus Question 1


#### 1. Purpose was to recreate a table output by reverse-engineering through queries.

**Query #1**

    select 
    	s.customer_id,
        s.order_date, 
        mm.product_name,
        mm.price,
        case
        	when s.order_date < m.join_date then 'N'
            when m.join_date is null then 'N'
            else 'Y'
        end as member
    from sales s
    left join menu mm on s.product_id = mm.product_id
    left join members m on s.customer_id = m.customer_id
    order by s.customer_id

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

    with joined_table as 
    (
    select 
    	s.customer_id,
        s.order_date, 
        mm.product_name,
        mm.price,
        case
        	when s.order_date < m.join_date then 'N'
            when m.join_date is null then 'N'
            else 'Y'
        end as member
    from sales s
    left join menu mm on s.product_id = mm.product_id
    left join members m on s.customer_id = m.customer_id
    order by s.customer_id
    )
    select 
    	customer_id,
        order_date,
        product_name,
        price,
        member, 
        case
        	when member = 'N' then 'null'
            when member = 'Y' then dense_rank() over (partition by customer_id, member order by order_date) 
        end as ranking
    from joined_table

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