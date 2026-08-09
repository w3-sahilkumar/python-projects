# 🚲 Bike Store Sales & Business Intelligence — SQL Portfolio Project

> **A complete SQL-based sales analysis project covering database design, data quality validation, exploratory data analysis, and business performance insights.**

---

## 📌 Project Overview

This project analyzes sales data from a **multi-store bicycle retail business** using **MySQL**.

The objective was to build a structured relational database, validate the imported data, and use SQL to answer important business questions around:

- 💰 Revenue performance
- 📈 Sales trends
- 🛒 Orders and units sold
- 🚲 Product performance
- 🏷️ Brand performance
- 📦 Category performance
- 🏪 Store operations
- 📊 Average Order Value
- 🔍 Data quality and consistency

The project follows a practical analytics workflow:

**Database Creation → Data Import → Data Quality Analysis → EDA → Business Analysis**

---

## 🛠️ Tools & Technologies

| Tool / Technology | Purpose |
|---|---|
| **MySQL** | Database creation, querying & analysis |
| **MySQL Workbench** | SQL development & database management |
| **SQL** | Data validation, transformation & analysis |
| **GitHub** | Project documentation & version control |

### SQL Concepts Used

- `CREATE DATABASE`
- `CREATE TABLE`
- Primary Keys
- Composite Primary Keys
- Foreign Keys
- Self-referencing Foreign Keys
- `JOIN`
- `GROUP BY`
- `ORDER BY`
- Aggregate Functions
- `COUNT()`
- `SUM()`
- `ROUND()`
- `MIN()` / `MAX()`
- `CASE WHEN`
- `HAVING`
- `DISTINCT`
- Date Functions
- `YEAR()`
- `MONTH()`
- `MONTHNAME()`
- NULL analysis
- Duplicate analysis
- Data validation
- KPI calculations

---

# 🗂️ Database Schema

The project uses **9 relational tables**:

```text
brands
   │
   └── products
          │
          ├── categories
          │
          └── order_items
                  │
                  └── orders
                        ├── customers
                        ├── stores
                        └── staffs
                              │
                              └── staffs (manager relationship)

stores
   │
   └── stocks
          │
          └── products
```

### Tables

| Table | Description |
|---|---|
| `brands` | Bicycle brands |
| `categories` | Product categories |
| `customers` | Customer information |
| `stores` | Store information |
| `staffs` | Store employees and reporting structure |
| `products` | Product catalog |
| `orders` | Customer orders |
| `order_items` | Individual products within each order |
| `stocks` | Product inventory by store |

---

# 🧱 Database Design

The database was created using relational database principles with appropriate **primary keys and foreign-key relationships**.

### Example

```sql
CREATE TABLE products(
    product_id INT PRIMARY KEY,
    product_name VARCHAR(150) NOT NULL,
    brand_id INT NOT NULL,
    category_id INT NOT NULL,
    model_year YEAR,
    list_price DECIMAL(10,2) NOT NULL,

    FOREIGN KEY(brand_id)
        REFERENCES brands(brand_id),

    FOREIGN KEY(category_id)
        REFERENCES categories(category_id)
);
```

The `order_items` table uses a **composite primary key**:

```sql
PRIMARY KEY(order_id, item_id)
```

This ensures that each item within an order is uniquely identified.

---

# 🔍 Data Quality Analysis

Before performing business analysis, the dataset was validated for common data-quality issues.

## 1. Row Count Validation

Row counts were checked across all major tables to ensure the imported data was complete and consistent.

```sql
SELECT COUNT(*) AS total_orders
FROM orders;

SELECT COUNT(*) AS total_orderitems
FROM order_items;

SELECT COUNT(*) AS total_products
FROM products;
```

This step helped verify that the imported CSV data matched the expected database structure.

---

## 2. NULL Analysis

NULL values were investigated across important fields.

Example:

```sql
SELECT
    COUNT(*) AS total_rows,
    SUM(CASE WHEN customer_id IS NULL THEN 1 ELSE 0 END) AS null_id,
    SUM(CASE WHEN first_name IS NULL THEN 1 ELSE 0 END) AS null_f_name,
    SUM(CASE WHEN last_name IS NULL THEN 1 ELSE 0 END) AS null_l_name,
    SUM(CASE WHEN phone IS NULL THEN 1 ELSE 0 END) AS null_phone,
    SUM(CASE WHEN email IS NULL THEN 1 ELSE 0 END) AS null_email
FROM customers;
```

