# Maven_Pizza_Challenge
Tanles Details :

CREATE SCHEMA dannys_diner;
SET search_path = dannys_diner;

CREATE TABLE sales (
  "customer_id" VARCHAR(1),
  "order_date" DATE,
  "product_id" INTEGER
);

INSERT INTO sales
  ("customer_id", "order_date", "product_id")
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
  "product_id" INTEGER,
  "product_name" VARCHAR(5),
  "price" INTEGER
);

INSERT INTO menu
  ("product_id", "product_name", "price")
VALUES
  ('1', 'sushi', '10'),
  ('2', 'curry', '15'),
  ('3', 'ramen', '12');
  

CREATE TABLE members (
  "customer_id" VARCHAR(1),
  "join_date" DATE
);

INSERT INTO members
  ("customer_id", "join_date")
VALUES
  ('A', '2021-01-07'),
  ('B', '2021-01-09');
/* --------------------
   Case Study Questions
   --------------------*/

-- 1. What is the total amount each customer spent at the restaurant?

SELECT s.customer_id, SUM(m.price) AS total_amount
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id;

-- 2. How many days has each customer visited the restaurant?

SELECT s.customer_id, COUNT(DISTINCT s.order_date) AS visit_days
FROM sales s
JOIN members m ON s.customer_id = m.customer_id
GROUP BY s.customer_id;


-- 3. What was the first item from the menu purchased by each customer?

SELECT s.customer_id, MIN(m.product_name) AS first_product
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id;

-- 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

SELECT m.product_name,COUNT(*) AS purchase_count
FROM menu m 
INNER JOIN sales s m.product_id = s.product_id
Group BY m.product_id
ORDER BY purchase_count DESC
Limit 1;

-- 5. Which item was the most popular for each customer?

SELECT S.customer_id,M.product_name , purchase_count
FROM (
   SELECT S.customer_id,M.product_name, count(*) as purchase_count,
   ROW_NUMBER()OVER(PARTITION BY S.customer_id ORDER BY COUNT(*) DESC AS 
   popular_food
FROM sales S
JOIN menu M ON S.product_id = M.product_id
GROUP BY S.customer_id ) AS SUBQUERY
where popular_food=1 ;

-- SUBQUERY
  SELECT S.customer_id, M.product_name, COUNT(*) AS purchase_count
FROM sales S
JOIN menu M ON S.product_id = M.product_id
GROUP BY S.customer_id, M.product_name
HAVING COUNT(*) = (
  SELECT MAX(purchase_count)
  FROM (
    SELECT customer_id, COUNT(*) AS purchase_count
    FROM sales
    GROUP BY customer_id, product_id
  ) AS subquery
  WHERE S.customer_id = subquery.customer_id
)

-- 6. Which item was purchased first by the customer after they became a member?

SELECT s.customer_id, m.product_name, MIN(s.order_date) AS first_purchase_date
FROM sales s
JOIN menu m ON s.product_id = m.product_id
JOIN members mem ON s.customer_id = mem.customer_id
WHERE s.order_date >= mem.join_date
GROUP BY s.customer_id, m.product_name
ORDER BY s.customer_id, first_purchase_date ;


-- 7. Which item was purchased just before the customer became a member?


SELECT s.customer_id,m.product_name,MAX(s.order_date) AS last_purchased
FROM  sales s
JOIN members me ON s.customer_id = me.customer_id
JOIN menu m ON s.product_id = m.product_id
where s.order_date < mem.join_date
Group BY s.customer_id,m.product_name 
Limit 5;


-- 8. What is the total items and amount spent for each member before they became a member?
  
SELECT me.customer_id, COUNT(*) AS total_items, SUM(m.price) AS total_amount
FROM members me
JOIN sales s ON me.customer_id = s.customer_id
JOIN menu m ON s.product_id = m.product_id
WHERE s.order_date < me.join_date
GROUP BY me.customer_id;


-- 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

SELECT s.customer_id, SUM(
   CASE
      WHEN m.product_name = 'Sushi' THEN m.price * 20
      WHEN m.product_name = 'Curry' THEN m.price * 10
      ELSE 0
   END) AS totals
FROM sales s
JOIN menu m ON s.product_id = m.product_id
GROUP BY s.customer_id;



  
  
  
  
  
  
  
  
