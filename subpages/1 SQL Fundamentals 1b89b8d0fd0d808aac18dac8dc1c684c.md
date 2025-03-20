# 1. SQL Fundamentals

# Basic Query Structure

### Explanation

- `SELECT` is used to specify which columns to retrieve.
- `FROM` specifies the table from which to retrieve data.
- `WHERE` filters rows based on conditions.

### Logical and Comparison Operators

- `=`: Equal to
- `>`: Greater than
- `>=`: Greater than or equal to
- `<`: Less than
- `<=`: Less than or equal to
- `<>` or `!=`: Not equal to
- `LIKE`: Pattern matching (e.g., `A%` for strings starting with ‘A’)
- `IN`: Checks if a value exists in a set (`WHERE country IN (’USA’, ‘Canada’)`)
- `AND`, `OR`, `NOT`: Logical operators to combine conditions

### Example

Lets say we have a `customers` table:

| `customer_id` | `name` | `country` | `age` | `status` |
| --- | --- | --- | --- | --- |
| 1 | John Doe | USA | 30 | Active |
| 2 | Jane Smith | UK | 25 | Inactive |
| 3 | Alice Lee | Canada | 35 | Active |
| 4 | Bob Brown | USA | 40 | Active |

Query: Get all active customers from the USA.

```sql
SELECT customer_id, name, country
FROM customers
WHERE status = 'Active' AND country = 'USA';
```

| `customer_id` | `name` | `country` |
| --- | --- | --- |
| 1 | John Doe | USA |

Practice Problem: Write a query to find customers who are **Inactive** or from **Canada.**

```sql
SELECT customer_id, name, country
FROM customers
WHERE status = 'Inactive' AND country = 'Canada';
```

| `customer_id` | `name` | `country` |
| --- | --- | --- |
| 3 | Alice Lee | Canada |

# Ordering and Limiting Results

## Explanation

- `ORDER BY` is used to sort query results.
- `ASC` (default) sorts in ascending order.
- `DESC` sorts in descending order.
- `LIMIT n` (MySQL, PostgreSQL) or `TOP n` (SQL Server) restricts the number or rows.

## Example

Query: Get the top 2 oldest customers.

```sql
SELECT name, age
FROM customers
ORDER BY age DESC
LIMIT 2;
```

| `name` | `age` |
| --- | --- |
| Bob Brown | 40 |
| Alice Lee | 35 |

Practice Problem: Write a query to get the **youngest** 3 customers.

```sql
SELECT name, age
FROM customers
ORDER BY age
LIMIT 3;
```

| `name` | `age` |
| --- | --- |
| Jane Smith | 25 |
| Alice Lee | 30 |
| John Doe | 35 |

# Basic Aggregation

## Explanation

- `COUNT(*)` counts rows.
- `SUM(column)` adds values.
- `AVG(column)` calculates the average.
- `MIN(column)` and `MAX(column)` find the smallest/largest values.
- `GROUP BY` groups rows with the same values.
- `HAVING` filters groups after aggregation.

## Example

Query: Count the number of customers per country.

```sql
SELECT country, COUNT(*) AS customer_count
FROM customers
GROUP BY country;
```

| `country` | `customer_count` |
| --- | --- |
| USA | 2 |
| UK | 1 |
| Country | 1 |

Practice Problem: Find the **average age of Active customers.**

```sql
SELECT AVG(age) AS avg_age
FROM customers
WHERE status = 'Active';
```

| `avg_age` |
| --- |
| 35.00 |

# Understanding Tables and Relationships

## Explanation

- **Primary Key (PK):** A column that uniquely identifies each row.
- **Foreign Key (FK):** A column that links to the primary key in another table.

## Example Tables

`customers`

| `customer_id` | `name` | `country` |
| --- | --- | --- |
| 1 | John Doe | USA |
| 2 | Jane Smith | UK |

`orders`

| `order_id` | `customer_id` | `amount` | `order_date` |
| --- | --- | --- | --- |
| 101 | 1 | 250 | 2024-03-01 |
| 102 | 2 | 150 | 2024-03-02 |

To join these tables:

```sql
SELECT c.name, o.amount
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id;
```

| `name` | `amount` |
| --- | --- |
| John Doe | 250 |
| Jane Smith | 150 |

Practice Problem: Write a query to find **all orders with customer names.**

```sql
SELECT c.name, o.order_id, o.amount, o.order_date
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id;
```

| `name` | `order_id` | `amount` | `order_date` |
| --- | --- | --- | --- |
| John Doe | 101 | 250 | 2024-03-01 |
| Jane Smith | 102 | 150 | 2024-03-02 |

# Aliases and Comments

## Explanation

- **Aliases** make column/table names more readable.
- **Comments** document SQL code.

## Example

```sql
SELECT c.name, AS CustomerName, o.amount AS OrderAmount
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.amount > 200; -- Filter high-value orders
```

| `CustomerName` | `OrderAmount` |
| --- | --- |
| John Doe | 250 |

Practice Problem: Write a query with **aliases** to retrieve `customer_id` as `ID` and `name` as `FullName`.

```sql
SELECT customer_id AS ID, name as FullName
FROM customers;
```

| `ID` | `FullName` |
| --- | --- |
| 1 | John Doe |
| 2 | Jane Smith |
| 3 | Alice Lee |
| 4 | Bob Brown |