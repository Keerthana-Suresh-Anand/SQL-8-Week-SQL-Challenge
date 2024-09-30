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
