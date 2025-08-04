
# 🧠 SQL Hierarchical Relationships - FAANG Interview Guide

## 1. Core Concepts Covered (Detailed)

Hierarchical queries are essential in real-world applications like org charts, product categories, file systems, permissions, and resource access trees. These problems test your ability to model, traverse, and analyze parent-child relationships in SQL.

---

### 🔁 Recursive CTEs
- Recursive Common Table Expressions allow SQL to reference its own output.
- Used for traversing unknown-depth trees (e.g., manager → employee chains).
- **Key components:**
  - **Anchor member** (starting point)
  - **Recursive member** (joins back to the CTE itself)
  - **Termination** happens when no new rows are produced.

---

### 🌲 Tree Traversal (DFS, BFS)
- Simulate traversal using recursive CTEs.
- Add a `LEVEL` column to track depth.
- Add `PATH` column to track traversal order.

---

### 🧩 Path Aggregation
- Use string aggregation (e.g., `FOR XML PATH` in SQL Server or `STRING_AGG` in Postgres) to form a readable path.
- Useful for breadcrumb UIs or organization lineage.

---

### 🧮 Aggregate Calculations
- Hierarchical SUMs, COUNTs, AVG across subtrees.
- Important for computing metrics like total revenue per category or salary under a manager.

---

### 🔄 Cycle Detection
- Malformed data can lead to infinite loops in recursive CTEs.
- Prevent this by tracking visited nodes (e.g., using a path column).

---

### 🌳 Nested Set Model
- An alternative to adjacency lists.
- Represents trees using `lft` and `rgt` values.
- Faster for certain types of queries but harder to maintain on update.

---

### ⚠️ Orphan and Root Detection
- Orphan: Child whose parent doesn't exist.
- Root: Node with no parent.

---

### 🧠 FAANG-Level Additions
- Efficient traversal over large datasets.
- Use `materialized path`, `graph tables`, or `precomputed hierarchies`.
- Often expect clean, readable, and performant solutions.

---

## 2. Detailed Solutions + Query Patterns

[The full content continues with solutions for Q1 to Q36, as shown in the chat.]
