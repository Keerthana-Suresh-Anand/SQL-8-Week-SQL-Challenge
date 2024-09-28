# [#2 - Pizza Runner](https://8weeksqlchallenge.com/case-study-2/)
# A. Pizza Metrics
## Questions and Solutions

**1. How many pizzas were ordered?**

```sql
SELECT COUNT(order_id) AS pizzas_ordered 
FROM pizza_runner.customer_orders;
```

|pizzas_ordered|
|--------------|
|14|

**2. How many unique customer orders were made?**

```sql
SELECT COUNT(DISTINCT order_id) AS unique_customer_orders 
FROM pizza_runner.customer_orders;
```

|unique_customer_orders|
|--------------|
|10|


**3. How many successful orders were delivered by each runner?**

```sql
SELECT runner_id, COUNT(order_id) AS orders_successfully_delivered 
FROM cleaned_runner_orders
WHERE cancellation IS NULL
GROUP BY runner_id
ORDER BY runner_id;
```

|runner_id|orders_successfully_delivered|
|--------------|----|
|1|4|
|2|3|
|3|1|

**4. How many of each type of pizza was delivered?**

```sql
SELECT 
  p.pizza_name, 
  COUNT(r.order_id) AS no_of_times_ordered
FROM cleaned_customer_orders c
JOIN cleaned_runner_orders r
  ON c.order_id = r.order_id
JOIN pizza_runner.pizza_names p
  ON p.pizza_id = c.pizza_id
WHERE r.cancellation IS NULL
GROUP BY p.pizza_name;
```

|pizza_name|no_of_times_ordered|
|--------------|----|
|Meatlovers|9|
|Vegetarian|3|

**5. How many Vegetarian and Meatlovers were ordered by each customer?**

```sql
SELECT 
  c.customer_id, 
  p.pizza_name, 
  COUNT(c.order_id) AS no_of_times_ordered
FROM cleaned_customer_orders c
JOIN pizza_runner.pizza_names p
  ON p.pizza_id = c.pizza_id
GROUP BY c.customer_id, p.pizza_name
ORDER BY c.customer_id, p.pizza_name;
```

|customer_id|pizza_name|no_of_times_ordered|
|--------------|----|-----|
| 101         | Meatlovers   | 2        |
| 101         | Vegetarian   | 1        |
| 102         | Meatlovers   | 2        |
| 102         | Vegetarian   | 1        |
| 103         | Meatlovers   | 3        |
| 103         | Vegetarian   | 1        |
| 104         | Meatlovers   | 3        |
| 105         | Vegetarian   | 1        |


**6. What was the maximum number of pizzas delivered in a single order?**

```sql
WITH cte_max_pizzas_in_single_order AS
(
SELECT 
  c.order_id, 
  COUNT(c.order_id) AS pizza_count
FROM cleaned_customer_orders c
JOIN cleaned_runner_orders r
  ON c.order_id = r.order_id
WHERE cancellation IS NULL
GROUP BY c.order_id
)
SELECT MAX(pizza_count) AS max_pizzas_in_single_order 
FROM cte_max_pizzas_in_single_order;
```

|max_pizzas_in_single_order|
|--------|
|3|

**7. For each customer, how many delivered pizzas had at least 1 change, and how many had no changes?**

```sql
SELECT 
  c.customer_id,
  COUNT(CASE
          WHEN c.exclusions IS NOT NULL OR c.extras IS NOT NULL THEN pizza_id
          ELSE NULL
        END) AS pizza_with_changes,
  COUNT(CASE
          WHEN c.extras IS NULL AND c.exclusions IS NULL THEN pizza_id
          ELSE NULL
        END) AS pizza_with_no_changes 
FROM cleaned_customer_orders c
JOIN cleaned_runner_orders r
  ON c.order_id = r.order_id
WHERE r.cancellation IS NULL
GROUP BY c.customer_id
ORDER BY c.customer_id;
```

| customer_id | pizza_with_changes | pizza_with_no_changes |
|-------------|------------|------------|
| 101         | 0          | 2          |
| 102         | 0          | 3          |
| 103         | 3          | 0          |
| 104         | 2          | 1          |
| 105         | 1          | 0          |


**8. How many pizzas were delivered that had both exclusions and extras?**

```sql
SELECT COUNT(c.pizza_id) AS pizzas_with_exclusions_extras_delivered
FROM cleaned_customer_orders c
JOIN cleaned_runner_orders r
  On c.order_id = r.order_id
WHERE r.cancellation IS NULL 
  AND c.exclusions IS NOT NULL 
  AND c.extras IS NOT NULL;
```

|pizzas_with_exclusions_extras_delivered|
|---------|
|1|

**9. What was the total volume of pizzas ordered for each hour of the day?**

```sql
SELECT 
  HOUR( order_time) AS hour_of_the_day, 
  COUNT(pizza_id) AS pizzas_ordered 
FROM cleaned_customer_orders
GROUP BY hour_of_the_day
ORDER BY hour_of_the_day;
```

| hour_of_the_day  | pizzas_ordered |
|--------|-------|
| 11     | 1     |
| 13     | 3     |
| 18     | 3     |
| 19     | 1     |
| 21     | 3     |
| 23     | 3     |


**10. What was the volume of orders for each day of the week?**

```sql
SELECT 
  dayname(order_time) AS day_of_the_week, 
  COUNT(pizza_id) AS pizzas_ordered 
FROM cleaned_customer_orders
GROUP BY day_of_the_week
ORDER BY pizzas_ordered DESC;
```

| day_of_the_week| pizzas_ordered |
|-----------|-------|
| Wednesday | 5     |
| Saturday  | 5     |
| Thursday  | 3     |
| Friday    | 1     |

