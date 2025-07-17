# SQL Window Functions Interview Guide (Beginner to Advanced)

This guide is crafted to help you prepare for **MAANG** and **top product-based company interviews** with a deep focus on **SQL Window Functions**. It includes an exhaustive list of real-world questions, explanations, and fully working solutions.

---

## What Are Window Functions?
Window functions perform calculations across a set of table rows that are somehow related to the current row. Unlike aggregate functions, they do not collapse rows.

---

## Table of Contents

- [Beginner-Level](#beginner-level)
- [Intermediate-Level](#intermediate-level)
- [Advanced-Level](#advanced-level)
- [Conceptual Interview](#conceptual-questions)

---

## Window Functions Deep Dive Questions & Solutions


## Beginner-Level

### 1. Running Total by Date
```sql
SELECT sale_date, amount,
       SUM(amount) OVER (ORDER BY sale_date) AS running_total
FROM sales;
```

### 2. Row Number by Group
```sql
SELECT customer_id, order_id, order_date,
       ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date) AS rn
FROM orders;
```

### 3. Rank Employees by Salary Within Department
```sql
SELECT department_id, employee_id, salary,
       RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rnk
FROM employees;
```

### 4. Second Highest Salary per Department
```sql
SELECT * FROM (
  SELECT department_id, employee_id, salary,
         DENSE_RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS drnk
  FROM employees
) t
WHERE drnk = 2;
```

### 5. Add Row Number
```sql
SELECT transaction_id, user_id, timestamp,
       ROW_NUMBER() OVER (ORDER BY timestamp) AS rownum
FROM transactions;
```

---

## Intermediate-Level Solutions

### 6. Previous Transaction Amount
```sql
SELECT user_id, transaction_id, amount,
       LAG(amount, 1) OVER (PARTITION BY user_id ORDER BY transaction_time) AS prev_amt
FROM transactions;
```

### 7. Next Event Timestamp
```sql
SELECT customer_id, event_time,
       LEAD(event_time) OVER (PARTITION BY customer_id ORDER BY event_time) AS next_event
FROM events;
```

### 8. 7-Day Moving Average
```sql
SELECT visit_date, revenue,
       AVG(revenue) OVER (ORDER BY visit_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS moving_avg
FROM daily_revenue;
```

### 9. Top 3 Products per Category
```sql
WITH ranked_products AS (
  SELECT category, product_id, SUM(sales) AS total_sales,
         RANK() OVER (PARTITION BY category ORDER BY SUM(sales) DESC) AS rnk
  FROM sales
  GROUP BY category, product_id
)
SELECT * FROM ranked_products
WHERE rnk <= 3;
```

### 10. Orders in Consecutive Months
```sql
WITH lagged_orders AS (
  SELECT user_id, order_month,
         LAG(order_month) OVER (PARTITION BY user_id ORDER BY order_month) AS prev_month
  FROM orders
)
SELECT * FROM lagged_orders
WHERE DATEDIFF(MONTH, prev_month, order_month) = 1;
```

### 11. Percentile Rank
```sql
SELECT student_id, score,
       PERCENT_RANK() OVER (ORDER BY score) AS prnk
FROM scores;
```

### 12. Ranked Retention
```sql
SELECT customer_id,
       MIN(visit_date) AS first,
       MAX(visit_date) AS last,
       DATEDIFF(DAY, MIN(visit_date), MAX(visit_date)) AS days_retained
FROM visits
GROUP BY customer_id;
```

### 13. First and Last Purchase Dates
```sql
SELECT customer_id, order_id, order_date,
       FIRST_VALUE(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS first_order,
       LAST_VALUE(order_date) OVER (PARTITION BY customer_id ORDER BY order_date 
          ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS last_order
FROM orders;
```

### 14. Days Since Last Login
```sql
SELECT user_id, login_date,
       DATEDIFF(DAY, LAG(login_date) OVER (PARTITION BY user_id ORDER BY login_date), login_date) AS days_diff
FROM logins;
```

### 15. Rank Stores by Monthly Growth
```sql
WITH growth AS (
  SELECT city, store_id, month, revenue,
         revenue - LAG(revenue) OVER (PARTITION BY store_id ORDER BY month) AS growth_amt
  FROM store_revenue
)
SELECT city, store_id, month, growth_amt,
       RANK() OVER (PARTITION BY city, month ORDER BY growth_amt DESC) AS growth_rank
FROM growth;
```

---

## Advanced-Level Solutions

### 16. Gaps in Daily Orders
```sql
SELECT customer_id, order_date,
       LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date) AS prev_order,
       DATEDIFF(DAY, LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date), order_date) AS gap
FROM orders;
```

### 17. New vs Returning
```sql
SELECT user_id, visit_time,
       CASE WHEN ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY visit_time) = 1 THEN 'New' ELSE 'Returning' END AS user_type
FROM visits;
```

### 18. Monthly Running Total
```sql
SELECT user_id, order_month, order_value,
       SUM(order_value) OVER (PARTITION BY user_id, order_month ORDER BY order_date) AS monthly_cumulative
FROM user_orders;
```

### 19. Longest Login Streak
```sql
WITH dates AS (
  SELECT user_id, login_date,
         ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) -
         DATEDIFF(DAY, '2000-01-01', login_date) AS grp
  FROM logins
)
SELECT user_id, COUNT(*) AS streak_len
FROM dates
GROUP BY user_id, grp
ORDER BY streak_len DESC;
```

### 20. Retention Cohorts
```sql
WITH cohort AS (
  SELECT user_id, MIN(order_date) AS cohort_month
  FROM orders
  GROUP BY user_id
), retention AS (
  SELECT o.user_id, c.cohort_month, o.order_date,
         DATEDIFF(MONTH, c.cohort_month, o.order_date) AS months_since_signup
  FROM orders o
  JOIN cohort c ON o.user_id = c.user_id
)
SELECT cohort_month, months_since_signup, COUNT(DISTINCT user_id) AS active_users
FROM retention
GROUP BY cohort_month, months_since_signup;
```

### 21. Sessionization
```sql
WITH events_with_gap AS (
  SELECT user_id, event_time,
         CASE WHEN DATEDIFF(MINUTE, LAG(event_time) OVER (PARTITION BY user_id ORDER BY event_time), event_time) > 30 THEN 1 ELSE 0 END AS new_session_flag
  FROM events
), sessions AS (
  SELECT *, SUM(new_session_flag) OVER (PARTITION BY user_id ORDER BY event_time) AS session_id
  FROM events_with_gap
)
SELECT * FROM sessions;
```

### 22. Order Deviation
```sql
WITH monthly_avg AS (
  SELECT customer_id, MONTH(order_date) AS mth, AVG(order_value) AS avg_val
  FROM orders
  GROUP BY customer_id, MONTH(order_date)
)
SELECT o.*, o.order_value - m.avg_val AS deviation
FROM orders o
JOIN monthly_avg m ON m.customer_id = o.customer_id AND MONTH(o.order_date) = m.mth;
```

### 23. Funnel Drop-off
```sql
SELECT user_id, session_id, page_name,
       ROW_NUMBER() OVER (PARTITION BY user_id, session_id ORDER BY timestamp) AS step
FROM pageviews;
```

### 24. Salary vs Department Avg
```sql
WITH dept_avg AS (
  SELECT department_id, AVG(salary) AS dept_avg
  FROM employees
  GROUP BY department_id
)
SELECT e.*, e.salary - d.dept_avg AS deviation
FROM employees e
JOIN dept_avg d ON e.department_id = d.department_id;
```

### 25. Running Unique Users
```sql
SELECT visit_date,
       COUNT(DISTINCT user_id) OVER (ORDER BY visit_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_distinct_users
FROM visits;
```

### 26. Quartiles Using NTILE
```sql
SELECT customer_id, total_spend,
       NTILE(4) OVER (ORDER BY total_spend DESC) AS quartile
FROM customer_spend;
```

### 27. Promo Effectiveness
```sql
WITH before_after AS (
  SELECT customer_id, order_date, order_value,
         CASE WHEN order_date < promo_date THEN 'Before' ELSE 'After' END AS phase
  FROM orders
)
SELECT phase, AVG(order_value) AS avg_order_value
FROM before_after
GROUP BY phase;
```

### 28. Frequent Purchase Time
```sql
SELECT user_id, EXTRACT(HOUR FROM order_time) AS hr,
       COUNT(*) AS cnt,
       RANK() OVER (PARTITION BY user_id ORDER BY COUNT(*) DESC) AS rnk
FROM orders
GROUP BY user_id, hr
HAVING rnk = 1;
```

### 29. Track Rank Change
```sql
WITH weekly_ranks AS (
  SELECT product_id, week, RANK() OVER (PARTITION BY week ORDER BY sales DESC) AS rank
  FROM weekly_sales
)
SELECT *, rank - LAG(rank) OVER (PARTITION BY product_id ORDER BY week) AS rank_diff
FROM weekly_ranks;
```

### 30. Aggregate vs Window Function
```sql
-- Aggregate version
SELECT department_id, SUM(salary) FROM employees GROUP BY department_id;

-- Window version
SELECT employee_id, department_id, salary,
       SUM(salary) OVER (PARTITION BY department_id) AS dept_total
FROM employees;
```

---

## Conceptual Questions

- What is the difference between `RANK()`, `DENSE_RANK()`, and `ROW_NUMBER()`?
- Why does `LAST_VALUE()` sometimes return unexpected results?
- What is the default frame for a window function?
- When would you use a window function instead of a join or subquery?





