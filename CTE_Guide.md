
# SQL Server CTE Interview Questions With Real-Time Scenarios

## 1. Coding Interview Questions

### Q1: Find each employee’s department and salary alongside their department’s average salary

**Scenario:** A company wants to analyze salary vs department average.

```sql
WITH avg_salary AS (
    SELECT department, AVG(salary) AS average_salary
    FROM employees
    GROUP BY department
)
SELECT
    e.first_name,
    e.last_name,
    e.department,
    e.salary,
    avgs.average_salary
FROM employees AS e
JOIN avg_salary AS avgs
  ON e.department = avgs.department
ORDER BY e.department;
```

**Use Case:** Simplifies salary comparisons across departments.

**Performance Note:** Use CTE when computation is lightweight or referenced once.

---

### Q2: Highest-salary employee per department

```sql
WITH highest_salary AS (
    SELECT *,
           RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS salary_rank
    FROM employees
)
SELECT first_name, last_name, department, salary
FROM highest_salary
WHERE salary_rank = 1;
```

**Use Case:** Identify top performers by department.

**Performance Note:** Indexes on department/salary help window function speed.

---

### Q3: Find all subordinates (direct and indirect) of a manager

```sql
WITH subordinates AS (
    SELECT id, first_name, last_name, manager_id
    FROM employees
    WHERE id = 18
    UNION ALL
    SELECT e.id, e.first_name, e.last_name, e.manager_id
    FROM employees e
    JOIN subordinates s ON e.manager_id = s.id
)
SELECT * FROM subordinates
WHERE id <> 18;
```

**Use Case:** Hierarchical data traversal.

**Performance Note:** Recursive CTEs are powerful but watch out for MAXRECURSION.

---

## 2. Theoretical / Architectural-Level Questions

### Q6: What is a CTE?

- Temporary result set defined within a query using `WITH`
- Improves readability, supports recursion, and breaks complex logic into parts

### Q7: Are CTEs materialized?

- **No**, SQL Server expands the CTE at runtime like an inline view.
- Use temp tables if reuse or indexing is needed.

### Q8: CTE vs Temp Table vs View

| Feature         | CTE         | Temp Table | View         |
|----------------|-------------|------------|--------------|
| Scope          | Single query| Session     | Global       |
| Storage        | In-memory   | Disk        | None (unless indexed) |
| Reusability    | ❌           | ✅           | ✅            |
| Indexable      | ❌           | ✅           | Only indexed views |

### Q9: Can I index a CTE?

- No. But you can index underlying tables or use temp tables if needed.

### Q10: CTE Limitations

- Single-use
- Not indexable
- Cannot persist across batches
- Recursive CTEs limited to 100 levels by default (`OPTION (MAXRECURSION)`)

---

## 3. Advanced Query Scenario: Consecutive Order Days

```sql
WITH groupings_by_date AS (
    SELECT
        customer_id,
        order_date,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date) - 
        DATEDIFF(day, 0, order_date) AS grp
    FROM orders
)
SELECT customer_id, COUNT(*) AS streak_length
FROM groupings_by_date
GROUP BY customer_id, grp;
```

**Use Case:** Detect streaks or gaps in activity.

**Performance Tip:** Use covering index on `(customer_id, order_date)` for performance.

---

## ✅ Bonus: Recursive CTE Max Depth

```sql
OPTION (MAXRECURSION 0); -- removes default 100 level limit
```

**Use Case:** Deep hierarchy traversal (e.g. company org, file system, etc.)

**Warning:** Ensure stopping condition to avoid infinite loops.

---

**End of Guide**
