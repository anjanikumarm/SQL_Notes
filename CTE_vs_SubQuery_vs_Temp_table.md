# Understanding CTEs vs Subqueries vs Temporary Tables in SQL

> Author: [Your Name]  
> Last Updated: July 2025

---

## 📘 Overview
This guide provides a **deep understanding of CTEs (Common Table Expressions), Subqueries, and Temporary Tables in SQL** — including their purpose, behavior (materialization), and real-world use cases. Designed for **data engineering interviews**, especially at **FAANG** and top product-based companies.

---

## 🔁 Table of Contents
1. [CTE (Common Table Expression)](#cte-common-table-expression)
2. [Subquery](#subquery)
3. [Temporary Table](#temporary-table)
4. [Materialization Explained](#materialization-explained)
5. [Comparative Summary](#comparative-summary)
6. [Interview Talking Points](#interview-talking-points)

---

## CTE (Common Table Expression)

### ✅ Purpose
- Simplify and modularize complex SQL queries.
- Enable **recursive queries** for hierarchical data.

### ✅ Syntax
```sql
WITH cte_name AS (
    SELECT ...
)
SELECT * FROM cte_name;
```

### ✅ Example: Top 3 Highest-Paid Employees per Department
```sql
WITH RankedSalaries AS (
    SELECT 
        department_id,
        employee_id,
        salary,
        RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS salary_rank
    FROM employees
)
SELECT *
FROM RankedSalaries
WHERE salary_rank <= 3;
```

### 🔁 Recursive CTE: Organizational Hierarchy
```sql
WITH RECURSIVE OrgHierarchy AS (
    SELECT employee_id, manager_id, employee_name
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    SELECT e.employee_id, e.manager_id, e.employee_name
    FROM employees e
    INNER JOIN OrgHierarchy oh ON e.manager_id = oh.employee_id
)
SELECT * FROM OrgHierarchy;
```

---

## Subquery

### ✅ Purpose
- Perform inline filtering, aggregation, or transformation within a query.

### ✅ Example: Employees with Salary > Average
```sql
SELECT employee_id, name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

### 🔁 Correlated Subquery
```sql
SELECT employee_id, name, department_id, salary
FROM employees e
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
    WHERE department_id = e.department_id
);
```

### 🔁 Subquery in FROM Clause (Derived Table)
```sql
SELECT department_id, AVG(salary) AS avg_salary
FROM (
    SELECT department_id, salary
    FROM employees
    WHERE status = 'active'
) AS filtered
GROUP BY department_id;
```

---

## Temporary Table

### ✅ Purpose
- Store intermediate results for **reuse across multiple queries**.
- Suitable for **large datasets and ETL steps**.

### ✅ Syntax & Example
```sql
CREATE TEMPORARY TABLE recent_sales AS
SELECT *
FROM sales
WHERE sale_date >= CURRENT_DATE - INTERVAL '30 days';
```

### ✅ Reuse Across Queries
```sql
-- Product-level sales
SELECT product_id, SUM(amount) AS total_sales
FROM recent_sales
GROUP BY product_id;

-- Region-level sales
SELECT region, SUM(amount) AS total_sales
FROM recent_sales
GROUP BY region;
```

### 🔁 Indexing Example
```sql
CREATE INDEX idx_product ON recent_sales(product_id);
```

---

## Materialization Explained

### 🔎 Definition
**Materialization** means a result set is **physically stored** (in memory/disk) before being used in the main query.

### 🔍 Materialization by Construct
| Construct        | Materialized?                            | Notes                                                  |
|------------------|-------------------------------------------|--------------------------------------------------------|
| CTE              | ❓ Depends on DB engine/version           | PostgreSQL 12+ allows `MATERIALIZED` or `NOT MATERIALIZED` |
| Subquery         | ❌ Typically not                         | Inlined unless in FROM clause                          |
| Temporary Table  | ✅ Always                                | Physically stored, can be indexed                      |

### 🔁 PostgreSQL Example
```sql
WITH my_cte AS MATERIALIZED (
  SELECT * FROM big_table WHERE status = 'active'
)
SELECT * FROM my_cte;
```

---

## Comparative Summary

| Feature             | CTE                             | Subquery                         | Temporary Table                  |
|---------------------|----------------------------------|----------------------------------|----------------------------------|
| Scope               | Single query                    | Inline                          | Session/transaction             |
| Reusable            | Yes (in same query)             | No                               | Yes                              |
| Recursive Support   | ✅ Yes                           | ❌ No                            | ❌ No                            |
| Materialized        | Sometimes                       | Rarely                          | ✅ Always                        |
| Optimizer Control   | Some DBs allow hinting          | Minimal                         | Full control                    |
| Use Case            | Modularity, hierarchy            | Filters, inline calcs            | ETL, reuse, staging              |
| Interview Edge Case | Recursive CTE, multiple uses    | Correlated vs uncorrelated       | Indexing, multi-step pipelines   |

---

## Interview Talking Points

- **“When do you use a CTE?”**
  > For modular queries, readability, and recursion (like org charts).

- **“What are the drawbacks of correlated subqueries?”**
  > They execute row-by-row, and can be slow — I’d refactor with CTEs or joins.

- **“Why use a temp table?”**
  > To stage data for multi-step processes, and optimize performance across queries.

- **“What is materialization in SQL?”**
  > It's when an intermediate result is physically stored before use. CTEs may be materialized, subqueries usually are not, temp tables always are.

---

## Optimization Tips
- Always test if your DB **inlines or materializes CTEs**.
- Use **temporary tables** in stored procedures for **large data reuse**.
- Replace **correlated subqueries** with **joins or CTEs** when performance is an issue.
- Avoid overusing **CTEs inside loops** in procedures — use temp tables instead.

