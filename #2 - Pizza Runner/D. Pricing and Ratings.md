# [#2 - Pizza Runner](https://8weeksqlchallenge.com/case-study-2/)
# D. Pricing and Ratings
## Questions and Solutions

**1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes - how much money has Pizza Runner made so far if there are no delivery fees?**

```sql
SELECT 
  SUM(CASE 
        WHEN c.pizza_id = 1 THEN 12
        WHEN c.pizza_id = 2 THEN 10
      END) AS revenue
FROM cleaned_customer_orders c
JOIN cleaned_runner_orders r
  ON c.order_id = r.order_id
WHERE r.cancellation IS NULL;
```
|revenue|
|--|
|138|
---
**2. What if there was an additional $1 charge for any pizza extras?
Add cheese is $1 extra**
```sql
SELECT 
  SUM(CASE 
        WHEN c.pizza_id = 1 THEN 12 + CASE
                                        WHEN c.extras IS NOT NULL THEN LENGTH(REPLACE(REPLACE(extras,',',''),' ',''))
                                        ELSE 0
                                      END
        WHEN c.pizza_id = 2 THEN 10 + CASE
                                        WHEN c.extras IS NOT NULL THEN LENGTH(REPLACE(REPLACE(extras,',',''),' ',''))
                                        ELSE 0
                                      END
        ELSE 0
      END) AS revenue_with_extra_charge
FROM cleaned_customer_orders c
JOIN cleaned_runner_orders r
  ON c.order_id = r.order_id
WHERE r.cancellation IS NULL;
```
|revenue_with_extra_charge|
|--|
|142|
---
**3.The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset - generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.**
```sql
CREATE TABLE runner_ratings(
	order_id INTEGER,
    rating INTEGER);
```
```sql
INSERT INTO runner_ratings(order_id, rating)
VALUES
(1,2),
(2,5),
(3,1),
(4,3),
(5,2),
(7,5),
(8,4),
(10,5);
```
```sql
SELECT * FROM runner_ratings;
```
|order_id|rating|
|----|----|
|1|2|
|2|5|
|3|1|
|4|3|
|5|2|
|7|5|
|8|4|
|10|5|
---
**4.Using your newly generated table - can you join all of the information together to form a table which has the following information for successful deliveries?
customer_id
order_id
runner_id
rating
order_time
pickup_time
Time between order and pickup
Delivery duration
Average speed
Total number of pizzas**
```sql
SELECT 
  c.customer_id, 
  c.order_id, 
  r.runner_id, rr.rating, c.order_time, r.pickup_time, 
  timestampdiff(MINUTE, c.order_time, r.pickup_time) AS time_between_order_and_pickup, 
  r.duration, 
  ROUND(distance/duration*60,2) AS speed, 
  COUNT(c.pizza_id) AS no_of_pizzas
FROM cleaned_customer_orders c
JOIN cleaned_runner_orders r
  ON c.order_id = r.order_id
JOIN runner_ratings rr
  ON rr.order_id = r.order_id
WHERE r.cancellation IS NULL
GROUP BY 
  c.customer_id, 
  c.order_id, 
  r.runner_id, 
  rr.rating, 
  c.order_time, 
  r.pickup_time, 
  time_between_order_and_pickup, 
  r.duration, 
  speed
ORDER BY customer_id;
```
| customer_id | order_id | runner_id | rating | order_time           | pickup_time           | time_between_order_pickup | duration | speed | no_of_pizzas |
|-------------|----------|----------|-----------|----------------------|-----------------------|----------|----------|--------|----------------|
| 101         | 1        | 1        | 2         | 2020-01-01 18:05:02  | 2020-01-01 18:15:34   | 10       | 32       | 37.5   | 1              |
| 101         | 2        | 1        | 5         | 2020-01-01 19:00:52  | 2020-01-01 19:10:54   | 10       | 27       | 44.44  | 1              |
| 102         | 3        | 1        | 1         | 2020-01-02 23:51:23  | 2020-01-03 00:12:37   | 21       | 20       | 40.2   | 2              |
| 102         | 8        | 2        | 4         | 2020-01-09 23:54:33  | 2020-01-10 00:15:02   | 20       | 15       | 93.6   | 1              |
| 103         | 4        | 2        | 3         | 2020-01-04 13:23:46  | 2020-01-04 13:53:03   | 29       | 40       | 35.1   | 3              |
| 104         | 5        | 3        | 2         | 2020-01-08 21:00:29  | 2020-01-08 21:10:57   | 10       | 15       | 40     | 1              |
| 104         | 10       | 1        | 5         | 2020-01-11 18:34:49  | 2020-01-11 18:50:20   | 15       | 10       | 60     | 2              |
| 105         | 7        | 2        | 5         | 2020-01-08 21:20:29  | 2020-01-08 21:30:45   | 10       | 25       | 60     | 1              |

---
**5.If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled - how much money does Pizza Runner have left over after these deliveries?**
```sql
WITH cte_revenue AS
(
SELECT c.order_id,
  SUM(CASE 
        WHEN c.pizza_id = 1 THEN 12
        WHEN c.pizza_id = 2 THEN 10
      END) AS revenue
FROM cleaned_customer_orders c
GROUP BY c.order_id
)
SELECT ROUND(SUM(revenue) - SUM(distance) * 0.3,2) AS amount_left_after_deliveries
FROM cleaned_runner_orders r
JOIN cte_revenue cte
  ON cte.order_id = r.order_id
WHERE r.cancellation IS NULL;
```
|amount_left_after_deliveries|
|--|
|94.44|
