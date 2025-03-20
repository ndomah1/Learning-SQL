# 11. Industry-Specific Considerations

SQL is used across **different industries,** but each sector has **unique data challenges and common SQL patterns.** Understanding industry-specific SQL skills can help **tailor your approach** in job interviews or real-world analysis.

# Finance

## Common SQL Use Cases in Finance

- Aggregating transactions by **account** or **date**
- Computing **running balances** using **window functions**
- Detecting **fraudulent activity** by comparing transactions to averages
- Ensuring **data integrity** with proper handling of **monetary data (DECIMAL type)**

`transactions`

| **`transaction_id`** | `account_id` | `amount` | `transaction_date` | `transaction_type` | `status` |
| --- | --- | --- | --- | --- | --- |
| 1001 | 1 | 500.00 | 2024-01-10 | deposit | completed |
| 1002 | 2 | -250.00 | 2024-01-12 | withdrawal | completed |
| 1003 | 1 | -100.00 | 2024-01-20 | withdrawal | completed |
| 1004 | 3 | 200.00 | 2024-02-01 | deposit | pending |

## Compute Running Balance for Each Account

```sql
SELECT account_id,
			 transaction_date,
			 amount
			 SUM(amount) OVER (PARTITION BY account_id ORDER BY transaction_date) AS running_balance
FROM transactions;		
```

| `account_id` | `transaction_date` | `amount` | `running_balance` |
| --- | --- | --- | --- |
| 1 | 2024-01-10 | 500.00 | 500.00 |
| 1 | 2024-01-20 | -100.00 | 400.00 |

## Detect Suspicious Large Transactions

```sql
SELECT transaction_id, 
		   account_id, 
		   amount
FROM transactions
WHERE ABS(amount) > (SELECT AVG(ABS(amount)) * 3 
										 FROM transactions;
```

# Healthcare

## Common SQL Use Cases in Healthcare

- Aggregating **patient counts, treatments,** or **hospital visits**
- Handling **missing data** in admissions/discharge records
- Using **recursive CTEs** for hierarchical medical codes

`patients`

| `patient_id` | `name` | `birth_date` | `gender` | `admission_date` | `discharge_date` | `diagnosis_code` |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | John Doe | 1985-03-12 | M | 2024-01-10 | 2024-01-15 | A123 |
| 2 | Jane Smith | 1990-07-24 | F | 2024-02-05 | NULL | B456 |
| 3 | Alice Lee | 1978-11-05 | F | 2024-03-02 | 2024-03-10 | C789 |

## Count Patients by Diagnosis Code

```sql
SELECT diagnosis_code,
			 COUNT(patient_id) AS patient_code
FROM patients
GROUP BY diagnosis_code;
```

| `diagnosis_code` | `patient_code` |
| --- | --- |
| A123 | 1 |
| B456 | 1 |
| C789 | 1 |

## Handle Missing Discharge Dates

```sql
SELECT patient_id,
		   name,
		   COALESCE(discharge_date, 'Still Admitted') AS discharge_status
FROM patients;
```

| `patient_id` | `name` | `discharge_status` |
| --- | --- | --- |
| 1 | John Doe | 2024-01-15 |
| 2 | Jane Smith | Still Admitted |
| 3 | Alice Lee | 2024-03-10 |

# Retail

## Common SQL Use Cases in Retail

- Generating **weekly/monthly sales reports**
- Identifying **top-selling products** or **low-stock items**
- **Cohort analysis** for customer retention

`sales`

| `sale_id` | `customer_id` | `product_id` | `quantity` | `total_amount` | `sale_date` | `store_location` |
| --- | --- | --- | --- | --- | --- | --- |
| 101 | 1 | 501 | 2 | 40.00 | 2024-01-15 | New York |
| 102 | 2 | 502 | 1 | 20.00 | 2024-02-10 | Los Angeles |
| 103 | 1 | 503 | 3 | 60.00 | 2024-03-05 | New York |

## Compute Monthly Sales

```sql
SELECT DATE_FORMAT(sale_date, '%Y-%m') AS month,
		   SUM(total_amount) AS total_sales
FROM sales
GROUP BY month
ORDER BY month;
```

| `month` | `total_sales` |
| --- | --- |
| 2024-01 | 40.00 |
| 2024-02 | 20.00 |
| 2024-03 | 60.00 |

## Identifying Returning vs. New Customers

```sql
SELECT customer_id,
			 MIN(sale_date) AS first_purchase,
			 COUNT(sale_id) AS total_purchases,
			 CASE
					 WHEN COUNT(sale_id) > 1 THEN 'Returning'
					 ELSE 'New'
			 END AS customer_type
FROM sales
GROUP BY customer_id;
```

| `customer_id` | `first_purchase` | `total_purchases` | `customer_type` |
| --- | --- | --- | --- |
| 1 | 2024-01-15 | 2 | Returning |
| 2 | 2024-02-10 | 1 | New |

# Tech

## Common SQL Use Cases in Tech

- **Tracking user behavior** from event logs
- Performing **A/B test analysis**
- **Window Functions** for calculating **user engagement**

`events`

| `event_id` | `user_id` | `event_name` | `event_time` | `platform` |
| --- | --- | --- | --- | --- |
| 1 | 101 | page_view | 2024-01-10 08:30 | mobile |
| 2 | 102 | purchase | 2024-01-12 14:05 | web |
| 3 | 103 | page_view | 2024-02-20 10:15 | mobile |
| 4 | 101 | click | 2024-03-01 18:45 | web |

## Count Events by Type

```sql
SELECT event_name, COUNT(event_id) AS event_count
FROM events
GROUP BY event_name;
```

| `event_name` | `event_count` |
| --- | --- |
| page_view | 2 |
| purchase | 1 |
| click | 1 |

## Computer User Retention (Time Between Events)

```sql
SELECT user_id, 
       event_name, 
       event_time,
       LAG(event_time) OVER (PARTITION BY user_id ORDER BY event_time) AS last_event_time,
       TIMESTAMPDIFF(DAY, LAG(event_time) OVER (PARTITION BY user_id ORDER BY event_time), event_time) AS days_since_last_event
FROM events;

```

| `user_id` | `event_name` | `event_time` | `last_event_time` | `days_since_last_event` |
| --- | --- | --- | --- | --- |
| 101 | page_view | 2024-01-10 08:30 | NULL | NULL |
| 102 | purchase | 2024-01-12 14:05 | NULL | NULL |
| 103 | page_view | 2024-02-20 10:15 | NULL | NULL |
| 101 | click | 2024-03-01 18:45 | 2024-01-10 08:30 | 51 |