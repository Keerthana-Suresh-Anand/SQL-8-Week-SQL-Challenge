# [#2 - Pizza Runner](https://8weeksqlchallenge.com/case-study-2/)
# C. Ingredient Optimisation
## Questions and Solutions

**1. What are the standard ingredients for each pizza?**
```sql
SELECT 
  n.pizza_name, 
  GROUP_CONCAT(t.topping_name) AS toppings
FROM pizza_recipes_transposed r
JOIN pizza_toppings t
  ON r.toppings = t.topping_id
JOIN pizza_names n
  ON r.pizza_id = n.pizza_id
GROUP BY n.pizza_name;
```
|pizza_name|toppings|
|--|--|
|Meatlovers|	Bacon,BBQ Sauce,Beef,Cheese,Chicken,Mushrooms,Pepperoni,Salami|
|Vegetarian|	Cheese,Mushrooms,Onions,Peppers,Tomatoes,Tomato Sauce|
---
**2. What was the most commonly added extra?**
```sql
SELECT p.topping_name, COUNT(e.extra_id) AS no_of_times_added 
FROM extras e
JOIN pizza_toppings p
ON e.extra_id = p.topping_id
GROUP BY p.topping_name
ORDER BY no_of_times_added DESC
LIMIT 1;
```
|topping_name|no_of_times_added|
|--|--|
|Bacon|4|
---
**3. What was the most common exclusion?**
```sql
SELECT p.topping_name, COUNT(e.exclusion_id) AS no_of_times_removed
FROM exclusions e
JOIN pizza_toppings p
ON e.exclusion_id = p.topping_id
GROUP BY p.topping_name
ORDER BY no_of_times_removed DESC
LIMIT 1;
```
|topping_name|no_of_times_removed|
|--|--|
|Cheese|4|
---
**4. Generate an order item for each record in the customers_orders table in the format of one of the following:**
- **Meat Lovers**
- **Meat Lovers - Exclude Beef**
- **Meat Lovers - Extra Bacon**
- **Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers**
```sql
```
---

**5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients**
- **For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"**
```sql
```
---
**6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?**
```sql
```
