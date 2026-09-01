# Olist E-Commerce Customer Cohort & Retention Analysis

## 📌 Project Overview

This project analyzes the **Brazilian E-Commerce Public Dataset by Olist** to understand customer purchasing behavior and retention patterns.

The project uses **SQL within a Jupyter Notebook**, with the Olist CSV datasets loaded into an in-memory **SQLite database**. Advanced SQL techniques including **Common Table Expressions (CTEs), Window Functions, JOINs, conditional aggregation, and indexing** are used to clean the transactional data and build a monthly customer cohort retention matrix.

The analysis focuses on identifying when customers made their first purchase, tracking their activity in subsequent months, and measuring customer retention across different acquisition cohorts.

---

## 🎯 Project Objectives

The main objectives of this project are to:

* Clean and validate Olist transactional data using SQL.
* Identify and handle duplicate records.
* Analyze and handle missing values.
* Structure the datasets as relational SQL tables.
* Create indexes on frequently used columns and join keys.
* Use CTEs to create a multi-step analytical SQL pipeline.
* Apply Window Functions such as `ROW_NUMBER()` and `LAG()`.
* Identify each customer's first purchase month.
* Assign customers to monthly cohorts.
* Track customer activity after their first purchase.
* Calculate monthly customer retention rates.
* Build a customer cohort retention matrix.
* Visualize retention patterns using Python.
* Generate business insights from customer purchasing behavior.

---

## 📊 Dataset

The project uses the **Brazilian E-Commerce Public Dataset by Olist**, which contains approximately 100K orders from the Brazilian marketplace Olist.

### Main datasets

| Dataset                                 | Description                                        |
| --------------------------------------- | -------------------------------------------------- |
| `olist_customers_dataset.csv`           | Customer information                               |
| `olist_orders_dataset.csv`              | Order information and timestamps                   |
| `olist_order_items_dataset.csv`         | Products included in each order                    |
| `olist_order_payments_dataset.csv`      | Payment information                                |
| `olist_order_reviews_dataset.csv`       | Customer reviews                                   |
| `olist_products_dataset.csv`            | Product information                                |
| `olist_sellers_dataset.csv`             | Seller information                                 |
| `olist_geolocation_dataset.csv`         | Customer and seller geographic information         |
| `product_category_name_translation.csv` | Portuguese-to-English product category translation |

---

## 🛠️ Tools & Technologies

* **Python**
* **Jupyter Notebook**
* **SQL**
* **SQLite**
* **Pandas**
* **NumPy**
* **Matplotlib**

### SQL Concepts Used

* `SELECT`
* `WHERE`
* `GROUP BY`
* `HAVING`
* `JOIN`
* `INNER JOIN`
* `LEFT JOIN`
* `CASE WHEN`
* `COALESCE`
* Common Table Expressions (`WITH`)
* Window Functions
* `ROW_NUMBER()`
* `LAG()`
* `RANK()`
* `NTILE()`
* `PARTITION BY`
* Conditional Aggregation
* Indexing
* `EXPLAIN QUERY PLAN`

---

# 📁 Project Structure

```text
olist-ecommerce-cohort-analysis/
│
├── data/
│   ├── olist_customers_dataset.csv
│   ├── olist_orders_dataset.csv
│   ├── olist_order_items_dataset.csv
│   ├── olist_order_payments_dataset.csv
│   ├── olist_order_reviews_dataset.csv
│   ├── olist_products_dataset.csv
│   ├── olist_sellers_dataset.csv
│   ├── olist_geolocation_dataset.csv
│   └── product_category_name_translation.csv
│
├── Olist_Customer_Cohort_Analysis.ipynb
│
└── README.md
```

---

# 🔄 Project Workflow

```text
Olist CSV Dataset
       ↓
Load Data with Pandas
       ↓
Create SQLite Database
       ↓
Load DataFrames into SQL Tables
       ↓
Data Profiling
       ↓
Duplicate & NULL Analysis
       ↓
Data Cleaning
       ↓
Create SQL Indexes
       ↓
Customer & Order Analysis
       ↓
CTE-Based Cohort Analysis
       ↓
Window Function Analysis
       ↓
Monthly Cohort Assignment
       ↓
Retention Calculation
       ↓
Cohort Retention Matrix
       ↓
Visualization
       ↓
Business Insights
```

---

# 🔍 1. Data Loading

The Olist CSV files are loaded into Pandas DataFrames:

