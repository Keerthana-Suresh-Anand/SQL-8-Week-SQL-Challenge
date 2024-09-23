# [#1 - Danny's Diner](https://8weeksqlchallenge.com/case-study-1/)
[<img src="https://github.com/Keerthana-Suresh-Anand/SQL-8-Week-SQL-Challenge/raw/main/1.png" alt="Danny's Diner" width="200"/>](https://8weeksqlchallenge.com/case-study-1/)

**1. What is the total amount each customer spent at the restaurant?**
```sql
SELECT sales.customer_id,
       SUM(menu.price) AS total_amount_spent
FROM dannys_diner.sales
JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY total_amount_spent DESC;
```
**2. How many days has each customer visited the restaurant?**
```sql
SELECT customer_id, 
       COUNT(DISTINCT order_date)
FROM dannys_diner.sales
GROUP BY customer_id;
```
**3. What was the first item from the menu purchased by each customer?**
```sql
WITH first_item_ordered AS
(
SELECT customer_id,
       product_id,
ROW_NUMBER() OVER(PARTITION BY customer_id) row_numb
FROM dannys_diner.sales
)
SELECT customer_ID,
       product_id AS first_item_purchased
FROM first_item_ordered
WHERE row_numb = 1;
```
**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**
```sql
SELECT m.product_name AS most_purchased_item,
       COUNT(s.product_id) no_of_times_purchased
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
  ON s.product_id = m.product_id 
GROUP BY m.product_name
ORDER BY no_of_times_purchased DESC
LIMIT 1;
```
**5. Which item was the most popular for each customer?**
```sql
WITH most_populat_items_per_customer AS
(
SELECT s.customer_id,
       m.product_name, COUNT(s.product_id) AS no_of_times_ordered,
DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY COUNT(s.product_id) DESC) dense_rank_per_customer
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
  ON s.product_id = m.product_id
GROUP BY s.customer_id, m.product_name
ORDER BY s.customer_id
)
SELECT customer_id, product_name AS most_popular_item FROM most_populat_items_per_customer
WHERE dense_rank_per_customer = 1;
```
**6. Which item was purchased first by the customer after they became a member?**
```sql
WITH item_ordered_after_member AS
(
SELECT sales.customer_id,
       menu.product_name,
ROW_NUMBER() OVER(PARTITION BY sales.customer_id) as row_numb
FROM dannys_diner.sales JOIN dannys_diner.members
  ON sales.customer_id = members.customer_id
JOIN dannys_diner.menu
  ON sales.product_id = menu.product_id
WHERE sales.order_date >= members.join_date
)
SELECT customer_id, product_name AS item_order_after_becoming_member FROM item_ordered_after_member
WHERE row_numb = 1;
```
**7. Which item was purchased just before the customer became a member?**
```sql
WITH CTE_last_item_ordered_before_membership AS
(
SELECT s.customer_id,
       s.order_date,
       m.product_name, 
ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS row_numb
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
  ON s.product_id = m.product_id
JOIN dannys_diner.members
  ON s.customer_id = members.customer_id 
WHERE s.order_date < members.join_date
),
CTE_max_row_number AS
(
SELECT customer_id,
       MAX(row_numb) AS max_row_numb
FROM CTE_last_item_ordered_before_membership
GROUP BY customer_id
)
SELECT cte1.customer_id,
       cte1.product_name
FROM CTE_last_item_ordered_before_membership cte1
JOIN CTE_max_row_number cte2 
  ON cte1.customer_id = cte2.customer_id
  AND cte1.row_numb = cte2.max_row_numb;
```
**8. What is the total items and amount spent for each member before they became a member?**
```sql
SELECT s.customer_id,
       COUNT(s.product_id) AS no_of_items_ordered,
       SUM(m.price) AS amount_spent
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
  ON s.product_id = m.product_id
JOIN dannys_diner.members
  ON s.customer_id = members.customer_id
WHERE s.order_date < members.join_date
GROUP BY s.customer_id
ORDER BY s.customer_id;
```
**9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
```sql
SELECT s.customer_id,
       TO_CHAR(s.order_date, 'YYYY-MM-DD') AS order_date,
       m.product_name,
       m.price,
       CASE
            WHEN s.customer_id = members.customer_id AND s.order_date >= members.join_date THEN 'Y'
            ELSE 'N'
       END AS member
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
  ON s.product_id = m.product_id
LEFT JOIN dannys_diner.members
  ON s.customer_id = members.customer_id
ORDER BY s.customer_id,
         s.order_date;
```
**10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**
```sql
WITH cte_member AS
(
SELECT s.customer_id,
       TO_CHAR(s.order_date, 'YYYY-MM-DD') AS order_date,
       m.product_name,
       m.price,
       CASE
           WHEN s.customer_id = members.customer_id AND s.order_date >= members.join_date THEN 'Y'
           ELSE 'N'
       END AS member
FROM dannys_diner.sales s
JOIN dannys_diner.menu m
  ON s.product_id = m.product_id
LEFT JOIN dannys_diner.members
  ON s.customer_id = members.customer_id
)
SELECT customer_id,
       order_date,
       product_name,
       price,
       CASE
           WHEN member = 'Y' THEN DENSE_RANK() OVER(PARTITION BY customer_id,member ORDER BY order_date)
           ELSE NULL
       END AS ranking
FROM cte_member
ORDER BY customer_id,
         order_date;
```
