# [#3 - Foodie-Fi](https://8weeksqlchallenge.com/case-study-3/)
[<img src="https://github.com/Keerthana-Suresh-Anand/SQL-8-Week-SQL-Challenge/raw/main/Images/%233.png" alt="Pizza Runnerr" width="180"/>](https://8weeksqlchallenge.com/case-study-3/)

## Problem Statement
Danny owns a food-based digital subscription service offering multiple subscription plans. He needs answers for important business questions.

## Entity Relationship Diagram

<img src = "https://github.com/Keerthana-Suresh-Anand/SQL-8-Week-SQL-Challenge/raw/main/Images/%233%20-%20ERD.png" width="700">

## A. Customer Journey
### Questions and Solutions

**1. Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.**
- Sample data:

| customer_id | plan_id | start_date  |
|-------------|---------|-------------|
| 1           | 0       | 2020-08-01  |
| 1           | 1       | 2020-08-08  |
| 2           | 0       | 2020-09-20  |
| 2           | 3       | 2020-09-27  |
| 11          | 0       | 2020-11-19  |
| 11          | 4       | 2020-11-26  |
| 13          | 0       | 2020-12-15  |
| 13          | 1       | 2020-12-22  |
| 13          | 2       | 2021-03-29  |
| 15          | 0       | 2020-03-17  |
| 15          | 2       | 2020-03-24  |
| 15          | 4       | 2020-04-29  |
| 16          | 0       | 2020-05-31  |
| 16          | 1       | 2020-06-07  |
| 16          | 3       | 2020-10-21  |
| 18          | 0       | 2020-07-06  |
| 18          | 2       | 2020-07-13  |
| 19          | 0       | 2020-06-22  |
| 19          | 2       | 2020-06-29  |
| 19          | 3       | 2020-08-29  |

```sql
SELECT 
  s.customer_id, 
  s.start_date, 
  p.plan_name, 
  p.price
FROM foodie_fi.plans p
JOIN foodie_fi.subscriptions s
  ON p.plan_id = s.plan_id
WHERE s.customer_id IN (1,2,11,13,15,16,18,19)
ORDER BY 
  s.customer_id, 
  s.start_date;
```
| customer_id | start_date  | plan_name     | price  |
|-------------|-------------|---------------|--------|
| 1           | 2020-08-01  | trial         | 0.00   |
| 1           | 2020-08-08  | basic monthly | 9.90   |
| 2           | 2020-09-20  | trial         | 0.00   |
| 2           | 2020-09-27  | pro annual    | 199.00 |
| 11          | 2020-11-19  | trial         | 0.00   |
| 11          | 2020-11-26  | churn         |        |
| 13          | 2020-12-15  | trial         | 0.00   |
| 13          | 2020-12-22  | basic monthly | 9.90   |
| 13          | 2021-03-29  | pro monthly   | 19.90  |
| 15          | 2020-03-17  | trial         | 0.00   |
| 15          | 2020-03-24  | pro monthly   | 19.90  |
| 15          | 2020-04-29  | churn         |        |
| 16          | 2020-05-31  | trial         | 0.00   |
| 16          | 2020-06-07  | basic monthly | 9.90   |
| 16          | 2020-10-21  | pro annual    | 199.00 |
| 18          | 2020-07-06  | trial         | 0.00   |
| 18          | 2020-07-13  | pro monthly   | 19.90  |
| 19          | 2020-06-22  | trial         | 0.00   |
| 19          | 2020-06-29  | pro monthly   | 19.90  |
| 19          | 2020-08-29  | pro annual    | 199.00 |

- Customer 1 started on the trial plan on Aug 1, 2020, and switched to the basic monthly after the trial.
- Customer 2 started on the trial plan on Sep 20, 2020, and switched to the pro annual after the trial.
- Customer 11 started on the trial plan on Nov 19, 2020, and canceled after the trial.
- Customer 13 started on the trial plan on Dec 15, 2020, and switched to the basic monthly after the trial. They switched to the pro monthly on Mar 29, 2021.
- Customer 15 started on the trial plan on Mar 17, 2020, and switched to the pro monthly after the trial. They canceled on Apr 29, 2020.
- Customer 16 started on the trial plan on May 31, 2020, and switched to the basic monthly after the trial. They switched to the pro annual on Oct 21, 2020.
- Customer 18 started on the trial plan on Jul 6, 2020, and switched to the pro monthly after the trial.
- Customer 19 started on the trial plan on Jun 22, 2020, and switched to the pro monthly after the trial. They switched to the pro annual on Aug 29, 2020.
