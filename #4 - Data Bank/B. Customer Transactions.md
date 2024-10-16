# [#4 - Data Bank](https://8weeksqlchallenge.com/case-study-3/)

# B. Customer Transactions

## Questions and Solutions
**1. What is the unique count and total amount for each transaction type?**
```sql
SELECT 
  txn_type, 
  COUNT(*) AS total_transactions, 
  SUM(txn_amount) AS total_amount 
FROM customer_transactions
GROUP BY txn_type;
```
|txn_type|total_transactions|total_amount|
|--|--|--|
|withdrawal	|1580|	793003|
|deposit	|2671|	1359168|
|purchase	|1617|	806537|
---
**2. What is the average total historical deposit counts and amounts for all customers?**

**3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?**
**4. What is the closing balance for each customer at the end of the month?**
**5. What is the percentage of customers who increase their closing balance by more than 5%?**