```python
import pandas as pd
import sqlite3
import matplotlib.pyplot as plt

customers = pd.read_csv("olist_customers_dataset.csv")
orders = pd.read_csv("olist_orders_dataset.csv")
order_items = pd.read_csv("olist_order_items_dataset.csv")
payments = pd.read_csv("olist_order_payments_dataset.csv")
reviews = pd.read_csv("olist_order_reviews_dataset.csv")
products = pd.read_csv("olist_products_dataset.csv")
sellers = pd.read_csv("olist_sellers_dataset.csv")
```

The DataFrames are then loaded into an SQLite database:

```python
conn = sqlite3.connect(":memory:")

customers.to_sql(
    "customers",
    conn,
    if_exists="replace",
    index=False
)

orders.to_sql(
    "orders",
    conn,
    if_exists="replace",
    index=False
)
```

---

# 🧹 2. Data Cleaning

The project performs SQL-based data quality checks before conducting the cohort analysis.

### Duplicate Detection

Duplicates are identified using aggregation:

```sql
SELECT
    customer_id,
    COUNT(*) AS duplicate_count
FROM customers
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

### Missing Value Analysis

Missing values are analyzed using conditional aggregation:

```sql
SELECT
    COUNT(*) AS total_rows,

    SUM(
        CASE
            WHEN customer_unique_id IS NULL
            THEN 1
            ELSE 0
        END
    ) AS missing_unique_id,

    SUM(
        CASE
            WHEN customer_city IS NULL
            THEN 1
            ELSE 0
        END
    ) AS missing_city

FROM customers;
```

Critical identifiers such as `customer_unique_id` are not artificially populated with fabricated values.

---

# 📅 3. Customer Cohort Definition

A customer's **cohort month** is defined as the month in which they made their first valid purchase.

For example:

```text
Customer A → First Purchase: January 2017 → January 2017 Cohort

Customer B → First Purchase: February 2017 → February 2017 Cohort

Customer C → First Purchase: February 2017 → February 2017 Cohort
```

The first purchase is calculated using a CTE:

```sql
WITH monthly_purchases AS (

    SELECT DISTINCT

        c.customer_unique_id,

        STRFTIME(
            '%Y-%m',
            o.order_purchase_timestamp
        ) AS purchase_month

    FROM customers c

    JOIN orders o
        ON c.customer_id = o.customer_id

    WHERE o.order_status NOT IN (
        'canceled',
        'unavailable'
    )

),

customer_cohorts AS (

    SELECT

        customer_unique_id,

        MIN(purchase_month) AS cohort_month

    FROM monthly_purchases

    GROUP BY customer_unique_id
)

SELECT *
FROM customer_cohorts;
```

---

# 📈 4. Cohort Activity

After assigning each customer to a cohort, their activity is tracked in subsequent months.

The analysis calculates:

```text
Cohort Month
     ↓
Purchase Month
     ↓
Months Since First Purchase
     ↓
Active Customers
     ↓
Retention Rate
```

The cohort index represents the number of months since the customer's first purchase.

For example:

| Cohort Month | Purchase Month | Cohort Index |
| ------------ | -------------- | -----------: |
| 2017-01      | 2017-01        |            0 |
| 2017-01      | 2017-02        |            1 |
| 2017-01      | 2017-03        |            2 |
| 2017-01      | 2017-04        |            3 |

---

# 🪟 5. Window Functions

Window Functions are used to analyze individual customer purchase behavior.

### ROW_NUMBER()

```sql
SELECT

    customer_unique_id,
    order_id,
    order_purchase_timestamp,

    ROW_NUMBER() OVER (
        PARTITION BY customer_unique_id
        ORDER BY order_purchase_timestamp
    ) AS purchase_number

FROM customer_orders;
```

This identifies the sequence of purchases made by each customer.

### LAG()

The `LAG()` function is used to identify the previous purchase:

```sql
SELECT

    customer_unique_id,
    order_id,
    order_purchase_timestamp,

    LAG(order_purchase_timestamp)
        OVER (
            PARTITION BY customer_unique_id
            ORDER BY order_purchase_timestamp
        ) AS previous_purchase

FROM customer_orders;
```

This can be used to analyze the time between repeat purchases.

---

# 📊 6. Retention Calculation

Retention is calculated as:

```text
Retention Rate =
Active Customers in Cohort Month N
----------------------------------- × 100
Total Customers in Original Cohort
```

For example:

```text
January Cohort
────────────────────────

