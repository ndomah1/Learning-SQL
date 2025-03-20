# 8. Stored Procedures and Views

**Views** and **Stored Procedures** help **simplify queries, automate tasks,** and **enhance security** by controlling how data is accessed and processed. 

# Tables Used in This Lesson

`orders`

| `order_id` | `customer_id` | `order_amount` | `order_date` | `status` |
| --- | --- | --- | --- | --- |
| 101 | 1 | 250.75 | 2024-01-10 | shipped |
| 102 | 2 | 150.50 | 2024-01-12 | pending |
| 103 | 3 | NULL | 2024-02-15 | canceled |
| 104 | 4 | 100.25 | 2024-03-05 | shipped |

`customers`

| `customer_id` | `name` | `country` | `signup_date` |
| --- | --- | --- | --- |
| 1 | John Doe | USA | 2023-11-15 |
| 2 | Jane Smith | UK | 2023-12-20 |
| 3 | Alice Lee | Canada | 2024-01-05 |
| 4 | Bob Brown | USA | 2024-02-10 |

# Views

## What is a View?

A **view** is a virtual table based on a query. It **simplifies** complex joins, ensures **consistent business logic,** and **secures** underlying data by restricting access to only the columns included in the view.

### Why Use Views?

- **Simplified Queries:** Can query a **predefined view** instead of writing complex joins.
- **Enhances Security:** Limits access to specific columns or rows (e.g., hiding sensitive data).
- **Ensures Data Consistency:** Standardized calculations prevent inconsistent reporting.

## Creating a View

Create a view showing customer orders with only shipped statuses.

```sql
CREATE VIEW ShippedOrders AS
SELECT o.order_id, o.customer_id, c.name, o.order_amout, o.order_date
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.status = 'shipped';

SELECT * FROM ShippedOrders;
```

## Updating Views

Modify an existing view to also include the `status` column.

```sql
CREATE OR REPLACE VIEW ShippedOrders AS
		SELECT o.order_id, o.customer_id, c.name, o.order_amout, o.order_date, o.status
		FROM orders o
		JOIN customers c ON o.customer_id = c.customer_id
		WHERE o.status = 'shipped';
```

## Deleting Views

Remove an unused view.

```sql
DROP VIEW ShippedOrders;
```

# Stored Procedures

## What is a Stored Procedure?

A **Stored Procedure** is a saved script that can be executed **multiple times** with a single command. It can **take parameters**, perform **multiple queries,** and **modify data.**

### Why Use Stored Procedures?

- **Automates recurring tasks** (e.g., daily sales report).
- **Encapsulates complex queries** for **easy execution.**
- **Improves performance** by pre-compiling queries.

## Creating a Simple Stored Procedure

Create a stored procedure to get orders by customer ID. Then run the store procedure for ID of 2.

```sql
CREATE PROCEDURE GetOrdersByCustomer (IN cust_id INT)
BEGIN
		SELECT * FROM orders WHERE customer_id = cust_id;
END;

CALL GetOrdersByCustomer(2);
```

## Stored Procedure with Multiple Steps

Insert a new order and return confirmation.

```sql
CREATE PROCEDURE AddOrder (
		IN cust_id INT,
		IN amount DECIMAL(10,2),
		IN order_date DATE,
		IN order_status VARCHAR(20)
)

BEGIN
		INSERT INTO orders (customer_id, order_amount, order_date, status)
		VALUES (cust_id, amount, order_date, order_status);
		
		SELECT 'Order Added Successfully' AS confirmation;
END;

CALL AddOrder(5, 300.00, '2024-04-10', 'pending');
```

## Stored Procedure with Output Parameters

Return the total amount spent by a customer.

```sql
CREATE PROCEDURE GetTotalSpent (
		IN cust_id INT,
		OUT total_spent DECIMAL(10,2)
)

BEGIN
		SELECT SUM(order_amount) INTO total_spent
		FROM orders
		WHERE customer_id = cust_id;
END;

CALL GetTotalSpent(1, @total);
SELECT @total AS TotalSpent;
```

# User-Defined Functions (UDFs)

## What is a UDF?

A **User-Defined Function (UDF)** returns a single value and can be in `SELECT` statements like a built-in function. 

### Why Use UDFs?

- Encapsulate **complex calculations** for reuse.
- Improve **query readability.**

## Create a Scalar Function

Create a function that returns a customer’s total spending. 

```sql
CREATE FUNCTION TotalSpent(cust_id INT) RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
		DECLARE total DECIMAL(10,2):
		SELECT SUM(order_amount) INTO total FROM orders WHERE customer_id = cust_id;
		RETURN total;
END;

SELECT name, TotalSpent(customer_id) AS total_spent FROM customers;
```

# When to Use Views vs. Stored Procedures

| Feature | Views | Stored Procedures |
| --- | --- | --- |
| **Purpose** | Virtual table (read-only) | Automates queries, updates, inserts |
| **Use Case** | Simplify complex joins, restrict access | Automate reports, modify data |
| **Performance** | Query executes each time | Pre-compiled, can improve performance |
| **Flexibility** | Limited to `SELECT` statements | Can perform multiple actions |