# 7. Indexing and Query Performance Basics

As datasets grow, query performance becomes increasingly important. Understanding indexes and querying optimization techniques can drastically improve how efficiently a database retrieves data.

# Tables Used in This Lesson

`orders`

| `order_id` | `customer_id` | `order_amount` | `order_date` | `status` |
| --- | --- | --- | --- | --- |
| 101 | 1 | 250.75 | 2024-01-10 | shipped |
| 102 | 2 | 150.50 | 2024-01-12 | pending |
| 103 | 3 | NULL | 2024-02-15 | canceled |
| 104 | 4 | 100.25 | 2024-03-05 | shipped |

`customers`

| `cutomer_id` | `name` | `country` | `signup_date` |
| --- | --- | --- | --- |
| 1 | John Doe | USA | 2023-11-15 |
| 2 | Jane Smith | UK | 2023-12-20 |
| 3 | Alice Lee | Canada | 2024-01-05 |
| 4 | Bob Brown | USA | 2024-02-10 |

# Indexes and Why They Matter

## What is an Index?

An **index** is a data structure that speeds up queries by allowing databases to quickly locate rows based on indexed columns. Think of it like an index of a book - rather than scanning every page, you can jump directly into relevant sections.

### Benefits of Indexing

- Faster lookups for `WHERE`, `JOIN` , and `ORDER BY` clauses).
- Reduces the need for **full table scans,** making queries more efficient.

### Trade-Offs of Indexing

- Indexes **take up space** and increase storage costs.
- They **slow down** `INSERT`, `UPDATE`, and `DELETE` operations because indexes must also be updated.

## Creating an Index

Create an index on the `order_date` column to speed up queries filtering by date.

```sql
CREATE INDEX idx_order_date ON orders(order_date);
```

## Checking Index Usage

Use `EXPLAIN` (MySQL/PostgreSQL) or `EXPLAIN ANALYZE` to check how a query runs.

```sql
EXPLAIN SELECT * FROM orders WHERE order_date = '2024-01-10';
```

| `id` | `select_type` | `table` | `type` | `possible_keys` | `key` | `rows` | `Extra` |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | SIMPLE | orders | ref | idx_order_date | idx_order_date | 1 | Using index |

The query uses the `idx_order_date` index, making it faster than scanning the entire table.

# Identifying and Avoiding Slow Queries

## Avoid Full Table Scans

Example: Use an indexed column in `WHERE` instead of a computed expression.

### Slow Query (Prevents Index Usage)

```sql
SELECT * FROM orders WHERE YEAR(order_date) = 2024;
```

### Optimized Query (Uses Index)

```sql
SELECT * FROM orders WHERE order_date >= '2024-01-01' AND order_date < '2025-01-01';
```

## Avoid `SELECT *` on Large Tables

Example: Retrieve only necessary columns to reduce transfer.

### Slow Query (Retrieves Unnecessary Data)

```sql
SELECT * FROM orders WHERE status = 'shipped';
```

### Optimized Query (Retrieves Only Needed Columns)

```sql
SELECT order_id, customer_id FROM orders WHERE status = 'shipped';
```

## Filtering Data Early in Query Execution

Example: Filter Rows in a subquery **before** performing joins.

### Inefficient Query (Joins Before Filtering, Unnecessary Processing)

```sql
SELECT c.name, o.order_amount
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_amount > 100;
```

### Optimized Query (Filters Before Join, Reducing Rows Processed)

```sql
SELECT c.name, 
			 o.order_amount
FROM customers c
JOIN (SELECT * 
			FROM orders 
			WHERE order_amount > 100) o
		ON c.customer_id = o.customer_id;
```

# Basic Optimization Techniques

## Adding Indexes to Optimize Queries

Example: Improve a frequently used query with an index.

### Slow Query (No Index on `status`)

```sql
SELECT * FROM orders WHERE status = 'shipped';
```

### Create an Index on `status`

```sql
CREATE INDEX idx_order_status ON orders(status);
```

## Using `LIMIT` for Exploratory Queries

Example: Use `LIMIT` to avoid unnecessary processing during data exploration.

```sql
SLECT * FROM orders LIMIT 10;
```

# Summary

| Optimization Strategy | How It Helps |
| --- | --- |
| **Use Indexes** | Speeds up queries by allowing direct access to rows. |
| **Avoid Full Table Scans** | Write queries to take advantage of indexes (e.g., avoid functions in `WHERE`). |
| **Limit Data Early** | Use `WHERE` and filtering in subqueries to reduce processed rows. |
| **Retrieve Only Needed Columns** | Avoid `SELECT *` to reduce data retrieval load. |
| **Use `LIMIT` for Testing** | Prevents long query execution times during exploration. |