# [#4 - Data-Bank](https://8weeksqlchallenge.com/case-study-4/)
[<img src="https://github.com/Keerthana-Suresh-Anand/SQL-8-Week-SQL-Challenge/raw/main/Images/%234.png" alt="Pizza Runnerr" width="180"/>](https://8weeksqlchallenge.com/case-study-3/)

## Problem Statement
Danny owns a digital-only bank that offers banking activities and a distributed data storage platform. Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts. He needs help with increasing the total customer base and tracking how much data storage their customer will need.

## Entity Relationship Diagram

<img src = "https://github.com/Keerthana-Suresh-Anand/SQL-8-Week-SQL-Challenge/raw/main/Images/%234%20-%20ERD.png" width="700">

## A. Customer Nodes Exploration
### Questions and Solutions

**1. How many unique nodes are there on the Data Bank system?**
```sql
SELECT COUNT(DISTINCT node_id) AS distinct_nodes FROM customer_nodes;
```
|distinct_nodes|
|--|
|5|
---
**2. What is the number of nodes per region?**
```sql
SELECT 
  r.region_name, 
  COUNT(DISTINCT node_id) AS distinct_nodes
FROM customer_nodes n
JOIN regions r
  ON n.region_id = r.region_id
GROUP BY r.region_name;
```
|region_name|distinct_nodes|
|--|--|
|Africa|5|
|America|5|
|Asia|5|
|Australia|5|
|Europe|5|
---
**3. How many customers are allocated to each region?**
```sql
SELECT 
  r.region_name, ?
  COUNT(DISTINCT customer_id) AS distinct_customers
FROM customer_nodes n
JOIN regions r
ON n.region_id = r.region_id
GROUP BY r.region_name
ORDER BY distinct_customers DESC;
```
|region_name|distinct_customers|
|--|--|
|Australia|110|
|America|105|
|Africa|102|
|Asia|95|
|Europe|88|
---
**4. How many days on average are customers reallocated to a different node?**
```sql

```
---
**5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?**
```sql

```
---
