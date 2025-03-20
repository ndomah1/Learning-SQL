# 4. Aggregation and Window Functions

Aggregation and window functions allow you to summarize and analyze data effectively. Aggregations help summarize data into meaningful statistics, while window functions provide powerful analytics without collapsing rows.

# Aggregation & `GROUP BY`

- `GROUP BY` groups data into categories before applying aggregate functions.
- Aggregate functions include:
    - `COUNT()` - Count the number of rows.
    - `SUM()` - Calculate the total sum.
    - `AVG()` - Find the average value.
    - `MIN() / MAX()` - Find the smallest/largest values.
- `HAVING` filters grouped results, while `WHERE` filters **before** grouping.

## Tables Used

`sales`

| `sale_id` | `customer_id` | `amount` | `sale_date` | `region` |
| --- | --- | --- | --- | --- |
| 101 | 1 | 250 | 2024-01-10 | East |
| 102 | 2 | 150 | 2024-01-12 | West |
| 103 | 1 | 300 | 2024-02-15 | East |
| 104 | 3 | 200 | 2024-03-05 | North |
| 105 | 4 | 100 | 2024-03-10 | East |

`customers`

| `customer_id` | `name` | `country` |
| --- | --- | --- |
| 1 | John Doe | USA |
| 2 | Jane Smith | UK |
| 3 | Alice Lee | Canada |
| 4 | Bob Brown | USA |

## Basic `GROUP BY` Example

Find total sales amount per region.

```sql
SELECT region, SUM(amount) AS total_sales
FROM sales
GROUP BY region;
```

| `region` | `total_sales` |
| --- | --- |
| East | 650 |
| West | 150 |
| North | 200 |

## `HAVING` vs. `WHERE`

Find regions where total sales exceed 300.

```sql
SELECT region, SUM(amount) AS total_sales
FROM sales
GROUP BY region
HAVING SUM(amount) > 300;
```

| `region` | `total_sales` |
| --- | --- |
| East | 650 |

# Window Functions

A window function performs a calculation **across a set of rows** that are related to the current row. Unlike `GROUP BY`, which collapses rows into a single summary row, window functions **retain individual rows** while adding analytical calculations.

## Key Features

- `OVER()` Clause: Defines the “window” of rows to consider.
- `PARTITION BY`: Groups rows **within** the window, like `GROUP BY` but does not collapse rows.
- `ORDER BY`: Specifies how rows are ordered **within** the window.

## Key Functions

- **Ranking Functions:** `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `NTILE(n)`
- **Aggregation Over a Window:** `SUM() OVER()` , `AVG() OVER()`, `COUNT() OVER()`
- **Lag & Lead:** `LAG()`, `LEAD()`
- **Percentile Ranking:** `PERCENT_RANK()`, `CUME_DIST()`

## Table Used

`sales`

| `sale_id` | `customer_id` | `amount` | `sale_date` | `region` |
| --- | --- | --- | --- | --- |
| 101 | 1 | 500 | 2024-01-01 | East |
| 102 | 2 | 400 | 2024-01-02 | East |
| 103 | 3 | 400 | 2024-01-03 | East |
| 104 | 4 | 300 | 2024-01-04 | East |
| 105 | 5 | 250 | 2024-01-05 | West |
| 106 | 6 | 150 | 2024-01-06 | West |

## Ranking Functions

Ranking functions assign a unique rank to each row **within a partition** (optional) **based on an order.**

### `ROW_NUMBER()` - Unique Row Number Per Partition

Assign a **unique** row number to each sale **within each region.**

```sql
SELECT sale_id, region, amount,
		ROW_NUMBER() OVER(PARTITION BY region ORDER BY amount DESC) AS row_num
FROM sales;
```

| `sale_id` | `region` | `amount` | `row_num` |
| --- | --- | --- | --- |
| 101 | East | 500 | 1 |
| 102 | East | 400 | 2 |
| 103 | East | 400 | 3 |
| 104 | East | 300 | 4 |
| 105 | West | 250 | 1 |
| 106 | West | 150 | 2 |

### `RANK()` - Handles Ties with Gaps

Assigns **the same rank** to duplicate values but **skips ranks** for ties

```sql
SELECT sale_id, region, amount,
		RANK() OVER(PARTITION BY region ORDER BY amount DESC) AS sales_rank
