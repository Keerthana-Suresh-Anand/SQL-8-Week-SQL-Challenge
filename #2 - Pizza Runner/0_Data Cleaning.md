# [#2 - Pizza Runner](https://8weeksqlchallenge.com/case-study-2/)
[<img src="https://github.com/Keerthana-Suresh-Anand/SQL-8-Week-SQL-Challenge/raw/main/Images/%232.png" alt="Pizza Runnerr" width="180"/>](https://8weeksqlchallenge.com/case-study-2/)

## Problem Statement
Danny owns a pizza shop that delivers food to customers. He has an app for that and has collected data from it. He needs help with data cleaning and analysis to optimize operations.

## Entity Relationship Diagram

<img src = "https://github.com/Keerthana-Suresh-Anand/SQL-8-Week-SQL-Challenge/raw/main/Images/%232%20-%20ERD.png" width="700">

## Data Cleaning
### 1) Customer Orders

| order_id | customer_id | pizza_id | exclusions | extras    | order_time          |
|----------|-------------|----------|------------|-----------|---------------------|
| 1        | 101         | 1        |            |           | 2021-01-01 18:05:02 |
| 2        | 101         | 1        |            |           | 2021-01-01 19:00:52 |
| 3        | 102         | 1        |            |           | 2021-01-02 23:51:23 |
| 3        | 102         | 2        |            | NaN       | 2021-01-02 23:51:23 |
| 4        | 103         | 1        | 4          |           | 2021-01-04 13:23:46 |
| 4        | 103         | 1        | 4          |           | 2021-01-04 13:23:46 |
| 4        | 103         | 2        | 4          |           | 2021-01-04 13:23:46 |
| 5        | 104         | 1        | null       | 1         | 2021-01-08 21:00:29 |
| 6        | 101         | 2        | null       | null      | 2021-01-08 21:03:13 |
| 7        | 105         | 2        | null       | 1         | 2021-01-08 21:20:29 |
| 8        | 102         | 1        | null       | null      | 2021-01-09 23:54:33 |
| 9        | 103         | 1        | 4          | 1, 5      | 2021-01-10 11:22:59 |
| 10       | 104         | 1        | null       | null      | 2021-01-11 18:34:49 |
| 10       | 104         | 1        | 2, 6       | 1, 4      | 2021-01-11 18:34:49 |

- `exclusions` contains NULL and blank values. The data type is VARCHAR.
- `extras` contains NULL, NaN, and blank values. The data type is VARCHAR.

#### Steps:
- Create a temporary table `cleaned_customer_orders` and modify `exclusions` and `extras`.
- `exclusions`: Change blanks to NULL.
- `extras`: Change NaN, blanks to NULL.
  
```sql
CREATE TEMPORARY TABLE cleaned_customer_orders AS
  SELECT
    order_id,
    customer_id,
    pizza_id,
    CASE
          WHEN exclusions LIKE 'null' OR exclusions = '' THEN NULL
          ELSE exclusions
    END AS exclusions,
    CASE
          WHEN extras = 'NaN' OR extras LIKE 'null'  OR extras = '' THEN NULL
          ELSE extras
    END AS extras,
    order_time
  FROM pizza_runner.customer_orders;
```

| order_id | customer_id | pizza_id | exclusions | extras | order_time                |
|----------|-------------|----------|------------|--------|---------------------------|
| 1        | 101         | 1        | null       | null   | 2021-01-01 18:05:02      |
| 2        | 101         | 1        | null       | null   | 2021-01-01 19:00:52      |
| 3        | 102         | 1        | null       | null   | 2021-01-02 23:51:23      |
| 3        | 102         | 2        | null       | null   | 2021-01-02 23:51:23      |
| 4        | 103         | 1        | 4          | null   | 2021-01-04 13:23:46      |
| 4        | 103         | 1        | 4          | null   | 2021-01-04 13:23:46      |
| 4        | 103         | 2        | 4          | null   | 2021-01-04 13:23:46      |
| 5        | 104         | 1        | null       | 1      | 2021-01-08 21:00:29      |
| 6        | 101         | 2        | null       | null   | 2021-01-08 21:03:13      |
| 7        | 105         | 2        | null       | 1      | 2021-01-08 21:20:29      |
| 8        | 102         | 1        | null       | null   | 2021-01-09 23:54:33      |
| 9        | 103         | 1        | 4          | 1, 5   | 2021-01-10 11:22:59      |
| 10       | 104         | 1        | null       | null   | 2021-01-11 18:34:49      |
| 10       | 104         | 1        | 2, 6       | 1, 4   | 2021-01-11 18:34:49      |


---

### 2) Runner Orders

| order_id | runner_id | pickup_time          | distance | duration   | cancellation             |
|----------|-----------|----------------------|----------|------------|--------------------------|
| 1        | 1         | 2021-01-01 18:15:34  | 20km     | 32 minutes |                          |
| 2        | 1         | 2021-01-01 19:10:54  | 20km     | 27 minutes |                          |
| 3        | 1         | 2021-01-03 00:12:37  | 13.4km   | 20 mins    | NaN                      |
| 4        | 2         | 2021-01-04 13:53:03  | 23.4     | 40         | NaN                      |
| 5        | 3         | 2021-01-08 21:10:57  | 10       | 15         | NaN                      |
| 6        | 3         | null                 | null     | null       | Restaurant Cancellation  |
| 7        | 2         | 2020-01-08 21:30:45  | 25km     | 25 mins    | null                     |
| 8        | 2         | 2020-01-10 00:15:02  | 23.4 km  | 15 minute  | null                     |
| 9        | 2         | null                 | null     | null       | Customer Cancellation    |
| 10       | 1         | 2020-01-11 18:50:20  | 10km     | 10 minutes | null                     |

- `pickup_time` contains NULL values. The data type is VARCHAR.
- `distance` contains NULL values. Some of the values have 'km' at the end. The data type is VARCHAR.
- `duration` contains NULL values. Some of the values have 'minute' or 'minutes' or 'mins' at the end. The data type is VARCHAR.
- `cancellation` contains NULL, NaN, and blank values. The data type is VARCHAR.

#### Steps:
- Create a temporary table `cleaned_runner_orders` and modify `pickup_time`, `distance`, `duration`, and `cancellation`.
- `pickup_time`: Change blanks to NULL. # Check if this is needed
- `distance`: Remove 'km'.
- `duration`: Remove 'minutes', 'minute', and 'mins'.
- `cancellation`: Change NaN, blanks to NULL.

Creating a temporary table and modifying the above 4 columns. All the NaN and blank values are changed to NULL. The extra words at the end of the numbers are removed (using TRIM function) so that the data type of the column can be changed.
```sql
CREATE TEMPORARY TABLE cleaned_runner_orders AS
  SELECT
    order_id,
    runner_id,
    CASE
        WHEN pickup_time = 'null' THEN NULL
        ELSE pickup_time
    END AS pickup_time,
    CASE
        WHEN distance = 'null' THEN NULL
        WHEN distance LIKE '%km' THEN TRIM('km' FROM distance)
        ELSE distance
    END AS distance,
    CASE
        WHEN duration = 'null' THEN NULL
        WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration)
        WHEN duration LIKE '%mins' THEN TRIM('mins' FROM duration)
        WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)
        ELSE duration
    END AS duration,
    CASE
        WHEN cancellation LIKE '%null' OR cancellation = 'NaN' OR cancellation = '' THEN NULL
        ELSE cancellation
    END AS cancellation
  FROM pizza_runner.runner_orders;
 ```
#### Steps:
- `pickup_time`: Change data type to TIMESTAMP.
- `distance`: Change data type to FLOAT.
- `duration`: Change data type to INTEGER.

```sql
ALTER TABLE cleaned_runner_orders
ALTER COLUMN pickup_time TYPE TIMESTAMP USING pickup_time::TIMESTAMP;

ALTER TABLE cleaned_runner_orders
ALTER COLUMN distance TYPE FLOAT USING distance::FLOAT;

ALTER TABLE cleaned_runner_orders
ALTER COLUMN duration TYPE INTEGER USING duration::INTEGER;
 ```

| order_id | runner_id | pickup_time          | distance | duration | cancellation            |
|----------|-----------|----------------------|----------|----------|--------------------------|
| 1        | 1         | 2020-01-01 18:15:34  | 20       | 32       | null                     |
| 2        | 1         | 2020-01-01 19:10:54  | 20       | 27       | null                     |
| 3        | 1         | 2020-01-03 00:12:37  | 13.4     | 20       | null                     |
| 4        | 2         | 2020-01-04 13:53:03  | 23.4     | 40       | null                     |
| 5        | 3         | 2020-01-08 21:10:57  | 10       | 15       | null                     |
| 6        | 3         | null                 | null     | null     | Restaurant Cancellation   |
| 7        | 2         | 2020-01-08 21:30:45  | 25       | 25       | null                     |
| 8        | 2         | 2020-01-10 00:15:02  | 23.4     | 15       | null                     |
| 9        | 2         | null                 | null     | null     | Customer Cancellation     |
| 10       | 1         | 2020-01-11 18:50:20  | 10       | 10       | null                     |

