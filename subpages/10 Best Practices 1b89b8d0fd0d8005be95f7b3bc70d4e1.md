# 10. Best Practices

Writing **efficient, readable**, and **optimized SQL queries** is essential. Following **best practices** ensures your queries are easy to understand, maintainable, and performant. These principles help in debugging, collaboration, and performance optimization.

# Tables Used in This Lesson

`orders`

| `order_id` | `customer_id` | `order_amount` | `order_date` | `region` | `status` |
| --- | --- | --- | --- | --- | --- |
| 101 | 1 | 250.75 | 2024-01-10 | East | shipped |
| 102 | 2 | 150.50 | 2024-01-12 | West | pending |
| 103 | 3 | NULL | 2024-02-15 | North | canceled |
| 104 | 4 | 100.25 | 2024-03-05 | East | shipped |
| 105 | 1 | 350.00 | 2024-03-12 | East | shipped |

`customers`

| `customer_id` | `name` | `country` | `signup_date` |
| --- | --- | --- | --- |
| 1 | John Doe | USA | 2023-11-15 |
| 2 | Jane Smith | UK | 2023-12-20 |
| 3 | Alice Lee | Canada | 2024-01-05 |
| 4 | Bob Brown | USA | 2024-02-10 |

# Writing Readable SQL Code

## Formatting for Readability

### Bad Example (Hard to Read)

```sql
SELECT orders.order_id, customers.name, orders.order_amount FROM orders INNER JOIN customers ON orders.customer_id = customers.customer_id WHERE orders.status = 'shipped' ORDER BY orders.order_date DESC;;
```

### Good Example

```sql
SELECT o.order_id,
			 c.name AS customer_name,
			 o.order_amount
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.status = 'shipped'
ORDER BY o.order_date DESC;
```

# Only Retrieve What You Need

## Avoid `SELECT *`

### Bad Example (Pulls Unnecessary Data)

```sql
SELECT * FROM orders WHERE status = 'shipped';
```

### Good Example (Selects Only Necessary Columns)

```sql
SELECT order_id, customer_id, order_amount
FROM orders
WHERE status = 'shipped';
```

# Test and Validate Your Queries

## Debugging Step-by-Step

### Best Approach

1. Start with a Simple Query:
    
    ```sql
    SELECT * FROM orders LIMIT 10;
    ```
    
2. Test Filters Separately:
    
    ```sql
    SELECT * FROM orders WHERE order_date >= '2024-01-01';
    ```
    
3. Incrementally Add Joins and Aggregations:
    
    ```sql
    SELECT c.name, COUNT(o.order_id) AS total_orders
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
    WHERE o.order_date >= '2024-01-01'
    GROUP BY c.name;
    ```
    

# Optimize When Necessary

## Using Indexes

### Check Index Usage

```sql
EXPLAIN SELECT * FROM orders WHERE order_date - '2024-01-10';
```

### Add an Index for Faster Queries

```sql
CREATE INDEX idx_order_date ON orders(order_date);
```

## Filtering Data Before Joins

### Bad Example (Joining Before Filtering, Slower Query)

```sql
SELECT c.name, o.order_amount
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_amount > 100;
```

### Good Example (Filter Before Join, Improves Performance)

```sql
SELECT c.name, o.order_amount
FROM customers c
JOIN (SELECT *
			FROM orders
			WHERE order_Amount > 100) o
		ON c.customer_id = o.customer_id;
```

## Proper Use of `HAVING` vs `WHERE`

### Bad Example (Using `HAVING` When `WHERE` is Better)

```sql
SELECT region, SUM(order_amount) AS total_sales
FROM orders
GROUP BY region
HAVING order_amount > 200;
```

`HAVING` is used incorrectly because `order_amount` is not aggregated.

### Good Example (Use `WHERE` for Filtering Rows Before Aggregation)

```sql
SELECT region, SUM(order_amount) AS total_sales
FROM orders
WHERE order_amount > 200
GROUP BY region;
```

# Maintain Query Results (Documentation & Views)

## Create a View to Store a Common Query for Reuse

```sql
CREATE VIEW ActiveCustomers AS
		SELECT customer_id, name, country, signup_date
		FROM customers
		WHERE signup_date >= CURDATE() - INTERVAL 1 YEAR;
```

# Security and Privacy Considerations

## Removing Personal Information for Reporting

### Bad Practice (Exposing Full Customer Data)

```sql
SELECT * FROM customers;
```

### Better Practice (Only Retrieve Necessary Fields, Anonymized ID)

```sql
SELECT customer_id, country
FROM customers;
```

# Summary of Best Practices

| Best Practice | Why It’s Important | Example |
| --- | --- | --- |
| Format Code Clearly | Improves readability and collaboration | Proper indentation, comments |
| Retrieve Only Needed Data | Avoids unnecessary load on the database | `SELECT column_name` instead of `SELECT *` |
| Test Queries Step-by-Step | Helps validate accuracy and troubleshoot errors | Start small, then build complexity |
| Optimize Query Performance | Ensures queries run efficiently | Use indexes, filter before joining |
| Use Views for Reusability | Saves complex queries for future use | `CREATE VIEW ReportView AS ...` |
| Respect Data Security | Prevents unauthorized data access | Limit exposure of personal data |