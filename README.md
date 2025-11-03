# ☕ Monday Coffee – SQL Data Analysis Project  

### 📋 Overview  
**Monday Coffee** is a fictional coffee chain aiming to analyze its sales performance, customer behavior, and city-level growth potential using **PostgreSQL**.  

This project demonstrates end-to-end **data modeling, querying, and business analysis** using SQL. It includes schema creation, data relationships, and advanced analytics like window functions and CTEs.  

---

## 🧩 Database Schema

### Tables and Relationships  
| Table | Description |
|--------|--------------|
| **city** | Contains city-level data such as population, rent, and ranking. |
| **customers** | Stores customer details and their associated city. |
| **products** | Lists all coffee products sold by the company. |
| **sales** | Tracks each sales transaction, including total amount and product rating. |

Entity Relationship:  
`city` ⟶ `customers` ⟶ `sales` ⟶ `products`  

---

## 🛠️ Schema Design (DDL)

```sql
CREATE TABLE city (
	city_id INT PRIMARY KEY,
	city_name VARCHAR(15),
	population BIGINT,
	estimated_rent FLOAT,
	city_rank INT
);

CREATE TABLE customers (
	customer_id INT PRIMARY KEY,
	customer_name VARCHAR(25),
	city_id INT REFERENCES city(city_id)
);

CREATE TABLE products (
	product_id INT PRIMARY KEY,
	product_name VARCHAR(35),
	price FLOAT
);

CREATE TABLE sales (
	sale_id INT PRIMARY KEY,
	sale_date DATE,
	product_id INT REFERENCES products(product_id),
	customer_id INT REFERENCES customers(customer_id),
	total FLOAT,
	rating INT
);
```

---

## 📊 Data Analysis Queries

### 1️⃣ Coffee Consumers Estimation  
```sql
SELECT 
	city_name,
	ROUND((population * 0.25)/1000000, 2) AS coffee_consumers_in_millions,
	city_rank
FROM city
ORDER BY 2 DESC;
```

### 2️⃣ Total Revenue in Q4 2023  
```sql
SELECT SUM(total) AS total_revenue
FROM sales
WHERE EXTRACT(YEAR FROM sale_date) = 2023
  AND EXTRACT(QUARTER FROM sale_date) = 4;
```

### 3️⃣ Revenue by City (Q4 2023)  
```sql
SELECT 
	ci.city_name,
	SUM(s.total) AS total_revenue
FROM sales s
JOIN customers c ON s.customer_id = c.customer_id
JOIN city ci ON ci.city_id = c.city_id
WHERE EXTRACT(YEAR FROM s.sale_date) = 2023
  AND EXTRACT(QUARTER FROM s.sale_date) = 4
GROUP BY 1
ORDER BY 2 DESC;
```

### 4️⃣ Product Sales Count  
```sql
SELECT 
	p.product_name,
	COUNT(s.sale_id) AS total_orders
FROM products p
LEFT JOIN sales s ON s.product_id = p.product_id
GROUP BY 1
ORDER BY 2 DESC;
```

### 5️⃣ Average Sales Amount per Customer per City  
```sql
SELECT 
	ci.city_name,
	SUM(s.total) AS total_revenue,
	COUNT(DISTINCT s.customer_id) AS total_cx,
	ROUND(SUM(s.total)::NUMERIC / COUNT(DISTINCT s.customer_id)::NUMERIC, 2) AS avg_sale_per_cx
FROM sales s
JOIN customers c ON s.customer_id = c.customer_id
JOIN city ci ON ci.city_id = c.city_id
GROUP BY 1
ORDER BY 2 DESC;
```

### 6️⃣ Coffee Consumers vs. Customers  
```sql
WITH city_table AS (
	SELECT 
		city_name,
		ROUND((population * 0.25)/1000000, 2) AS coffee_consumers
	FROM city
),
customers_table AS (
	SELECT 
		ci.city_name,
		COUNT(DISTINCT c.customer_id) AS unique_cx
	FROM sales s
	JOIN customers c ON c.customer_id = s.customer_id
	JOIN city ci ON ci.city_id = c.city_id
	GROUP BY 1
)
SELECT 
	ct.city_name,
	ct.coffee_consumers AS coffee_consumers_in_millions,
	c.unique_cx
FROM city_table ct
JOIN customers_table c ON ct.city_name = c.city_name;
```

### 7️⃣ Top 3 Selling Products per City  
```sql
SELECT *
FROM (
	SELECT 
		ci.city_name,
		p.product_name,
		COUNT(s.sale_id) AS total_orders,
		DENSE_RANK() OVER(PARTITION BY ci.city_name ORDER BY COUNT(s.sale_id) DESC) AS rank
	FROM sales s
	JOIN products p ON s.product_id = p.product_id
	JOIN customers c ON c.customer_id = s.customer_id
	JOIN city ci ON ci.city_id = c.city_id
	GROUP BY 1, 2
) t1
WHERE rank <= 3;
```

### 8️⃣ Unique Customers per City  
```sql
SELECT 
	ci.city_name,
	COUNT(DISTINCT c.customer_id) AS unique_cx
FROM city ci
JOIN customers c ON c.city_id = ci.city_id
JOIN sales s ON s.customer_id = c.customer_id
GROUP BY 1;
```

---

## 💡 Key Insights
- 25% of each city’s population is estimated to be coffee consumers.  
- Q4 2023 was analyzed for total and city-wise revenue.  
- Average sales per customer reveal spending potential.  
- Top-selling products and cities help in strategy and marketing.  

---

## 🚀 Skills Demonstrated
- SQL Schema Design (DDL)
- Aggregations & Joins
- Common Table Expressions (CTEs)
- Window Functions (`DENSE_RANK`)
- Real-world Business Analysis Queries

---

## 📈 Future Enhancements
- Add monthly and yearly trend analysis.  
- Integrate SQL output with **Tableau** or **Power BI** dashboards.  
- Include profitability and price sensitivity studies.  
