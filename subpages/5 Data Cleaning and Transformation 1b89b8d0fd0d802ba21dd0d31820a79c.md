# 5. Data Cleaning and Transformation

Data cleaning is a critical step in preparing raw data for analysis. We use various functions and techniques to handle missing data, remove duplicates, standardize text, and transform date values.

# Tables Used in This Lesson

`customers`

| `customer_id` | `name` | `email` | `country` | `state` | `signup_date` |
| --- | --- | --- | --- | --- | --- |
| 1 | John Doe | johnd@example.com | USA | Calif | 2024-01-10 |
| 2 | Jane Smith | NULL | UK | TX | 2024-01-12 |
| 3 | Alice Lee | alice@example.com | Canada | N/A | 2024-03-05 |
| 4 | Bob Brown | bob@example.com | USA | ca | 2024-03-10 |

`transactions`

| `transaction_id` | `customer_id` | `amount` | `transaction_date` |
| --- | --- | --- | --- |
| 1001 | 1 | 250.50 | 2024-01-10 |
| 1002 | 2 | NULL | 2024-01-12 |
| 1003 | 3 | 300.00 | 2024-03-05 |
| 1004 | 4 | 100.25 | 2024-03-10 |

# Handling Missing or Invalid Data

## Finding Missing Values (`IS NULL`)

Find all customers missing an email.

```sql
SELECT customer_id, name
FROM customers
WHERE email IS NULL;
```

| `customer_id` | `name` |
| --- | --- |
| 2 | Jane Smith |

## Replacing NULL Values (`COALESCE()`)

Replace missing emails with ‘No Email’.

```sql
SELECT customer_id, name, COALESCE(email, 'No email') AS email
FROM customers;
```

| `customer_id` | `name` | `email` |
| --- | --- | --- |
| 1 | John Doe | johnd@example.com |
| 2 | Jane Smith | No Email |
| 3 | Alice Lee | alice@example.com |
| 4 | Bob Brown | bob@example.com |

## Converting Invalid Values to NULL (`NULLIF()`)

Convert ‘N/A’ in the `state` column to NULL.

```sql
SELECT customer_id, name, NULLIF(state, 'N/A') AS cleaned_state
FROM customers;
```

| `customer_id` | `name` | `cleaned_state` |
| --- | --- | --- |
| 1 | John Doe | Calif |
| 2 | Jane Smith | TX |
| 3 | Alice Lee | NULL |
| 4 | Bob Brown | ca |

# Removing Duplicates

## Using `DISTINCT`

Find unique countries in the table.

```sql
SELECT DISTINCT country FROM customers;
```

| `country` |
| --- |
| USA |
| UK |
| Canada |

## Using `ROW_NUMBER()` to Identify Duplicates

```sql
-- If there were 2 John Doe
SELECT customer_id, name,
			 ROW_NUMBER() OVER(PARTITION BY name ORDER BY signup_date) AS row_num
FROM customers;
```

| `customer_id` | `name` | `row_num` |
| --- | --- | --- |
| 1 | John Doe | 1 |
| 5 | John Doe | 2 |

# Conditional Transformation (`CASE` Statements)

## Categorizing Data with `CASE`

```sql
SELECT transaction_id, amount,
		CASE
				WHEN amount >= 250 THEN 'High'
				WHEN amount >= 100 THEN 'Medium'
				ELSE 'Low'
		END AS category
FROM transactions;
```

| `transaction_id` | `amount` | `category` |
| --- | --- | --- |
| 1001 | 250.50 | High |
| 1002 | NULL | Low |
| 1003 | 300.00 | High |
| 1004 | 100.25 | Medium |

# String Cleaning and Manipulation

## Removing Leading and Trailing Spaces (`TRIM()`)

Ensure all data has no leading or trailing spaces.

```sql
SELECT TRIM(customer_id) AS customer_id,
		   TRIM(name) AS name,
		   TRIM(email) AS email,
		   TRIM(country) AS country,
		   TRIM(signup_date) AS signup_date
FROM customers;
```

## Standardizing Text Case (`UPPER()` and `LOWER()`)

Convert state names to uppercase.

```sql
SELECT customer_id, UPPER(state) AS standardized_state
FROM customers;
```

| `customer_id` | `standardized_state` |
| --- | --- |
| 1 | CALIF |
| 2 | TX |
| 3 | N/A |
| 4 | CA |

## Replacing Values (`REPLACE()`)

Replace ‘@example.com’ with ‘@company.com’ in emails.

```sql
SELECT customer_id, REPLACE(email, '@example.com', '@company.com') AS updated_email
FROM customers;
```

| `customer_id` | `email` |
| --- | --- |
| 1 | johnd@company.com |
| 2 | No Email |
| 3 | alice@company.com |
| 4 | bob@company.com |

# Date and Time Transformations

## Extracting Date Components (`YEAR()`, `MONTH()`)

Extract year and month from `signup_date`

```sql
SELECT customer_id, signup_date,
			 YEAR(signup_date) AS signup_year,
			 MONTH(signup_date) AS signup_month
FROM customers;
```

| `customer_id` | `signup_date` | `signup_year` | `signup_month` |
| --- | --- | --- | --- |
| 1 | 2024-01-10 | 2024 | 1 |
| 2 | 2024-01-12 | 2024 | 1 |
| 3 | 2024-03-05 | 2024 | 3 |
| 4 | 2024-03-10 | 2024 | 3 |

## Calculating Date Differences (`DATEDIFF()`)

Find days since customer signup.

```sql
SELECT customer_id, signup_date,
			 DATEDIFF(CURDATE(), signup_date) AS days_since_signup
FROM customers;
```

| `customer_id` | `signup_date` | `days_since_signup` |
| --- | --- | --- |
| 1 | 2024-01-10 | 432 |
| 2 | 2024-01-12 | 430 |
| 3 | 2024-03-05 | 377 |
| 4 | 2024-03-10 | 372 |

# Summary of Cleaning Techniques

| Technique | SQL Function(s) | Use Case |
| --- | --- | --- |
| **Handling Missing Data** | `IS NULL`, `COALESCE()`, `NULLIF()` | Detecting and replacing NULL values. |
| Removing Duplicates | `DISTINCT`, `ROW_NUMBER()` | Finding duplicate records. |
| Conditional Transformations | `CASE` | Categorizing or fixing values. |
| String Cleaning | `TRIM()`, `UPPER()`, `LOWER()`, `REPLACE()` | Standardizing text values. |
| Date Cleaning | `YEAR()`, `MONTH()`, `DATEDIFF()` | Extracting date components, calculating differences. |