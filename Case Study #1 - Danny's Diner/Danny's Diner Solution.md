### 1. What is the total amount each customer spent at the restaurant?

```sql
SELECT S.customer_id
	,SUM(M.price) AS total_amount
FROM [dannys_diner].[sales] S
INNER JOIN [dannys_diner].[menu] M ON S.product_id = M.product_id
GROUP BY S.customer_id
```
![image](https://github.com/prasadgaikwad77/8-Week-SQL-Challenge/assets/74951542/c9ee333a-64e9-4e3b-b7ad-4e46520d261c)


### 2. How many days has each customer visited the restaurant?

```sql
SELECT customer_id
	,COUNT(DISTINCT order_date) AS Days
FROM [dannys_diner].[sales]
GROUP BY customer_id
```
![image](https://github.com/prasadgaikwad77/8-Week-SQL-Challenge/assets/74951542/8b492d11-5145-4d89-986f-06df550fde38)


### 3. What was the first item from the menu purchased by each customer?

```sql
WITH TEMP
AS (
	SELECT DISTINCT DENSE_RANK() OVER (
			PARTITION BY customer_id ORDER BY order_date
			) AS RowNum
		,S.customer_id
		,M.product_name
	FROM [dannys_diner].[sales] S
	INNER JOIN [dannys_diner].[menu] M ON M.product_id = S.product_id
	)
SELECT customer_id
	,STRING_AGG(product_name, ',') AS Product
FROM TEMP
WHERE RowNum = 1
GROUP BY customer_id
```
![image](https://github.com/prasadgaikwad77/8-Week-SQL-Challenge/assets/74951542/97bed303-1bed-423e-98f7-a0cd7086c530)


### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT TOP 1 M.product_name
	,count(*) AS Count
FROM [dannys_diner].[sales] S
INNER JOIN [dannys_diner].[menu] M ON S.product_id = M.product_id
GROUP BY M.product_name
ORDER BY count(*) DESC
```
![image](https://github.com/prasadgaikwad77/8-Week-SQL-Challenge/assets/74951542/85ff52fd-02b8-483a-a4e5-24e9974be15d)


### 5. Which item was the most popular for each customer?

```sql
WITH orders
AS (
	SELECT product_name
		,customer_id
		,count(product_name) AS order_count
		,rank() OVER (
			PARTITION BY customer_id ORDER BY count(product_name) DESC
			) AS rank
	FROM dannys_diner.menu
	INNER JOIN dannys_diner.sales ON menu.product_id = sales.product_id
	GROUP BY customer_id
		,product_name
	)
SELECT customer_id
	,product_name
	,order_count
FROM orders
WHERE rank = 1
ORDER BY customer_id
```
![image](https://github.com/prasadgaikwad77/8-Week-SQL-Challenge/assets/74951542/4c2f7993-cff0-4a9b-b238-f97eedd5e3a5)

### 6. Which item was purchased first by the customer after they became a member?
```sql
WITH Sales
AS (
	SELECT S.customer_id
		,S.order_date
		,S.product_id
		,DENSE_RANK() OVER (
			PARTITION BY S.customer_id ORDER BY S.order_date ASC
			) AS Rnum
	FROM [dannys_diner].[sales] S
	INNER JOIN [dannys_diner].[members] M ON M.customer_id = S.customer_id
	WHERE S.order_date >= M.join_date
	)
SELECT S.customer_id
	,M.product_name
	,S.order_date
FROM Sales S
INNER JOIN [dannys_diner].[menu] M ON M.product_id = S.product_id
WHERE Rnum = 1
```
![image](https://github.com/prasadgaikwad77/8-Week-SQL-Challenge/assets/74951542/18b2fd91-4cb5-4b72-97e0-f8586f0b7e86)


### 7. Which item was purchased just before the customer became a member?
```sql
WITH Sales
AS (
	SELECT S.customer_id
		,S.order_date
		,S.product_id
		,DENSE_RANK() OVER (
			PARTITION BY S.customer_id ORDER BY S.order_date DESC
			) AS Rnum
	FROM [dannys_diner].[sales] S
	INNER JOIN [dannys_diner].[members] M ON M.customer_id = S.customer_id
	WHERE S.order_date < M.join_date
	)
SELECT S.customer_id
	,M.product_name
	,S.order_date
FROM Sales S
INNER JOIN [dannys_diner].[menu] M ON M.product_id = S.product_id
WHERE Rnum = 1
```
![image](https://github.com/prasadgaikwad77/8-Week-SQL-Challenge/assets/74951542/0b8b39d3-7145-401a-8085-5acb6c76fd11)


### 8. What is the total items and amount spent for each member before they became a member?
```sql
WITH Sales
AS (
	SELECT S.customer_id
		,S.order_date
		,S.product_id
		,DENSE_RANK() OVER (
			PARTITION BY S.customer_id ORDER BY S.order_date ASC
			) AS Rnum
	FROM [dannys_diner].[sales] S
	INNER JOIN [dannys_diner].[members] M ON M.customer_id = S.customer_id
	WHERE S.order_date < M.join_date
	)
SELECT S.customer_id
	,COUNT(M.product_name) AS total_items
	,SUM(M.price) AS amount_spent
FROM Sales S
INNER JOIN [dannys_diner].[menu] M ON M.product_id = S.product_id
GROUP BY S.customer_id
```
![image](https://github.com/prasadgaikwad77/8-Week-SQL-Challenge/assets/74951542/56b24663-5036-4dd9-aaf6-809101f0dc4a)

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
WITH CTE
AS (
	SELECT S.customer_id
		,CASE 
			WHEN M.product_name = 'sushi'
				THEN M.price * 20
			ELSE M.price * 10
			END AS points
	FROM [dannys_diner].[sales] S
	INNER JOIN [dannys_diner].[menu] M ON S.product_id = M.product_id
	)
SELECT customer_id
	,SUM(points) AS points
FROM CTE
GROUP BY customer_id
```
![image](https://github.com/prasadgaikwad77/8-Week-SQL-Challenge/assets/74951542/9971e922-7a71-4459-8b54-f57e846f43c8)

### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql
WITH cte
AS (
	SELECT *
		,DATEADD(DAY, 6, join_date) AS valid_date
		,EOMONTH('2021-01-31') AS last_date
	FROM [dannys_diner].[members] AS m
	)
SELECT d.customer_id
	,SUM(CASE 
			WHEN m.product_name = 'sushi'
				THEN 2 * 10 * m.price
			WHEN s.order_date BETWEEN d.join_date
					AND d.valid_date
				THEN 2 * 10 * m.price
			ELSE 10 * m.price
			END) AS points
FROM cte AS d
JOIN [dannys_diner].[sales] AS s ON d.customer_id = s.customer_id
JOIN [dannys_diner].[menu] AS m ON s.product_id = m.product_id
WHERE s.order_date < d.last_date
GROUP BY d.customer_id
```
![image](https://github.com/prasadgaikwad77/8-Week-SQL-Challenge/assets/74951542/325952b1-bddd-4d36-acd8-15bd1027d40f)

### Bonus Questions
```sql
WITH CTE
AS (
	SELECT S.customer_id
		,S.order_date
		,M.product_name
		,M.price
		,CASE 
			WHEN Me.customer_id IS NOT NULL
				AND Me.join_date <= S.order_date
				THEN 'Y'
			ELSE 'N'
			END AS member
	FROM [dannys_diner].[sales] S
	INNER JOIN [dannys_diner].[menu] M ON S.product_id = M.product_id
	LEFT JOIN [dannys_diner].[members] Me ON Me.customer_id = S.customer_id
	)
SELECT *
	,CASE 
		WHEN member = 'N'
			THEN NULL
		ELSE DENSE_RANK() OVER (
				PARTITION BY customer_id
				,member ORDER BY order_date
				)
		END AS ranking
FROM CTE
```
![image](https://github.com/prasadgaikwad77/8-Week-SQL-Challenge/assets/74951542/40bb35c9-9c02-423f-8d2e-098b2bb31f66)