NULL percentages were also calculated to understand the extent of missing information.

---

## 3. Duplicate Analysis

Potential duplicate records were investigated using grouping and `HAVING`.

```sql
SELECT
    customer_id,
    COUNT(*) AS count
FROM customers
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

Additional checks were performed on customer names and other relevant fields.

---

## 4. Range & Validity Checks

Numerical fields were checked for invalid values.

### Negative prices

```sql
SELECT *
FROM products
WHERE list_price < 0;
```

### Zero prices

```sql
SELECT *
FROM products
WHERE list_price = 0;
```

### Invalid quantities

```sql
SELECT *
FROM order_items
WHERE quantity <= 0;
```

These checks helped ensure that the sales calculations were based on valid transactional data.

---

## 5. Date Validation

The date range of the dataset was identified:

```sql
SELECT
    MIN(order_date) AS first_order,
    MAX(order_date) AS last_order
FROM orders;
```

Potentially invalid shipment dates were also checked:

```sql
SELECT *
FROM orders
WHERE shipped_date IS NOT NULL
AND shipped_date < order_date;
```

---

# 📊 Exploratory Data Analysis

After validating the data, SQL was used to analyze the business performance of the bike store.

---

# 💰 Business Overview & KPIs

The analysis calculates core business metrics including:

- Total Customers
- Total Orders
- Total Order Items
- Total Products
- Total Brands
- Total Categories
- Total Stores
- Total Staff
- Total Units Sold
- Total Revenue
- Average Order Value
- Average Units per Order

### Revenue Calculation

Revenue was calculated using:

```text
Revenue =
Quantity × List Price × (1 − Discount)
```

SQL implementation:

```sql
SELECT
    SUM(quantity) AS total_units_sold,
    ROUND(
        SUM(quantity * list_price * (1 - discount)),
        2
    ) AS revenue,
    COUNT(DISTINCT order_id) AS total_orders,
    ROUND(
        SUM(quantity * list_price * (1 - discount))
        / COUNT(DISTINCT order_id),
        2
    ) AS average_order_value
FROM order_items;
```

---

# 📈 Sales Trend Analysis

Sales performance was analyzed across years and months.

### Yearly Revenue

```sql
SELECT
    YEAR(o.order_date) AS order_year,
    SUM(
        oi.quantity *
        oi.list_price *
        (1 - oi.discount)
    ) AS revenue
FROM orders o
JOIN order_items oi
    ON o.order_id = oi.order_id
GROUP BY YEAR(o.order_date)
ORDER BY order_year;
```

### Monthly Analysis

The monthly analysis tracks:

- Number of orders
- Units sold
- Revenue
- Average Order Value

This helps identify seasonal trends and changes in sales performance over time.

---

# 🚲 Product Performance Analysis

Products were evaluated using both **revenue** and **unit sales**.

## Top Products by Revenue

The analysis identifies products generating the highest sales value.

Metrics include:

- Product ID
- Product Name
- Units Sold
- Number of Orders
- Revenue

```sql
SELECT
    p.product_id,
    p.product_name,
    SUM(oi.quantity) AS units_sold,
    COUNT(DISTINCT oi.order_id) AS total_orders,
    ROUND(
        SUM(
            oi.quantity *
            oi.list_price *
            (1 - oi.discount)
        ), 2
    ) AS revenue
FROM order_items oi
JOIN products p
    ON oi.product_id = p.product_id
GROUP BY
    p.product_id,
    p.product_name
ORDER BY revenue DESC
LIMIT 10;
```

---

## 📦 Top Products by Units Sold

A separate analysis identifies the products with the highest volume of units sold.

This distinction is important because the product with the highest **sales volume** may not necessarily be the product generating the highest **revenue**.

---

# 🏷️ Category Performance

Revenue was analyzed across product categories.

```sql
SELECT
    c.category_id,
    c.category_name,
    SUM(oi.quantity) AS units_sold,
    COUNT(DISTINCT oi.order_id) AS total_orders,
    ROUND(
        SUM(
            oi.quantity *
            oi.list_price *
            (1 - oi.discount)
        ), 2
    ) AS revenue
FROM order_items oi
JOIN products p
    ON oi.product_id = p.product_id
