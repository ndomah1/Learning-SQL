# 3. Subqueries and Common Table Expressions (CTEs)

**Subqueries** and **Common Table Expressions (CTEs)** allow us to break down complex queries into manageable steps.

# Tables Used in this Lesson

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
- The `orders` table tracks purchases made by customers.
- The `employees` table represents organizational hierarchy, where `manager_id` links employees to their manager.

# Subqueries (Nested Queries)

A **subquery** is a query enclosed in parentheses that is executed first and provides a result for the outer query.

- **Uncorrelated Subqueries:** Run independently of the outer query.
- **Correlated Subqueries:** Depend on values from the outer query and execute row by row.

## Subquery in `WHERE` Clause

Find all customers who have placed an order.

```sql
SELECT name
FROM customers
WHERE customer_id IN (SELECT customer_id FROM orders);
```

| `name` |
| --- |
| John Doe |
| Jane Smith |

## Subquery in `FROM` Clause (Derived Table)

Find the average order amount per customer.

```sql
SELECT customer_id, AVG(amount) AS avg_order_value
FROM (SELECT customer_id, amount FROM orders) AS order_summary
GROUP BY customer_id;
```

| `customer_id` | `avg_order_value` |
| --- | --- |
| 1 | 250.00 |
| 2 | 150.00 |

## Subquery in `SELECT` Clause

Show each customer’s total order amount.

```sql
SELECT name,
			 (SELECT SUM(amount)
				FROM orders o
				WHERE o.customer_id = c.customer_id) AS total_spent
FROM customers c;
```

| `name` | `total_spent` |
| --- | --- |
| John Doe | 250 |
| Jane Smith | 150 |
| Alice Lee | NULL |

## Correlated Subquery

Find customers whose order amount is above their average order amount.

```sql
SELECT customer_id, amount
FROM orders o1
WHERE amount > (SELECT AVG(amount) 
							  FROM orders o2
							  WHERE o1.customer_id = o2.customer_id);
```

| `customer_id` | `amount` |
| --- | --- |
| 1 | 250 |

# Common Table Expressions (CTEs)

**CTEs** (`WITH … AS`) are named temporary result sets that improve readability and allow repeated use within a query. They are especially helpful for multi-step transformations and recursive queries.

## Simple CTE EXample

Calculate the total order amount per customer using a CTE.

```sql
WITH CustomerOrders AS (
		SELECT customer_id, SUM(amount) AS total_spent
		FROM orders
		GROUP BY customer_id
)

SELECT c.name, CO.total_spent
FROM customers c
LEFT JOIN CustomerOrders CO ON c.customer_id = CO.customer_id;
LIMIT 2; -- For ranking
```

| `name` | `total_spent` |
| --- | --- |
| John Doe | 250 |
| Jane Smith | 150 |

## Recursive CTE (Hierarchical Data)

`employees`

| `employee_id` | `name` | `maanger_id` |
| --- | --- | --- |
| 1 | Alice Johnson | NULL |
| 2 | Bob Smith | 1 |
| 3 | Carol Williams | 1 |
| 4 | David Brown | 2 |
| 5 | Emily Davis | 2 |
| 6 | Frank Miller | 3 |
| 7 | Grace Wilson | 3 |

Find all employees and their managers.

```sql
WITH RECURSIVE ManagerEmployeePairs AS (
		-- Anchor: start with top-level managers (they have no mamager)
		SELECT employee_id, name, manager_id, NULL AS manager_name
		FROM employees
		WHERE manager_id IS NULL
		
		UNION ALL
		
		-- Recursive part: join each employee to its manager from the previous level
		SELECT e.employee_id, e.name, e.manager_id, m.name as manager_name
		FROM employees e
		JOIN ManagerEmployeePairs m ON e.manager_id = m.employee_id;
		
-- Only output rows that have a manager (skip top-level managers)
SELECT manager_name AS managers,
		   name AS employee
FROM ManagerEmployeePairs
WHERE manager_name IS NOT NULL;
```

| `manager` | `employee` |
| --- | --- |
| Alice Johnson | Bob Smith |
| Alice Johnson | Carol Williams |
| Bob Smith | David Brown |
| Bob Smith | Emily Davis |
| Carol Williams | Frank Miller |
| Carol Williams | Grace Wilson |