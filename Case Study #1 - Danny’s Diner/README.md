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

###2. How many days has each customer visited the restaurant?
```sql
SELECT customer_id, COUNT(DISTINCT order_date) AS number_of_days_visited
FROM sales
GROUP BY customer_id;
```

###3. What was the first item from the menu purchased by each customer?
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

###4. What is the most purchased item on the menu and how many times was it purchased by all customers?
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

###5. Which item was the most popular for each customer?
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

###6. Which item was purchased first by the customer after they became a member?
```sql
```
###7. Which item was purchased just before the customer became a member?
```sql
```
###8. What is the total items and amount spent for each member before they became a member?
```sql
```

###9.  If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
```
###10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql
```