Month 0 → 100%
Month 1 → 5.2%
Month 2 → 3.4%
Month 3 → 2.8%
```

The actual values are generated directly from the Olist dataset in the notebook.

---

# 🔥 7. Cohort Retention Matrix

The final retention data is transformed into a matrix using Pandas:

```python
retention_matrix = retention.pivot(
    index="cohort_month",
    columns="cohort_index",
    values="retention_rate"
)
```

The resulting structure is:

| Cohort   | Month 0 | Month 1 | Month 2 | Month 3 | Month 4 |
| -------- | ------: | ------: | ------: | ------: | ------: |
| Jan 2017 |    100% |       — |       — |       — |       — |
| Feb 2017 |    100% |       — |       — |       — |       — |
| Mar 2017 |    100% |       — |       — |       — |       — |
| Apr 2017 |    100% |       — |       — |       — |       — |

The values are populated automatically when the notebook is executed.

---

# 📉 8. Retention Visualization

The cohort matrix is visualized as a heatmap to make retention trends easier to identify.

```python
plt.figure(figsize=(14, 8))

plt.imshow(
    retention_matrix,
    aspect="auto"
)

plt.colorbar(
    label="Retention Rate (%)"
)

plt.xlabel("Months Since First Purchase")
plt.ylabel("Cohort Month")

plt.title(
    "Olist Customer Cohort Retention Matrix"
)

plt.show()
```

The visualization makes it easier to identify:

* Cohorts with stronger retention.
* Early customer drop-off.
* Long-term retention patterns.
* Differences between acquisition cohorts.
* Cohorts with unusually high or low repeat activity.

---

# ⚡ 9. SQL Optimization

Indexes are created on frequently queried and joined columns.

Examples include:

```sql
CREATE INDEX idx_customers_customer_id
ON customers(customer_id);
```

```sql
CREATE INDEX idx_customers_unique_id
ON customers(customer_unique_id);
```

```sql
CREATE INDEX idx_orders_customer_id
ON orders(customer_id);
```

```sql
CREATE INDEX idx_orders_purchase_timestamp
ON orders(order_purchase_timestamp);
```

```sql
CREATE INDEX idx_order_items_order_id
ON order_items(order_id);
```

Query execution plans can be inspected using:

```sql
EXPLAIN QUERY PLAN

SELECT
    c.customer_unique_id,
    o.order_purchase_timestamp

FROM customers c

JOIN orders o
    ON c.customer_id = o.customer_id

WHERE o.order_purchase_timestamp >= '2018-01-01';
```

This helps evaluate how SQLite executes joins and filtering operations.

---

# 💡 Business Questions

The analysis is designed to answer questions such as:

### Customer Retention

* How many customers make repeat purchases?
* What percentage of customers return after their first purchase?
* How does retention change after Month 1, Month 2, Month 3, etc.?
* Which customer cohorts have the strongest retention?

### Purchasing Behavior

* How frequently do customers make repeat purchases?
* What is the average time between purchases?
* Which customers make the most purchases?
* Which customers generate the highest revenue?

### Cohort Performance

* Which acquisition months produced the most loyal customers?
* Are newer cohorts retaining customers better or worse?
* At what point does the largest customer drop-off occur?
* Are there cohorts with unusually strong or weak retention?

---

# 📌 Key SQL Skills Demonstrated

This project demonstrates practical SQL skills including:

```text
Data Cleaning
      ↓
Duplicate Detection
      ↓
NULL Handling
      ↓
JOINs
      ↓
CTEs
      ↓
Window Functions
      ↓
Date Manipulation
      ↓
Aggregation
      ↓
Conditional Aggregation
      ↓
Indexing
      ↓
Query Optimization
      ↓
Cohort Analysis
```

---

# 🚀 Future Improvements

Potential extensions to this project include:

* Customer **RFM analysis**.
* Customer Lifetime Value (CLV).
* Revenue retention analysis.
* Product-category retention analysis.
* Geographic retention analysis.
* Customer segmentation.
* Repeat-purchase prediction.
* Churn prediction using machine learning.
* Interactive dashboard using Tableau or Power BI.
* Migrating the SQL workflow from SQLite to PostgreSQL.
* Comparing query performance before and after indexing.

---

# 📂 Files

### Jupyter Notebook

`Olist_Customer_Cohort_Analysis.ipynb`

Contains:

* Data loading
* Data exploration
* SQL database creation
* Data cleaning
* Duplicate analysis
* NULL analysis
* Index creation
* CTE queries
* Window Functions
* Cohort analysis
* Retention matrix
* Visualization
* Business insights

---

# 👩‍💻 Author

**Natasha Correa**

Data Analyst | SQL | Python | Data Science

This project was developed as a portfolio project to demonstrate practical SQL, data analysis, database optimization, and customer retention analytics skills.