FROM sales;
```

| `sale_id` | `region` | `amount` | `sales_rank` |
| --- | --- | --- | --- |
| 101 | East | 500 | 1 |
| 102 | East | 400 | 2 |
| 103 | East | 400 | 2 |
| 104 | East | 300 | 4 |
| 105 | West | 250 | 1 |
| 106 | West | 150 | 2 |

### `DENSE_RANK()` - No Gaps in Ranking

Assigns **the same rank** to duplicate values **without** skipping ranks.

```sql
SELECT sale_id, region, amount,
		DENSE_RANK() OVER(PARTITION BY region ORDER BY amount DESC) AS dense_rank
FROM sales;
```

| `sale_id` | `region` | `amount` |  |
| --- | --- | --- | --- |
| 101 | East | 500 | 1 |
| 102 | East | 400 | 2 |
| 103 | East | 400 | 2 |
| 104 | East | 300 | 3 |
| 105 | West | 250 | 1 |
| 106 | West | 150 | 2 |

### `NTILE(n)` - Split into Buckets

```sql
SELECT sale_id, region, amount,
		NTILE(2) OVER(PARTITION BY region ORDER BY amount DESC) AS bucket
FROM sales;
```

| `sale_id` | `region` | `amount` | `bucket` |
| --- | --- | --- | --- |
| 101 | East | 500 | 1 |
| 102 | East | 400 | 1 |
| 103 | East | 400 | 2 |
| 104 | East | 300 | 2 |
| 105 | West | 250 | 1 |
| 106 | West | 150 | 2 |

## Aggregation Over a Window

### `SUM() OVER()` - Running Total

Calculate a cumulative total of sales per region.

```sql
SELECT sale_id, region, amount, 
		SUM(amount) OVER(PARTITION BY region ORDER BY sale_date) AS running_total
FROM sales;
```

| `sale_id` | `region` | `amount` | `running_total` |
| --- | --- | --- | --- |
| 101 | East | 500 | 500 |
| 102 | East | 400 | 900 |
| 103 | East | 400 | 1300 |
| 104 | East | 300 | 1600 |
| 105 | West | 250 | 250 |
| 106 | West | 150 | 400 |

## Lag & Lead Functions

### `LAG()` - Previous Row’s Value

Find previous sale amount for each sale **within the same region.**

```sql
SELECT sale_id, region, amount, 
		LAG(amount, 1) OVER(PARTITION BY region ORDER BY sale_date) AS prev_sale
FROM sales;
```

| `sale_id` | `region` | `amount` | `prev_sale` |
| --- | --- | --- | --- |
| 101 | East | 500 | NULL |
| 102 | East | 400 | 500 |
| 103 | East | 400 | 400 |
| 104 | East | 300 | 400 |
| 105 | West | 250 | NULL |
| 106 | West | 150 | 250 |

### `LEAD()` - Next Row’s Value

```sql
SELECT sale_id, region, amount, 
		LEAD(amount, 1) OVER(PARTITION BY region ORDER BY sale_date) AS next_sale
FROM sales;
```

| `sale_id` | `region` | `amount` | `next_sale` |
| --- | --- | --- | --- |
| 101 | East | 500 | 400 |
| 102 | East | 400 | 400 |
| 103 | East | 400 | 300 |
| 104 | East | 300 | NULL |
| 105 | West | 250 | 150 |
| 106 | West | 150 | NULL |

## Percentile & Quartile Ranking

### `PERCENT_RANK()` - Percentile Rank

Find percentile rank of each sale within its region.

```sql
SELECT sale_id, region, amount, 
		PERCENT_RANK() OVER(PARTITION BY region ORDER BY amount) AS percentile_rank
FROM sales;
```

| `sale_id` | `region` | `amount` | `percentile_rank` |
| --- | --- | --- | --- |
| 101 | East | 500 | 1.0000 |
| 102 | East | 400 | 0.3333 |
| 103 | East | 400 | 0.3333 |
| 104 | East | 300 | 0.0000 |
| 105 | West | 250 | 1.0000 |
| 106 | West | 150 | 0.0000 |

### `CUME_DIST()` - Cumulative Distribution

```sql
SELECT sale_id, region, amount,
		CUME_DIST() OVER(PARTITION BY region ORDER BY amount) AS cume_dist
FROM sales;
```

| `sale_id` | `region` | `amount` | `cume_dist` |
| --- | --- | --- | --- |
| 101 | East | 500 | 0.25 |
| 102 | East | 400 | 0.75 |
| 103 | East | 400 | 0.75 |
| 104 | East | 300 | 1.00 |
| 105 | West | 250 | 0.50 |
| 106 | West | 150 | 1.00 |