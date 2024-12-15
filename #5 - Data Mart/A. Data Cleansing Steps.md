# [#4 - Data-Mart](https://8weeksqlchallenge.com/case-study-5/)
[<img src="https://github.com/Keerthana-Suresh-Anand/SQL-8-Week-SQL-Challenge/raw/main/Images/%235.png" alt="Pizza Runnerr" width="180"/>](https://8weeksqlchallenge.com/case-study-5/)

## Problem Statement
Danny owns an international online supermarket - Data Mart - that made major changes in June 2020 to use sustainable packaging in all its products. Danny needs help to quantify the impact of this change on the sales performance.

The key business questions he wants to find answers to are the following:

- What was the quantifiable impact of the changes introduced in June 2020?
- Which platform, region, segment, and customer types were the most impacted by this change?
- What can we do about the future introduction of similar sustainability updates to the business to minimise impact on sales?
## Entity Relationship Diagram

<img src = "https://github.com/Keerthana-Suresh-Anand/SQL-8-Week-SQL-Challenge/raw/main/Images/%235%20-%20ERD.png" width="300">

## A. Data Cleansing Steps
### Questions and Solutions

**In a single query, perform the following operations and generate a new table in the `data_mart` schema named `clean_weekly_sales`:**

1. **Convert the `week_date` to a DATE format.**
2. **Add a `week_number` as the second column for each `week_date` value, for example, any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2, etc.**
3. **Add a `month_number` with the calendar month for each `week_date` value as the 3rd column.**
4. **Add a `calendar_year` column as the 4th column containing either 2018, 2019, or 2020 values.**
5. **Add a new column called `age_band` after the original `segment` column using the following mapping on the number inside the `segment` value:**

   | segment | age_band     |
   |-----------|----------------|
   | 1         | Young Adults   |
   | 2         | Middle Aged    |
   | 3 or 4    | Retirees       |

6. **Add a new `demographic` column using the following mapping for the first letter in the `segment` values:**

   | segment | demographic |
   |-----------|---------------|
   | C         | Couples       |
   | F         | Families      |

7. **Ensure all `NULL` string values are replaced with "unknown" in the original `segment` column, as well as in the new `age_band` and `demographic` columns.**
8. **Generate a new `avg_transaction` column as the `sales` value divided by `transactions`, rounded to 2 decimal places for each record.**

```sql
SELECT 
    modified_week_date,
    DATEPART(WEEK,modified_week_date) AS week_number,
    MONTH(modified_week_date) AS month_number,
    YEAR(modified_week_date) AS calendar_year,
    region,
    platform,
    REPLACE(segment,'null','unknown') AS segment,
    CASE
    	WHEN RIGHT(segment,1) = '1' THEN 'Young Adults'
    	WHEN RIGHT(segment,1) = '2' THEN 'Middle Aged'
    	WHEN RIGHT(segment,1) IN ('3','4') THEN 'Retirees'
    	ELSE 'unknown'
    END AS age_band,
    CASE
    	WHEN LEFT(segment,1) = 'C' THEN 'Couples'
    	WHEN LEFT(segment,1) = 'F' THEN 'Families'
    	ELSE 'unknown'
    END AS demographic,
    customer_type,
    transactions,
    sales,
    ROUND((CAST(sales AS float))/CAST(transactions AS float),2) AS avg_transaction  
INTO clean_weekly_sales
FROM 
(
SELECT 
    CONVERT(DATE, week_date, 3) AS modified_week_date,
    region,
    platform,
    segment,
    customer_type,
    transactions,
    sales
FROM weekly_sales) AS temp_table;
```

```sql
SELECT TOP 5 * FROM clean_weekly_sales;
```

| modified_week_date | week_number | month_number | calendar_year | region | platform | segment | age_band     | demographic | customer_type | transactions | sales   | avg_transaction |
|--------------------|-------------|--------------|---------------|--------|----------|---------|--------------|-------------|---------------|--------------|---------|-----------------|
| 2020-08-31         | 36          | 8            | 2020          | ASIA   | Retail   | C3      | Retirees     | Couples     | New           | 120631       | 3656163 | 30.31           |
| 2020-08-31         | 36          | 8            | 2020          | ASIA   | Retail   | F1      | Young Adults | Families    | New           | 31574        | 996575  | 31.56           |
| 2020-08-31         | 36          | 8            | 2020          | USA    | Retail   | unknown | unknown      | unknown     | Guest         | 529151       | 16509610| 31.2            |
| 2020-08-31         | 36          | 8            | 2020          | EUROPE | Retail   | C1      | Young Adults | Couples     | New           | 4517         | 141942  | 31.42           |
| 2020-08-31         | 36          | 8            | 2020          | AFRICA | Retail   | C2      | Middle Aged  | Couples     | New           | 58046        | 1758388 | 30.29           |
