# Pizza-Sales-Project-SQL
This project explores SQL queries to analyze a pizza sales dataset. It progressively demonstrates fundamental to advanced SQL functions, including Common Table Expressions (CTEs), window functions, subqueries, UPDATE statements, CASE WHEN expressions, and RANK functions. The analysis aims to answer various business-oriented questions about pizza sales.

- Retrieve the total number of orders placed.
- Calculate the total revenue generated from pizza sales.
- Identify the highest-priced pizza.
- Identify the most common pizza size ordered.
- List the top 5 most ordered pizza types along with their quantities.
- Join the necessary tables to find the total quantity of each pizza category ordered.
- Determine the distribution of orders by hour of the day.
- Join relevant tables to find the category-wise distribution of pizzas.
- Group the orders by date and calculate the average number of pizzas ordered per day.
- Determine the top 3 most ordered pizza types based on revenue.
- Calculate the percentage contribution of each pizza type to total revenue.
- Analyze the cumulative revenue generated over time.
- Determine the top 3 most ordered pizza types based on revenue for each pizza category.

  -----------------


***1. Retrieve the total number of orders placed.***
```
SELECT 
	COUNT(ORDER_ID) AS TOTAL_ORDERS
FROM 
	PortfolioProject..orders
```

***2. Calculate the total revenue generated from pizza sales.***
```
SELECT 
	SUM(quantity*price) AS TOTAL_REVENUE
FROM 
	PortfolioProject..order_details OD
JOIN 
	PortfolioProject..pizzas P 
ON 
	OD.pizza_id = P.pizza_id
```
***3. Identify the highest-priced pizza.***
```
SELECT
	TOP(1) price
FROM 
	PortfolioProject..pizzas
```
***4. Identify the most common pizza size ordered.***
```
SELECT 
	size, COUNT(size) AS SIZE_ORD
FROM 
	PortfolioProject..order_details OD
JOIN 
	PortfolioProject..pizzas P
ON
	OD.pizza_id=P.pizza_id 
GROUP BY 
	size
ORDER BY 
	SIZE_ORD DESC
```
***5. List the top 5 most ordered pizza types along with their quantities.***
```
SELECT 
	TOP(5) name, SUM(quantity) AS QUANTITIES
FROM 
	PortfolioProject..order_details OD
JOIN 
	PortfolioProject..pizzas P 
	ON OD.pizza_id=P.pizza_id
JOIN
	PortfolioProject..pizza_types PT 
	ON P.pizza_type_id=PT.pizza_type_id
GROUP BY name
ORDER BY QUANTITIES DESC
```
***6. Join the necessary tables to find the total quantity of each pizza category ordered.***
```
SELECT 
	category, SUM(quantity) AS TOTAL_QUANTITY
FROM 
	PortfolioProject..order_details OD
JOIN 
	PortfolioProject..pizzas P 
	ON OD.pizza_id=P.pizza_id
JOIN 
	PortfolioProject..pizza_types PT 
	ON P.pizza_type_id=PT.pizza_type_id
GROUP BY category
ORDER BY TOTAL_QUANTITY DESC
```
***7. Determine the distribution of orders by hour of the day.***
```
SELECT 
	DATEPART(HOUR, TIME) AS hour, COUNT(order_id) 
FROM 
	PortfolioProject..orders
GROUP BY 
	DATEPART(HOUR, TIME)
ORDER BY 
	DATEPART(HOUR, TIME)
```
***8. Join relevant tables to find the category-wise distribution of pizzas.***
```
SELECT 
	category, COUNT(NAME) AS type
FROM 
	PortfolioProject..pizza_types
GROUP BY 
	category
```
***9. Group the orders by date and calculate the average number of pizzas ordered per day***
```
SELECT 
	ROUND(AVG(CNT),0) AS AVG_PIZZA_DAY 
FROM(
	SELECT 
		DATE, sum(quantity) AS CNT
	FROM 
		PortfolioProject..order_details OD
	JOIN 
		PortfolioProject..orders O 
	ON
		OD.order_id=O.order_id
	GROUP BY
		date) 
			AS order_quantity
```
***10. Determine the top 3 most ordered pizza types based on revenue.***
```
SELECT 
	TOP 3 NAME, SUM(quantity * price) AS TOTAL_REV
FROM 
	PortfolioProject..order_details OD 
JOIN 
	PortfolioProject..pizzas P 
	ON OD.pizza_id=P.pizza_id
JOIN 
	PortfolioProject..pizza_types PT 
	ON P.pizza_type_id=PT.pizza_type_id
GROUP BY 
	NAME
ORDER BY 
	TOTAL_REV DESC
```
***11. Calculate the percentage contribution of each pizza type to total revenue.***
```
SELECT 
	category, 
	SUM(quantity * price) AS TOTAL_REV, 
	ROUND(100 * SUM(quantity * price)/
					(SELECT SUM(quantity * price) 
					FROM 
						PortfolioProject..order_details 
					JOIN
						PortfolioProject..pizzas 
						ON order_details.pizza_id=pizzas.pizza_id), 2) 
					AS PCT_REV
FROM 
	PortfolioProject..order_details OD 
JOIN 
	PortfolioProject..pizzas P
	ON OD.pizza_id=P.pizza_id
JOIN 
	PortfolioProject..pizza_types PT
	ON P.pizza_type_id=PT.pizza_type_id
GROUP BY
	category
```
***12. Analyze the cumulative revenue generated over time.***
```
SELECT DATE, 
    ROUND(SUM(REVENUE) OVER (ORDER BY DATE),1) AS CumulativeRevenue
FROM
    (SELECT 
		DATE, SUM(quantity * price) AS REVENUE
    FROM 
		PortfolioProject..orders O
    JOIN 
		PortfolioProject..order_details OD 
		ON O.order_id = OD.order_id
    JOIN 
		PortfolioProject..pizzas P 
		ON OD.pizza_id = P.pizza_id
    JOIN 
		PortfolioProject..pizza_types PT 
		ON P.pizza_type_id = PT.pizza_type_id
    GROUP BY DATE) AS DATE_REV;
```
***13. Determine the top 3 most ordered pizza types based on revenue for each pizza category.***
```
SELECT 
	NAME, REVENUE 
FROM (SELECT 
		CATEGORY, NAME, REVENUE, 
		RANK() OVER(PARTITION BY CATEGORY ORDER BY REVENUE DESC) AS RN 
	FROM 
		(SELECT CATEGORY, NAME, SUM(QUANTITY*PRICE) AS REVENUE 
	FROM 
		PortfolioProject..order_details OD
	JOIN 
		PortfolioProject..pizzas P 
		ON OD.pizza_id=P.pizza_id
	JOIN 
		PortfolioProject..pizza_types PT 
		ON P.pizza_type_id=PT.pizza_type_id
	GROUP BY category, NAME) AS CAT_REV) AS RANK__BY_CAT
WHERE RN <=3;
```
