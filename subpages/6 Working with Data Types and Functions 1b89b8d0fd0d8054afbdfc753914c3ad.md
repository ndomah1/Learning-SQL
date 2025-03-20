# 6. Working with Data Types and Functions

Using the right data type ensures performance optimization, data integrity, and correct calculations.

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

# Understanding Data Types

Data types define the nature of stored data. Using the correct data type ensures data integrity and efficient querying.

| Data Type | Example | Use Case |
| --- | --- | --- |
| Text (`CHAR`, `VARCHAR`, `TEXT`) | `'USA'`, `'Pending'` | Store names, statuses, descriptions. |
| Numeric (`INT`, `DECIMAL` , `FLOAT`) | `250`, `150.75` | Store whole numbers and precise decimals. |
| Date/Time (`DATE`, `DATETIME`, `TIMESTAMP`) | `2024-01-10` | Store dates and timestamps for analysis. |
| Boolean (`BIT` , `BOOLEAN`) | `1`, `0`, `TRUE` , `FALSE` | Store binary values (Yes/No, True/False) |

## Casting Between Data Types (`CAST()`, `CONVERT()`)

Convert order amount to text for concatenation.

```sql
SELECT order_id,
			 'Order Amount: ' || CAST(order_amount AS VARCHAR) AS order_info
FROM orders;
```

| `order_id` | `order_info` |
| --- | --- |
| 101 | Order Amount: 250.75 |
| 102 | Order Amount: 150.50 |

# Built-In Functions

## Text

Format customer names.

```sql
SELECT customer_id,
		   UPPER(name) AS uppercase_name,
		   LOWER(name) AS lowercase_name,
		   TRIM(name) AS trimmed_name
FROM customers;
```

| `customer_id` | `uppercase_name` | `lowercase_name` | `trimmed_name` |
| --- | --- | --- | --- |
| 1 | JOHN DOE | john doe | John Doe |

## Numeric

Rounding Order Amounts

```sql
SELECT order_id,
			 order_amount,
			 ROUDN(order_amount, 1) AS rounded_amount
FROM orders;
```

| `order_id` | `order_amount` | `rounded_amount` |
| --- | --- | --- |
| 101 | 250.75 | 250.8 |
| 102 | 150.50 | 150.5 |

## Date/Time

### Extracting Date Parts

```sql
SELECT order_id, order_date,
			 EXTRACT(YEAR FROM order_date) AS order_year,
			 EXTRACT(MONTH FROM order_date) AS order_month
FROM orders
WHERE order_id = 101;
```

| `order_id` | `order_date` | `order_year` | `order_month` |
| --- | --- | --- | --- |
| 101 | 2024-01-10 | 2024 | 1 |

### Adding and Subtracting Time

```sql
SELECT customer_id, signup_date,
		   DATE_ADD(signup_date, INTERVAL 1 YEAR) AS next_anniversary
FROM customers
WHERE customer_id = 1;
```

| `customer_id` | `signup_date` | `next_anniversary` |
| --- | --- | --- |
| 1 | 2023-11-15 | 2024-11-15 |

## Conversion Functions (`CAST()`, `CONVERT()`, `TO_CHAR()`)

### Formatting Dates

```sql
SELECT order_id,
			 TO_CHAR(order_date, 'YYYY-DD-MM') AS formatted_date
FROM orders
WHERE order_id - 101;
```

| `order_id` | `formatted_date` |
| --- | --- |
| 101 | 2024-10-01 |

## Precision and Rounding

```sql
SELECT order_id,
			 CAST(order_amount AS DECIMAL(10,1)) AS rounded_amount
FROM orders;
```

| `order_id` | `precise_amount` |
| --- | --- |
| 101 | 250.8 |