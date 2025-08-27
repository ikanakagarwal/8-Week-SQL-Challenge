# 🍜 Case Study #1 - Danny’s Diner

This folder contains my Week 1 case study solutions for the [8 Week SQL Challenge](https://8weeksqlchallenge.com/).

🔍 **What this case study is about**:
- Analyze customer visit frequency
- Track menu item purchases
- Understand customer spending behavior


## 📑 Solutions 
### 1. What is the total amount each customer spent at the restaurant?  

```sql
SELECT customer_id , SUM(price) AS total_amount 
FROM (
    SELECT s.customer_id, m.price 
    FROM sales s 
    LEFT JOIN menu m 
      ON s.product_id = m.product_id
) AS m 
GROUP BY customer_id;
```

### 2. How many days has each customer visited the restaurant?
```sql
SELECT customer_id, COUNT(DISTINCT order_date) AS number_of_days_visited
FROM sales
GROUP BY customer_id;
```

### 3. What was the first item from the menu purchased by each customer?
```sql
SELECT customer_id, product_name
FROM (
    SELECT s.customer_id,
           m.product_name,
           ROW_NUMBER() OVER (
               PARTITION BY s.customer_id ORDER BY s.order_date
           ) AS rn
    FROM sales s
    JOIN menu m
      ON s.product_id = m.product_id
) ranked
WHERE rn = 1;
```

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
SELECT m.product_name AS most_purchased_item, s.cn AS no_of_times_purchased 
FROM menu m 
JOIN (
        SELECT product_id, COUNT(*) AS cn
        FROM sales 
        GROUP BY product_id 
        ORDER BY COUNT(*) DESC
        LIMIT 1
) AS s
ON m.product_id = s.product_id;
```

### 5. Which item was the most popular for each customer?
```sql
SELECT customer_id, product_name, total_orders
FROM (
    SELECT s.customer_id,
           m.product_name,
           s.product_id,
           COUNT(*) AS total_orders,
           ROW_NUMBER() OVER (
               PARTITION BY s.customer_id
               ORDER BY COUNT(*) DESC, s.product_id ASC
           ) AS rn
    FROM sales s
    JOIN menu m 
      ON s.product_id = m.product_id
    GROUP BY s.customer_id, m.product_name, s.product_id
) ranked
WHERE rn = 1;
```

### 6. Which item was purchased first by the customer after they became a member?
```sql
WITH joined_as_member AS (
  SELECT
    members.customer_id, 
    sales.product_id,
    ROW_NUMBER() OVER (
      PARTITION BY members.customer_id
      ORDER BY sales.order_date) AS row_num
  FROM dannys_diner.members
  INNER JOIN dannys_diner.sales
    ON members.customer_id = sales.customer_id
    AND sales.order_date > members.join_date
)

SELECT 
  customer_id, 
  product_name 
FROM joined_as_member
INNER JOIN dannys_diner.menu
  ON joined_as_member.product_id = menu.product_id
WHERE row_num = 1
ORDER BY customer_id ASC;
```
### 7. Which item was purchased just before the customer became a member?
```sql
WITH purchased_prior_member AS (
  SELECT 
    members.customer_id, 
    sales.product_id,
    ROW_NUMBER() OVER (
      PARTITION BY members.customer_id
      ORDER BY sales.order_date DESC) AS rank
  FROM dannys_diner.members
  INNER JOIN dannys_diner.sales
    ON members.customer_id = sales.customer_id
    AND sales.order_date < members.join_date
)

SELECT 
  p_member.customer_id, 
  menu.product_name 
FROM purchased_prior_member AS p_member
INNER JOIN dannys_diner.menu
  ON p_member.product_id = menu.product_id
WHERE rank = 1
ORDER BY p_member.customer_id ASC;
```
### 8. What is the total items and amount spent for each member before they became a member?
```sql
SELECT 
  sales.customer_id, 
  COUNT(sales.product_id) AS total_items, 
  SUM(menu.price) AS total_sales
FROM dannys_diner.sales
INNER JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
  AND sales.order_date < members.join_date
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```

### 9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
WITH points_cte AS (
  SELECT 
    menu.product_id, 
    CASE
      WHEN product_id = 1 THEN price * 20
      ELSE price * 10 END AS points
  FROM dannys_diner.menu
)

SELECT 
  sales.customer_id, 
  SUM(points_cte.points) AS total_points
FROM dannys_diner.sales
INNER JOIN points_cte
  ON sales.product_id = points_cte.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```
### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql
WITH dates_cte AS (
  SELECT 
    customer_id, 
      join_date, 
      join_date + 6 AS valid_date, 
      DATE_TRUNC(
        'month', '2021-01-31'::DATE)
        + interval '1 month' 
        - interval '1 day' AS last_date
  FROM dannys_diner.members
)

SELECT 
  sales.customer_id, 
  SUM(CASE
    WHEN menu.product_name = 'sushi' THEN 2 * 10 * menu.price
    WHEN sales.order_date BETWEEN dates.join_date AND dates.valid_date THEN 2 * 10 * menu.price
    ELSE 10 * menu.price END) AS points
FROM dannys_diner.sales
INNER JOIN dates_cte AS dates
  ON sales.customer_id = dates.customer_id
  AND dates.join_date <= sales.order_date
  AND sales.order_date <= dates.last_date
INNER JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id;
```