JOIN categories c
    ON p.category_id = c.category_id
GROUP BY
    c.category_id,
    c.category_name
ORDER BY revenue DESC;
```

This analysis helps identify which product categories contribute the most to overall business revenue.

---

# 🏆 Brand Performance

Brand-level performance was also analyzed using sales revenue.

```sql
SELECT
    b.brand_id,
    b.brand_name,
    SUM(oi.quantity) AS units_sold,
    COUNT(DISTINCT oi.order_id) AS total_orders,
    ROUND(
        SUM(
            oi.quantity *
            oi.list_price *
            (1 - oi.discount)
        ), 2
    ) AS revenue
FROM order_items oi
JOIN products p
    ON oi.product_id = p.product_id
JOIN brands b
    ON p.brand_id = b.brand_id
GROUP BY
    b.brand_id,
    b.brand_name
ORDER BY revenue DESC;
```

This provides a clear view of the strongest-performing brands in the product portfolio.

---

# ❓ Business Questions Answered

The project was designed around practical business questions:

### Sales Performance

- What is the overall revenue?
- How many orders were placed?
- How many units were sold?
- Which year performed best?
- Which month performed best?
- Which month performed worst?
- What is the Average Order Value?
- What is the average number of units per order?

### Product Performance

- Which product is the revenue leader?
- Which products sell the most units?
- Which products generate the highest revenue?

### Category Performance

- Which product category dominates revenue?
- Which categories generate the highest sales volume?

### Brand Performance

- Which brand generates the highest revenue?
- Which brands sell the most units?

### Data Quality

- Are there missing values?
- Are there duplicate records?
- Are there invalid prices?
- Are there invalid quantities?
- Are there invalid dates?
- Are foreign-key relationships properly structured?

---

# 🧠 Key SQL Learning Outcomes

This project helped develop practical understanding of:

### Database Design

- Relational database structure
- Primary and foreign keys
- Composite keys
- Referential integrity
- Self-referencing relationships

### Data Quality

- NULL analysis
- Duplicate detection
- Range validation
- Date validation
- Row-count reconciliation

### Data Analysis

- Aggregations
- Multi-table joins
- KPI calculations
- Time-series analysis
- Product segmentation
- Category analysis
- Brand analysis

### Business Intelligence

The project demonstrates how raw transactional data can be transformed into meaningful business metrics that support decision-making.

---

# 📁 Suggested GitHub Repository Structure

```text
bike-store-sql-analysis/
│
├── README.md
│
├── sql/
│   ├── 01_database_setup.sql
│   ├── 02_data_quality_analysis.sql
│   └── 03_exploratory_data_analysis.sql
│
├── data/
│   └── README.md
│
└── screenshots/
    └── README.md
```

> **Note:** Large raw CSV files can be excluded from GitHub if they exceed repository limits. A data-source/reference file can be provided instead.

---

# 🚀 Future Improvements

The project can be extended beyond the current SQL analysis by adding:

- 📊 Power BI interactive dashboard
- 📈 Sales trend visualizations
- 🗺️ Store/location performance analysis
- 👥 Customer segmentation
- 📦 Inventory analysis
- 🏪 Store-level performance
- 👨‍💼 Staff performance analysis
- 📉 Discount impact analysis
- 🔄 Customer repeat-purchase analysis
- 🏆 Product ranking using window functions
- 📅 Year-over-year growth analysis
- 📈 Running totals and cumulative revenue
- 🎯 Pareto analysis of products and brands

---

# 📌 Project Status

**Status:** ✅ SQL Analysis Completed

**Current Stage:**

```text
Database Design       ✅
Data Import           ✅
Data Validation       ✅
NULL Analysis         ✅
Duplicate Analysis    ✅
Validity Checks       ✅
EDA                   ✅
Business KPIs         ✅
Product Analysis      ✅
Category Analysis     ✅
Brand Analysis        ✅
Advanced Analysis    🔜
Dashboard             🔜
```

---

## 👨‍💻 About the Project

This project was created as part of a **Data Analytics portfolio** to demonstrate practical SQL skills using a real-world retail sales scenario.

The focus was not only on writing SQL queries, but on following a complete analytical workflow:

> **Build → Validate → Analyze → Extract Insights → Communicate**

---

⭐ **If you find this project useful, feel free to star the repository.**