# 9. Reporting and Business Intelligence

You need to be able to **write queries for reports, prep data for dashboards,** and **handle ad-hoc data requests efficiently.**

# Tables Used in This Lesson

`orders`

| `order_id` | `customer_id` | `order_amount` | `order_date` | `region` | `status` |
| --- | --- | --- | --- | --- | --- |
| 101 | 1 | 2500.75 | 2024-01-10 | East | shipped |
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

# Reporting Queries

## Aggregated Sales Report

Report: Total sales by region.

```sql
SELECT region, SUM(order_amount) AS total_sales
FROM orders
WHERE status = 'shipped'
GROUP BY region
ORDER BY total_sales DESC;
```

| `region` | `total_sales` |
| --- | --- |
| East | 701.00 |
| West | 150.50 |

## Computing KPIs

Report: Average order value per customer.

```sql
SELECT c.name, AVG(o.order_amout) AS avg_order_value
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.status = 'shipped'
GROUP BY c.name;
```

| `name` | `avg_order_value` |
| --- | --- |
| John Doe | 300.38 |
| Bob Brown | 100.25 |

# Views and Materialized Views for BI

## Creating a BI View

Create a view for easy reporting of shipped orders.

```sql
CREATE VIEW ShippedOrders AS
		SELECT o.order_id, o.customer_id, c.name, o.order_amount, o.order_date, o.region
		FROM orders o
		JOIN customers c ON o.customer_id = c.customer_id
		WHERE o.status = 'shipped';
		
SELECT * FROM ShippedOrders;
```

## Materialized Views (for Databases That Support Them)

Materialized views **store precomputed query results.** Unlike regular vies, they do not update automatically.

```sql
CREATE MATERIALIZED VIEW SalesSummary AS
		SELECT region, SUM(order_amount) AS total_sales
		FROM orders
		WHERE status = 'shipped'
		GROUP BY region;
		
-- Refresh manually when needed
REFRESH MATERIALIZED VIEW SalesSummary;
```

# SQL in BI Tools

Many BI tools (Power BI, Tableau, Looker) allow **custom SQL queries** as data sources.

Example: Power BI Custom SQL Query for Report Table

```sql
SELECT region, COUNT(order_id) AS total_orders, SUM(order_amount) AS revenue
FROM orders
WHERE status = 'shipped'
GROUP BY region;
```

# Ad-hoc Data Requests

## High-Value Customers for Last Month

Ad-hoc Request: “Find customers who spent over $300 in the last month.”

```sql
SELECT c.name, SUM(o.order_amount) AS total_spent
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE o.order_date >= CURDATE() - INTERVAL 1 MONTH
GROUP BY c.name
HAVING total_spent > 300
ORDER BY total_spent DESC;
```

## Active Customer Count

Ad-hoc Request: “How many active customers placed at least one order?”

```sql
SELECT COUNT(DISTINCT customer_id) AS active_customers
FROM orders
WHERE order_date >= CURDATE() - INTERVAL 6 MONTH;
```

# Exporting and Integrating SQL Data

### Exporting Results to Excel or CSV

```sql
SELECT * FROM orders
INTO OUTFILE '/tmp/orders_report.csv'
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
```

### Using SQL in ETL Pipelines

- SQL queries can be **scheduled** using **Python (pandas, SQLAlchemy)** or **BI tools.**
- Example: Power BI, Tableau, Metabase allow **direct SQL queries as data sources.**

# Summary

| Feature | Purpose | Example Use Case |
| --- | --- | --- |
| **Reporting Queries** | Aggregate data for dashboards | Monthly sales by region |
| **BI Views** | Simplify reporting queries | A view for shipped orders |
| **Materialized Views** | Precompute reports for performance | A daily sales summary table |
| **SQL in BI Tools** | Feed dashboards with SQL queries | Power BI, Tableau, custom queries |
| **Ad-hoc Requests** | Pull data for business questions | List of high-value customers |
| **Exporting Data** | Move data to reporting tools | CSV exports for Excel or ETL |