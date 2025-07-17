
# 🚀 SQL Subquery Interview Prep Guide for FAANG & Product-Based Companies

## 🧠 Overview

This document covers **beginner to advanced SQL subquery-based interview questions**, optimized for FAANG and top product-based company interviews. Each scenario includes real-time context, performance tips, and multiple optimized solutions with detailed explanations.

---

## 📘 Table of Contents

1. [Beginner Subquery Questions](#beginner-subquery-questions)
2. [Intermediate Subquery Questions](#intermediate-subquery-questions)
3. [Advanced Subquery Questions](#advanced-subquery-questions)
4. [Follow-Up Interview Questions and Solutions](#follow-up-interview-questions-and-solutions)
5. [Real-Time Scenarios with Optimization](#real-time-scenarios-with-optimization)

---

## 🔰 Beginner Subquery Questions

### 1. Customers Who Placed an Order

```sql
SELECT CustomerID, CustomerName
FROM Customers
WHERE CustomerID IN (
    SELECT DISTINCT CustomerID FROM Orders
);
```

- **Purpose**: Find customers who have at least one order.
- **Pattern**: IN + Subquery.
- **Use Case**: Simple filter across relations.
- **FAANG Tip**: Compare with `EXISTS` for large tables.

---

### 2. Employees Earning More Than Average Salary

```sql
SELECT EmployeeID, Name, Salary
FROM Employees
WHERE Salary > (
    SELECT AVG(Salary) FROM Employees
);
```

- **Goal**: Compare values with scalar subquery.
- **Alternative**: Use window functions for ranking.

---

## 🟠 Intermediate Subquery Questions

### 3. Top 3 Expensive Products Per Category

```sql
SELECT *
FROM Products P1
WHERE 3 > (
    SELECT COUNT(*)
    FROM Products P2
    WHERE P2.CategoryID = P1.CategoryID
    AND P2.Price > P1.Price
);
```

- **Type**: Correlated subquery
- **Use**: Simulates ranking logic
- **Drawback**: Doesn’t scale well on large data

---

### 4. Customers Who Only Ever Ordered One Product

```sql
SELECT CustomerID
FROM Orders O
JOIN OrderItems OI ON O.OrderID = OI.OrderID
GROUP BY CustomerID
HAVING COUNT(DISTINCT OI.ProductID) = 1;
```

- **Purpose**: Ensure only one product purchased across all orders.
- **Alternative**:

```sql
SELECT CustomerID
FROM (
  SELECT O.CustomerID, COUNT(DISTINCT OI.ProductID) AS product_count
  FROM Orders O
  JOIN OrderItems OI ON O.OrderID = OI.OrderID
  GROUP BY O.CustomerID
) AS ProductSummary
WHERE product_count = 1;
```

---

## 🔴 Advanced Subquery Questions

### 5. Users Who Made First Purchase in 2024

```sql
SELECT UserID
FROM Transactions
WHERE TransactionDate = (
    SELECT MIN(TransactionDate)
    FROM Transactions T2
    WHERE T2.UserID = Transactions.UserID
)
AND YEAR(TransactionDate) = 2024;
```

---

### 6. Vendors Who Supply All Categories

```sql
SELECT VendorID
FROM Products
GROUP BY VendorID
HAVING COUNT(DISTINCT CategoryID) = (
    SELECT COUNT(DISTINCT CategoryID) FROM Products
);
```

---

### 7. Employees Who Earn the Second Highest Salary

#### ✅ Scalar Subquery

```sql
SELECT Name, Salary
FROM Employees
WHERE Salary = (
    SELECT MAX(Salary)
    FROM Employees
    WHERE Salary < (
        SELECT MAX(Salary) FROM Employees
    )
);
```

#### ✅ Optimized: LIMIT + OFFSET (PostgreSQL, MySQL)

```sql
SELECT Name, Salary
FROM Employees
WHERE Salary = (
    SELECT DISTINCT Salary
    FROM Employees
    ORDER BY Salary DESC
    LIMIT 1 OFFSET 1
);
```

#### ✅ Using DENSE_RANK (Recommended)

```sql
WITH RankedSalaries AS (
    SELECT Name, Salary,
           DENSE_RANK() OVER (ORDER BY Salary DESC) AS rank
    FROM Employees
)
SELECT Name, Salary
FROM RankedSalaries
WHERE rank = 2;
```

---

### 8. Customers Who Bought More Than Average Per Order

#### ✅ Original (Nested Subqueries)

```sql
SELECT CustomerID
FROM Orders O
JOIN OrderItems I ON O.OrderID = I.OrderID
GROUP BY CustomerID
HAVING SUM(Quantity) > (
    SELECT AVG(TotalQty)
    FROM (
        SELECT SUM(Quantity) AS TotalQty
        FROM Orders O2
        JOIN OrderItems I2 ON O2.OrderID = I2.OrderID
        GROUP BY O2.OrderID
    ) AS AvgPerOrder
);
```

#### ✅ FAANG-Optimized with CTEs

```sql
WITH OrderQuantities AS (
    SELECT O.OrderID, O.CustomerID, SUM(OI.Quantity) AS OrderQty
    FROM Orders O
    JOIN OrderItems OI ON O.OrderID = OI.OrderID
    GROUP BY O.OrderID, O.CustomerID
),
AvgPerOrder AS (
    SELECT AVG(OrderQty) AS AvgQty FROM OrderQuantities
),
CustomerTotals AS (
    SELECT CustomerID, SUM(OrderQty) AS TotalQty
    FROM OrderQuantities
    GROUP BY CustomerID
)
SELECT CT.CustomerID
FROM CustomerTotals CT, AvgPerOrder A
WHERE CT.TotalQty > A.AvgQty;
```

---

## 🧠 FAANG Interview Follow-Up Solutions

---

### Q1: Average Quantity Per Order Per Customer

```sql
WITH OrderQuantities AS (
    SELECT O.OrderID, O.CustomerID, SUM(OI.Quantity) AS OrderQty
    FROM Orders O
    JOIN OrderItems OI ON O.OrderID = OI.OrderID
    GROUP BY O.OrderID, O.CustomerID
)
SELECT CustomerID, AVG(OrderQty) AS AvgQtyPerOrder
FROM OrderQuantities
GROUP BY CustomerID;
```

---

### Q2: Rewrite Using Window Functions

```sql
SELECT DISTINCT CustomerID,
       SUM(SUM(Quantity)) OVER (PARTITION BY CustomerID) AS TotalQty,
       COUNT(DISTINCT OrderID) OVER (PARTITION BY CustomerID) AS TotalOrders,
       SUM(SUM(Quantity)) OVER (PARTITION BY CustomerID) * 1.0 /
       COUNT(DISTINCT OrderID) OVER (PARTITION BY CustomerID) AS AvgQtyPerOrder
FROM Orders O
JOIN OrderItems OI ON O.OrderID = OI.OrderID
GROUP BY CustomerID, OrderID;
```

---

### Q3: Deviation From Global Average Order Quantity

```sql
WITH OrderQuantities AS (
    SELECT O.OrderID, O.CustomerID, SUM(OI.Quantity) AS OrderQty
    FROM Orders O
    JOIN OrderItems OI ON O.OrderID = OI.OrderID
    GROUP BY O.OrderID, O.CustomerID
),
CustomerAvgPerOrder AS (
    SELECT CustomerID, AVG(OrderQty) AS CustomerAvgQty
    FROM OrderQuantities
    GROUP BY CustomerID
),
OverallAvg AS (
    SELECT AVG(OrderQty) AS GlobalAvgQty FROM OrderQuantities
)
SELECT C.CustomerID,
       C.CustomerAvgQty,
       O.GlobalAvgQty,
       C.CustomerAvgQty - O.GlobalAvgQty AS DeviationFromGlobalAvg
FROM CustomerAvgPerOrder C
CROSS JOIN OverallAvg O;
```

---
