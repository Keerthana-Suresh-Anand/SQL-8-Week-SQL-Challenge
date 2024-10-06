# [#2 - Pizza Runner](https://8weeksqlchallenge.com/case-study-3/)

# B. Data Analysis Questions

## Questions and Solutions

**1. How many customers has Foodie-Fi ever had??**

```sql
SELECT COUNT(DISTINCT customer_id) AS customer_count
FROM foodie_fi.subscriptions;
```
|customer_count|
|--|
|1000|
---
**2. What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value**
```sql
SELECT 
  Monthname(start_date) AS start_month, 
  COUNT(customer_id) AS customer_count
FROM foodie_fi.subscriptions
WHERE plan_id = 0
GROUP BY start_month
ORDER BY MONTH(start_date);
```
| start_month| customer_count |
|------------|----------------|
| January    | 88             |
| February   | 68             |
| March      | 94             |
| April      | 81             |
| May        | 88             |
| June       | 79             |
| July       | 89             |
| August     | 88             |
| September  | 87             |
| October    | 79             |
| November   | 75             |
| December   | 84             |
---
**3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name**
```sql
SELECT 
  p.plan_name, 
  COUNT(s.customer_id) AS customer_count
FROM foodie_fi.subscriptions s
JOIN foodie_fi.plans p
  ON s.plan_id = p.plan_id
WHERE EXTRACT(YEAR FROM s.start_date) > 2020
GROUP BY p.plan_name;
```
| plan_name      | customer_count |
|----------------|----------------|
| Churn          | 71             |
| Pro Monthly    | 60             |
| Pro Annual     | 63             |
| Basic Monthly  | 8              |
---
**4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?**
```sql
SELECT 
  COUNT(DISTINCT s.customer_id) AS churn_customers,
  ROUND(COUNT(DISTINCT s.customer_id)/(SELECT COUNT(DISTINCT customer_id) FROM subscriptions)*100,1) AS churn_rate
FROM foodie_fi.subscriptions s
JOIN foodie_fi.plans p
  ON s.plan_id = p.plan_id
WHERE p.plan_name = 'churn';
```
|churn_customers|churn_rate|
|--|--|
|307|30.7|
---
**5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?**
```sql
SELECT 
  COUNT(s1.customer_id) AS churn_after_trial, 
  ROUND(COUNT(s1.customer_id)/ (SELECT COUNT(DISTINCT customer_id) FROM subscriptions)*100,1) AS churn_after_trial_percentage
FROM subscriptions s1
JOIN subscriptions s2
  ON s1.customer_id = s2.customer_id
  AND s1.plan_id = 0
  AND s2.plan_id = 4
WHERE datediff(s2.start_date, s1.start_date) <= 7;
```
|churn_after_trial|churn_after_trial_percentage|
|--|--|
|92|9.2|
---
**6. What is the number and percentage of customer plans after their initial free trial?**
```sql
WITH cte_next_plan AS
(SELECT 
  s.customer_id, 
  p.plan_name,
  LEAD(p.plan_name) OVER (PARTITION BY s.customer_id ORDER BY s.start_date) AS next_plan
FROM subscriptions s
JOIN plans p
  ON s.plan_id = p.plan_id)
SELECT 
  next_plan AS plan, 
  COUNT(customer_id) AS no_of_customers, 
  ROUND(COUNT(customer_id)/ (SELECT COUNT(DISTINCT customer_id) FROM subscriptions) *100,1) AS percentage
FROM cte_next_plan
WHERE plan_name = 'trial'
GROUP BY next_plan;
```
|plan|no_of_customers|percentage|
|----------------|-------|------------|
| Basic Monthly  | 546   | 54.6%      |
| Pro Annual     | 37    | 3.7%       |
| Pro Monthly    | 325   | 32.5%      |
| Churn          | 92    | 9.2%       |
---
**7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?**
```sql
WITH RankedPlans AS (
  SELECT 
    s.customer_id, 
    s.plan_id, 
    p.plan_name,
    ROW_NUMBER() OVER (
      PARTITION BY s.customer_id 
      ORDER BY s.start_date DESC
    ) AS rn
  FROM subscriptions s
  JOIN plans p
    ON s.plan_id = p.plan_id
  WHERE s.start_date <= '2020-12-31'
)
SELECT 
  plan_id
  plan_name, 
  COUNT(customer_id) AS no_of_customers, 
  ROUND(COUNT(customer_id)/(SELECT COUNT(DISTINCT customer_id) FROM subscriptions)*100,1) AS percentage
FROM RankedPlans
WHERE rn = 1
GROUP BY plan_name
ORDER BY plan_id;
```
|plan_id|plan_name|no_of_customers|percentage|
|--|--|---|---|
|0|trial|	19	|1.9|
|1|basic monthly|	224	|22.4|
|2|pro monthly|	326	|32.6|
|3|pro annual|	195	|19.5|
|4|churn|	236|	23.6|
---
**8. How many customers have upgraded to an annual plan in 2020?**
```sql
WITH cte_previous_plans AS
(
  SELECT 
    s.customer_id, 
    s.start_date, 
    p.plan_name, 
    LAG(p.plan_name) OVER (PARTITION BY s.customer_id ORDER BY s.start_date) AS previous_plan
  FROM subscriptions s
  JOIN plans p
    ON s.plan_id = p.plan_id)
SELECT COUNT(DISTINCT customer_id) AS customers_upgraded_to_annual
FROM cte_previous_plans
WHERE 
  plan_name = 'pro annual'
  AND previous_plan NOT LIKE '%annual%' 
  AND year(start_date) = 2020;
```
|customers_upgraded_to_annual|
|--|
|195|
---
**9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?**
```sql
SELECT ROUND(AVG(datediff(s2.start_date, s1.start_date)),1) as avg_time_to_upgrade_to_annual
FROM subscriptions s1
JOIN subscriptions s2
 ON s1.customer_id = s2.customer_id
 AND s1.plan_id = 0 
 AND s2.plan_id = 3;
```
|avg_time_to_upgrade_to_annual|
|--|
|104.6|
---
**10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)**
```sql
WITH cte_days_to_upgrade_to_annual AS
 (SELECT s1.customer_id, s1.start_date AS trial_start_date, s2.start_date AS annual_upgrade_date, datediff(s2.start_date, s1.start_date) AS days_to_upgrade_to_annual
 FROM subscriptions s1
 JOIN subscriptions s2
 ON s1.customer_id = s2.customer_id
 AND s1.plan_id = 0 
 AND s2.plan_id = 3)
SELECT 
CASE
WHEN days_to_upgrade_to_annual BETWEEN 0 AND 30 THEN '0-30 days'
WHEN days_to_upgrade_to_annual BETWEEN 31 AND 60 THEN '31-60 days'
WHEN days_to_upgrade_to_annual BETWEEN 61 AND 90 THEN '61-90 days'
WHEN days_to_upgrade_to_annual BETWEEN 91 AND 120 THEN '91-120 days'
WHEN days_to_upgrade_to_annual BETWEEN 121 AND 150 THEN '121-150 days'
WHEN days_to_upgrade_to_annual BETWEEN 151 AND 180 THEN '151-180 days'
WHEN days_to_upgrade_to_annual BETWEEN 181 AND 210 THEN '181-210 days'
WHEN days_to_upgrade_to_annual BETWEEN 211 AND 240 THEN '211-240 days'
WHEN days_to_upgrade_to_annual BETWEEN 241 AND 270 THEN '241-270 days'
WHEN days_to_upgrade_to_annual BETWEEN 271 AND 300 THEN '271-300 days'
WHEN days_to_upgrade_to_annual BETWEEN 301 AND 330 THEN '301-330 days'
WHEN days_to_upgrade_to_annual BETWEEN 331 AND 360 THEN '331-360 days'
ELSE '361+ days'
END AS days_grouping,
COUNT(customer_id) AS no_of_customers
FROM cte_days_to_upgrade_to_annual
GROUP BY days_grouping;
```
| Days Grouping    | No. of Customers |
|------------------|------------------|
| 0-30 days        | 49               |
| 31-60 days       | 24               |
| 61-90 days       | 34               |
| 91-120 days      | 35               |
| 121-150 days     | 42               |
| 151-180 days     | 36               |
| 181-210 days     | 26               |
| 211-240 days     | 4                |
| 241-270 days     | 5                |
| 271-300 days     | 1                |
| 301-330 days     | 1                |
| 331-360 days     | 1                |
---
**11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?**
```sql
WITH cte_previous_plans AS
(
  SELECT 
    s.customer_id, 
    s.start_date, 
    p.plan_name, 
    LAG(p.plan_name) OVER (PARTITION BY s.customer_id ORDER BY s.start_date) AS previous_plan
  FROM subscriptions s
  JOIN plans p
    ON s.plan_id = p.plan_id)
SELECT COUNT(DISTINCT customer_id) AS customers_downgraded_from_pro_to_basic_monthly
FROM cte_previous_plans
  WHERE plan_name = 'basic monthly'
  AND previous_plan = 'pro monthly' 
  AND year(start_date) = 2020;
```
|customers_downgraded_from_pro_to_basic_monthly|
|--|
|0|
---
