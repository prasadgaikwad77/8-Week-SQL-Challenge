### 1. What is the total amount each customer spent at the restaurant?

```sql
SELECT S.customer_id
	,SUM(M.price) AS total_amount
FROM [dannys_diner].[sales] S
INNER JOIN [dannys_diner].[menu] M ON S.product_id = M.product_id
GROUP BY S.customer_id
```

### 2. How many days has each customer visited the restaurant?

```sql
SELECT customer_id
	,COUNT(DISTINCT order_date) AS Days
FROM [dannys_diner].[sales]
GROUP BY customer_id
```

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

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT TOP 1 M.product_name
	,count(*) AS Count
FROM [dannys_diner].[sales] S
INNER JOIN [dannys_diner].[menu] M ON S.product_id = M.product_id
GROUP BY M.product_name
ORDER BY count(*) DESC
```

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