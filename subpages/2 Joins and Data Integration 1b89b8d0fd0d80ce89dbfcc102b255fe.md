# 2. Joins and Data Integration

# Tables Used for Examples

`customers`

| `customer_id` | `name` | `country` |
| --- | --- | --- |
| 1 | John Doe | USA |
| 2 | Jane Smith | UK |
| 3 | Alice Lee | Canada |
| 4 | Bob Brown | USA |

`orders`

| `order_id` | `customer_id` | `amount` | `order_date` |
| --- | --- | --- | --- |
| 101 | 1 | 250 | 2024-03-01 |
| 102 | 2 | 150 | 2024-03-02 |
| 103 | 5 | 100 | 2024-03-03 |

# Combining Tables with `JOIN`s

## `INNER JOIN`

Returns only matching records from both tables (intersection).

```sql
SELECT c.name, o.order_id, o.amount
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id;
```

| `name` | `order_id` | `amount` |
| --- | --- | --- |
| John Doe | 101 | 250 |
| Jane Smith | 102 | 150 |
- **Alice Lee** and **Bob Brown** are not included because they don’t have orders.
- **Order 103** (`customer_id` = 5) is not included because there is no customer with that ID.

Practice Problem: Find the **order ID** and **amount** for customers **not** in the USA.

```sql
SELECT o.order_id, o.amount
FROM orders o
INNER JOIN customers c ON o.customer_id = o.customer_id
WHERE c.country <> 'USA';
```

| `order_id` | `amount` |
| --- | --- |
| 102 | 150 |

## `LEFT JOIN`

Returns all records from the left table (`customers`), with matching records from the right (`orders`). Missing matches return NULLs.

```sql
SELECT c.name, o.order_id, o.amount
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;
```

| `name` | `order_id` | `amount` |
| --- | --- | --- |
| John Doe | 101 | 250 |
| Jane Smith | 102 | 150 |
| Alice Lee | NULL | NULL |
| Bob Brown | NULL | NULL |
- Alice Lee and Bob Brown have **NULL values** for `order_id` and `amount` because they  have no orders.

Practice Problem: Show customers who have **never** placed an order.

```sql
SELECT c.name
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE c.customer_id NOT IN o.customer_id;
```

| `name` |
| --- |
| Alice Lee |
| Bob Brown |

## `RIGHT JOIN`

Returns all records from the right table (`orders`), with matching records from the left (`customers`). Missing matches return NULLs.

```sql
SELECT c.name, o.order_id, o.amount
FROM customers c
RIGHT JOIN orders o ON c.customer_id = o.customer_id;
```

| `name` | `order_id` | `amount` |
| --- | --- | --- |
| John Doe | 101 | 250 |
| Jane Smith  | 102 | 150 |
| NULL | 103 | 100 |
- **Order 103** appears, but NULL is shown for `name` because no matching customer exists.

Practice Problem: Find all orders **where the customer name is NULL** (i.e., where the customer does not exist in the customers table).

```sql
SELECT o.*
FROM orders o
RIGHT JOIN customers c ON o.customer_id = c.customer_id
WHERE o.customer_id NOT IN c.customer_id;
```

| `order_id` | `customer_id` | `amount` | `order_date` |
| --- | --- | --- | --- |
| 103 | 5 | 100 | 2024-03-03 |

## `FULL JOIN` (`FULL OUTER JOIN`)

Returns all records **from both tables.** Missing values appear as NULLs.

```sql
SELECT c.name, o.order_id, o.amount
FROM customers c
FULL JOIN orders o ON c.customer_id = o.customer_id;
```

| `name` | `order_id` | `amount` |
| --- | --- | --- |
| John Doe | 101 | 250 |
| Jane Smith | 102 | 150 |
| Alice Lee | NULL | NULL |
| Bob Brown | NULL | NULL |
| NULL | 103 | 100 |
- Includes **all customers** (even those with no orders) and **all orders** (even if they have no matching customer).

## `CROSS JOIN`

Returns every combination of rows from both tables (Cartesian product).

```sql
SELECT c.name, o.order_id, o.amount
FROM customers c
CROSS JOIN orders o;
```

- If `customers` had 4 rows and `orders` had 3 rows, **the result will have 4 x 3 = 12 rows.**
    - Usually, `CROSS JOIN` is not used unless necessary (e.g., generating all possible product-customer pairings).

## `SELF JOIN`

Used to join a table with itself.

Example: Find customers from the same country.

```sql
SELECT a.name Customer1, b.name Customer2, a.country
FROM customers a
JOIN customers b ON a.country = b.country AND a.customer_id <> b.customer_id;
```

| `Customer1` | `Customer2` | `country` |
| --- | --- | --- |
| John Doe | Bob Brown | USA |
| Bob Brown | John Doe | USA |

# When to Use `JOIN`s

| Scenario | Best Join Type |
| --- | --- |
| Retrieve only matching records. | `INNER JOIN` |
| Get all records from a main table (e.g., all customers) with possible matches from another. | `LEFT JOIN` |
| Get all records from a secondary table (e.g., all orders) with possible matches from a main table. | `RIGHT JOIN` |
| Combine both left and right joins, showing all records from both tables. | `FULL JOIN` |
| Generate all possible combinations of two tables. | `CROSS JOIN` |
| Compare rows within the same table. | `SELF JOIN` |

# `UNION` and Set Operators

While `JOIN`s combine tables **horizontally,** `UNION` and set operators combine **vertically.**

## `UNION` (Removes Duplicates)

Get a list of unique countries from both `customers` and `suppliers`.

```sql
SELECT country FROM customers
UNION
SELECT country FROM suppliers;
```

| `country` |
| --- |
| USA |
| UK |
| Canada |
| Germany |

## `UNION ALL` (Keeps Duplicates)

Get a full list of product names from `electronics` and `furniture` tables.

```sql
SELECT product_name FROM electronics
UNION ALL
SELECT product_name FROM furniture;
```

| `product_name` |
| --- |
| Laptop |
| Phone |
| Desk |
| Chair |
| Phone |

## `INTERSECT` (Common Rows)

Find customers who are also listed as suppliers.

```sql
SELECT name FROM customers
INTERSECT
SELECT name FROM suppliers;
```

| `name` |
| --- |
| John Doe |
| Bob Brown |