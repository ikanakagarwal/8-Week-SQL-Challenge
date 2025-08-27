# 🍜 Case Study #1 - Danny’s Diner

This folder contains my Week 1 case study solutions for the [8 Week SQL Challenge](https://8weeksqlchallenge.com/).

🔍 **What this case study is about**:
- Analyze customer visit frequency
- Track menu item purchases
- Understand customer spending behavior

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

