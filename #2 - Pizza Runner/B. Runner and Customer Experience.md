# [#2 - Pizza Runner](https://8weeksqlchallenge.com/case-study-2/)

# B. Runner and Customer Experience

## Questions and Solutions

**1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)**

```sql
SELECT 
  DATE_ADD('2021-01-01', INTERVAL FLOOR(DATEDIFF(registration_date, '2021-01-01') / 7) * 7 DAY) AS week_start_date,
  COUNT(runner_id) AS runners_signed_up
FROM runners
GROUP BY week_start_date
ORDER BY week_start_date;
```

|week_start_date|runners_signed_up|
|--------------|---|
|2021-01-01|2|
|2021-01-08|1|
|2021-01-15|1|
---

**2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?**

```sql
SELECT 
  r.runner_id, 
  AVG(TIMESTAMPDIFF(MINUTE, c.order_time, r.pickup_time)) AS time_diff
FROM (SELECT DISTINCT order_id, order_time FROM cleaned_customer_orders) c
JOIN cleaned_runner_orders r
  ON c.order_id = r.order_id
WHERE r.cancellation IS NULL
GROUP BY runner_id;
```

|runner_id|time_diff|
|--------------|---|
|1|14|
|2|19.66|
|3|10|
---

**3. Is there any relationship between the number of pizzas and how long the order takes to prepare?**

```sql
WITH cte_pizza_preparation_time AS 
(
SELECT 
  c.order_id, 
  COUNT(c.pizza_id) AS no_of_pizzas, 
  timestampdiff(MINUTE, order_time, pickup_time) AS prep_time
FROM cleaned_customer_orders c
JOIN cleaned_runner_orders r
  ON c.order_id = r.order_id
WHERE cancellation IS NULL
GROUP BY order_id
)
SELECT 
  no_of_pizzas, 
  AVG(prep_time) AS avg_prep_time 
FROM cte_pizza_preparation_time
GROUP BY no_of_pizzas;
```

|no_of_pizzas|avg_prep_time|
|--------------|---|
|1|12.00|
|2|18.00|
|3|29.00|

- As the number of pizzas in an order increases, the time taken to prepare them increases as well.
---

**4. What was the average distance traveled for each customer?**

```sql
SELECT 
  customer_id, 
  ROUND(AVG(distance),2) AS average_distance
FROM cleaned_runner_orders r
JOIN cleaned_customer_orders c
  ON r.order_id = c.order_id
GROUP BY customer_id
ORDER BY customer_id;
```
|customer_id|average_distance|
|--------------|---|
|101|20|
|102|16.73|
|103|23.4|
|104|10|
|105|25|
---
**5. What was the difference between the longest and shortest delivery times for all orders?**
```sql
SELECT MAX(duration) - MIN(duration) as longest_minus_shortest_delivery
FROM cleaned_runner_orders;
```
|longest_minus_shortest_delivery|
|---|
|30|
---

**6. What was the average speed for each runner for each delivery and do you notice any trend for these values?**
```sql
SELECT 
  runner_id, 
  order_id, 
  ROUND(AVG(distance/duration) * 60,2) AS speed
 FROM cleaned_runner_orders
 WHERE cancellation IS NULL
 GROUP BY runner_id, order_id
 ORDER BY runner_id;
```
|runner_id|order_id|speed|
|--------------|---|---|
|1|1|37.5|
|1|2|44.44|
|1|3|40.2|
|1|10|60|
|2|4|35.1|
|2|7|40|
|2|8|93.6|
|3|5|40|
- Runner 1 averages from 37.5km/hr to 60km/hr.
- Runner 2 averages from 35.1km/hr to 93.6km/hr.
- Runner 3 averages from 40km/hr.
---
**7. What is the successful delivery percentage for each runner?**
```sql
WITH cte_deliveries AS
(
SELECT 
  runner_id, 
  COUNT(CASE
          WHEN cancellation IS NULL THEN order_id
	      END) AS successful_deliveries,
  COUNT(order_id) AS total_deliveries
FROM cleaned_runner_orders
GROUP BY runner_id
)
SELECT 
  runner_id, 
  successful_deliveries/total_deliveries * 100 AS successful_delivery_percentage
FROM cte_deliveries;
```
|runner_id|successful_delivery_percentage|
|--------------|---|
|1|100.00|
|2|75.00|
|3|50.00|
