# Coffee-Sales-Analysis

## 📊 Project Overview
This project delivers an end-to-end relational data analytics pipeline for a rapidly growing specialty coffee retail chain in India. Parsing over **10,000 transaction records** across **14 distinct urban markets**, the objective of this analysis is to evaluate localized market penetration, track peak product popularity metrics, and evaluate fixed overhead constraints (commercial real estate rent) to discover optimal growth regions.

---

## 🛠️ Tech Stack & Skills Highlighted
* **Database Engine:** MySQL Workbench (Relational Schema Design & Query Optimization)
* **Business Intelligence:** Power BI Desktop (Star Schema Modeling & Executive Dashboards)
* **SQL Mechanics Used:** Common Table Expressions (CTEs), Window Functions (`DENSE_RANK`), Advanced Inter-table Joins, Multi-variable Grouping.

---

## 🗄️ Relational Database Schema (Star Schema Architecture)
The data infrastructure leverages a normalized star schema approach to eliminate redundancy and maximize query calculation speeds. 

* **Fact Table:** `sales` (stores transactional line-items: dates, revenue total, consumer ratings)
* **Dimension Tables:** * `city` (regional tracking matrices: population scale, estimated lease/rent, city hierarchy ranks)
  * `products` (product master inventory lookup table: item designations, base pricing)
  * `customers` (demographic client registry table)

---

## 💻 Key SQL Business Features Solved
Below are highlights of the advanced analytical functions deployed within the production script:

### 1. Localized Menu Preferences (Top 3 Products per City)
Utilizes the `DENSE_RANK()` window function partitioned by geographic region to isolate distinct product affinities across diverse Indian urban hubs.
```sql
SELECT * FROM (
    SELECT 
        t.city_name, 
        p.product_name, 
        COUNT(s.sale_id) AS orders,
        DENSE_RANK() OVER(PARTITION BY t.city_name ORDER BY COUNT(s.sale_id) DESC) AS ranking 
    FROM sales AS s
    JOIN products AS p ON s.product_id = p.product_id
    JOIN customers AS c ON c.customer_id = s.customer_id
    JOIN city AS t ON c.city_id = t.city_id 
    GROUP BY t.city_name, p.product_name
) AS ranked_menu_table
WHERE ranking <= 3;
2. Operational Overhead Analysis (Rent vs. Average Customer Spend)
Combines localized real estate lease expenses within a dual-CTE matrix to evaluate individual consumer yield profiles against fixed operational overhead costs.

WITH city_revenue_matrix AS (
    SELECT 
        ct.city_name, 
        SUM(s.total) AS revenue,
        COUNT(DISTINCT s.customer_id) AS total_cust,
        ROUND((SUM(s.total) / COUNT(DISTINCT s.customer_id)), 2) AS avg_sales_per_customer
    FROM sales AS s 
    JOIN customers AS c ON c.customer_id = s.customer_id
    JOIN city AS ct ON c.city_id = ct.city_id
    GROUP BY ct.city_name
),
city_overhead_matrix AS (
    SELECT city_name, estimated_rent FROM city
)
SELECT 
    com.city_name,
    com.estimated_rent,
    crm.total_cust,
    crm.avg_sales_per_customer,
    ROUND((com.estimated_rent / crm.total_cust), 2) AS avg_rent_overhead_per_customer
FROM city_overhead_matrix AS com
JOIN city_revenue_matrix AS crm ON com.city_name = crm.city_name
ORDER BY avg_sales_per_customer DESC;
