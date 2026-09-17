# 🛒 Customer Shopping Behavior & Revenue Trends Analysis
### *An End-to-End Data Analytics Portfolio Project using Python, MySQL & Power BI*

![Python](https://img.shields.io/badge/Python-3.13-3776AB?style=for-the-badge&logo=python&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Power BI](https://img.shields.io/badge/Power_BI-Desktop-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![Pandas](https://img.shields.io/badge/Pandas-Data_Analysis-150458?style=for-the-badge&logo=pandas&logoColor=white)
![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-ORM-D71F00?style=for-the-badge&logo=sqlalchemy&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)

---

## 📌 Executive Summary

In modern retail and e-commerce ecosystems, understanding customer purchase drivers, subscription behavior, discount sensitivity, and demographic spending patterns is critical for revenue growth and customer retention. 

In this project, I developed an **end-to-end data analytics pipeline** to analyze 3,900+ retail customer transactions. Using **Python** for data cleaning and feature engineering, **MySQL** for relational modeling and deep-dive analytical querying, and **Power BI** for executive visual intelligence, I transformed raw transactional data into actionable business recommendations.

```
┌─────────────────────────┐     ┌────────────────────────┐     ┌─────────────────────────┐     ┌─────────────────────────┐
│       Raw Data          │ ──> │     Python (EDA &      │ ──> │      MySQL Server       │ ──> │     Power BI & DAX      │
│ (3,900 Customer Records)│     │  Feature Engineering)  │     │   (Advanced Analytics)  │     │  (Executive Dashboard)  │
└─────────────────────────┘     └────────────────────────┘     └─────────────────────────┘     └─────────────────────────┘
```

---

## 🎯 Business Problem & Objectives

Retail businesses often face challenges with:
1. **Inefficient Promotional Spend:** Over-discounting products without understanding customer elasticity.
2. **Suboptimal Subscription Conversions:** Low conversion rates of repeat buyers into recurring subscribers.
3. **Broad Demographics vs. Targeted Marketing:** Lack of granular visibility into which age groups and product categories drive the highest Lifetime Value (LTV).

### Key Project Objectives:
- 🧹 **Clean & Transform:** Ingest and preprocess raw customer data, imputing missing values and engineering key analytical features.
- 🗄️ **Relational Storage & ETL:** Establish an automated Python-to-MySQL pipeline using SQLAlchemy.
- 🔍 **SQL Deep Dive:** Formulate complex queries with Window Functions, CTEs, and aggregations to answer high-impact business questions.
- 📊 **Interactive Dashboarding:** Design a responsive, multi-page Power BI dashboard featuring dynamic DAX KPIs and interactive slicers.
- 💡 **Strategic Recommendations:** Deliver concrete business strategies to increase subscription rates, optimize promotional discounts, and maximize customer retention.

---

## 🛠️ Tech Stack & Architecture

| Stage | Tool / Technology | Purpose |
|---|---|---|
| **Data Cleaning & EDA** | Python (`pandas`, `numpy`, `urllib.parse`) | Missing value imputation, column normalization, binning, feature engineering |
| **ETL & Database** | `SQLAlchemy`, `PyMySQL`, MySQL Server | Automated table ingestion, relational data modeling, schema definition |
| **Data Analytics** | MySQL (Advanced SQL) | Window functions (`ROW_NUMBER()`), CTEs, conditional logic (`CASE WHEN`), Subqueries |
| **Business Intelligence** | Microsoft Power BI | Star schema modeling, DAX measures, KPI cards, visual dashboards |
| **Version Control** | Git & GitHub | Project tracking, documentation, code sharing |

---

## 🔬 Step-by-Step Implementation ("How I Built It")

### Step 1: Exploratory Data Analysis & ETL Pipeline (Python)
*File: [`Customer_Shopping_Behavior_Analysis.ipynb`](./Customer_Shopping_Behavior_Analysis.ipynb)*

1. **Data Ingestion & Quality Inspection:**
   - Ingested 3,900 customer records spanning 18 features (demographics, item purchases, shipping type, review ratings, subscription status, discounts, etc.).
   - Identified 37 missing records in the `Review Rating` column.
2. **Category-Specific Imputation:**
   - Imputed missing ratings using category-level median ratings rather than global mean to preserve item-level variance:
     ```python
     df['Review Rating'] = df.groupby('Category')['Review Rating'].transform(lambda x: x.fillna(x.median()))
     ```
3. **Standardization & Feature Engineering:**
   - Converted all column headers to `snake_case` for clean database mapping.
   - Binned customer ages into 4 distinct groups (`Young Adult`, `Adult`, `Middle-aged`, `Senior`) using `pd.qcut()`.
   - Mapped textual purchase frequencies to numerical intervals (`purchase_frequency_days`: *Weekly -> 7, Fortnightly -> 14, Monthly -> 30, etc.*).
   - Checked collinearity and dropped duplicate/redundant attributes (`promo_code_used` was 100% redundant with `discount_applied`).
4. **Automated Database Ingestion:**
   - Established a secure connection to MySQL using SQLAlchemy and PyMySQL, loading the transformed DataFrame into the `customer` table.

---

### Step 2: In-Depth SQL Business Queries & Analytics
*File: [`customer_behavior_sql_queries.sql`](./customer_behavior_sql_queries.sql)*

I authored 10 complex SQL queries targeting core business questions. Below are notable highlights:

#### 1. Customer Loyalty & Retention Segmentation (CTE & Conditional Logic)
Categorized customer lifecycle stages into **New** (1 order), **Returning** (2–10 orders), and **Loyal** (>10 orders):
```sql
WITH customer_type AS (
    SELECT 
        customer_id, 
        previous_purchases,
        CASE 
            WHEN previous_purchases = 1 THEN 'New'
            WHEN previous_purchases BETWEEN 2 AND 10 THEN 'Returning'
            ELSE 'Loyal'
        END AS customer_segment
    FROM customer
)
SELECT 
    customer_segment,
    COUNT(*) AS total_customers
FROM customer_type 
GROUP BY customer_segment;
```

#### 2. Top 3 Products per Category (Window Functions)
Leveraged `ROW_NUMBER() OVER (PARTITION BY ...)` to determine top revenue items within each product category:
```sql
WITH item_counts AS (
    SELECT 
        category,
        item_purchased,
        COUNT(customer_id) AS total_orders,
        ROW_NUMBER() OVER (PARTITION BY category ORDER BY COUNT(customer_id) DESC) AS item_rank
    FROM customer
    GROUP BY category, item_purchased
)
SELECT item_rank, category, item_purchased, total_orders
FROM item_counts
WHERE item_rank <= 3;
```

#### 3. Subscription vs. Non-Subscription Revenue Comparison
Evaluated customer lifetime value (LTV) and average spend between subscribers and non-subscribers:
```sql
SELECT 
    subscription_status,
    COUNT(customer_id) AS total_customers,
    ROUND(AVG(purchase_amount), 2) AS avg_spend,
    ROUND(SUM(purchase_amount), 2) AS total_revenue
FROM customer
GROUP BY subscription_status
ORDER BY total_revenue DESC;
```

#### 4. Margin Risk & Product Discount Dependency
Identified top products that are overly reliant on discounts:
```sql
SELECT 
    item_purchased,
    ROUND(100.0 * SUM(CASE WHEN discount_applied = 'Yes' THEN 1 ELSE 0 END) / COUNT(*), 2) AS discount_rate
FROM customer
GROUP BY item_purchased
ORDER BY discount_rate DESC
LIMIT 5;
```

---

### Step 3: Power BI Dashboard & Visual Intelligence
*File: [`customer_behavior_dashboard.pbix`](./customer_behavior_dashboard.pbix)*

I designed an interactive, modern Power BI dashboard to present clear, executive-level insights:

1. **Custom DAX Measures & KPIs:**
   - **Total Revenue:** `SUM(customer[purchase_amount])`
   - **Total Customers:** `DISTINCTCOUNT(customer[customer_id])`
   - **Average Transaction Value (AOV):** `AVERAGE(customer[purchase_amount])`
   - **Average Review Rating:** `AVERAGE(customer[review_rating])`
   - **Discount Penetration Rate:** Dynamic DAX measuring % of transactions with promotions.
2. **Dashboard Views & Visualizations:**
   - **Executive KPI Cards:** Instant high-level summary of total sales, average order value, customer count, and rating.
   - **Demographic Breakdown:** Revenue and order distribution across Age Groups and Genders.
   - **Category & Product Matrix:** Tree maps and bar charts displaying top product performance across seasons.
   - **Behavioral Filters / Slicers:** Interactive filtering by Subscription Status, Shipping Type, Season, and Payment Method.

---

## 📈 Key Findings & Insights

| Dimension | Finding | Business Implication |
|---|---|---|
| **Demographics** | Middle-aged and Adult segments represent the highest total revenue contribution. | Marketing campaigns should prioritize 30–55 age brackets with customized collections. |
| **Subscription** | Non-subscribers generate more total revenue due to larger volume, but repeat buyers have a low subscription opt-in rate. | Large untapped pool of repeat purchasers who can be converted to subscribers. |
| **Discount Elasticity** | Over 50%+ of purchases for specific items (e.g., Hats, Gloves) only occur with discounts. | Risk of margin erosion; promotional strategies must transition to product bundling. |
| **Shipping Preferences** | Express Shipping customers exhibit high average transaction values comparable to standard shipping. | Expedited shipping thresholds can be leveraged to incentivize higher basket sizes. |
| **Customer Segments** | Loyal customers (>10 purchases) constitute a major pillar of repeat sales. | Implementing a tiered VIP rewards program can reduce churn and expand lifetime value. |

---

## 💡 Strategic Business Recommendations

1. **Convert Repeat Buyers into Subscribers:**
   - Implement an automated post-purchase prompt offering a first-month subscription discount to customers after their 3rd purchase.
2. **Dynamic Discount & Margin Protection:**
   - Phase out heavy single-item discounts on high-demand clothing items; introduce *"Buy 2, Get 1 at 20% off"* bundles to boost Average Order Value (AOV).
3. **Optimized Shipping Thresholds:**
   - Provide free Express Shipping on orders above $80 to encourage customers in the $50–$60 average spend bracket to add another item.
4. **Targeted Seasonal Campaigns:**
   - Align inventory and personalized marketing with seasonal peaks (e.g., Spring/Winter collection launches targeted by geographical region).

---

## 📂 Repository Structure

```plaintext
├── Business Problem Document.pdf          # Detailed business problem statement & requirements
├── Customer Shopping Behavior Analysis.pdf # Executive slide deck of project findings
├── Customer-Shopping-Behavior-Analysis.pptx # Editable presentation deck for stakeholders
├── Customer_Shopping_Behavior_Analysis.ipynb# Python Jupyter notebook (EDA, Cleaning, MySQL ETL)
├── customer_behavior_dashboard.pbix      # Interactive Microsoft Power BI Dashboard file
├── customer_behavior_sql_queries.sql     # Production-ready SQL queries for business questions
├── customer_shopping_behavior.csv        # Raw dataset (3,900 customer records)
└── README.md                             # Comprehensive project documentation
```

---

## 🚀 How to Reproduce This Project

### 1. Prerequisites
- Python 3.10+ with `pandas`, `numpy`, `sqlalchemy`, and `pymysql` installed.
- MySQL Server (or PostgreSQL/MS SQL Server).
- Microsoft Power BI Desktop.

### 2. Setup & Execution
1. **Clone the repository:**
   ```bash
   git clone https://github.com/kiki0329/SQL-Python-PowerBI.git
   cd SQL-Python-PowerBI
   ```
2. **Run the Python Notebook:**
   - Open and execute [`Customer_Shopping_Behavior_Analysis.ipynb`](./Customer_Shopping_Behavior_Analysis.ipynb).
   - Configure your MySQL database credentials in the database connection cell to load the cleaned data into your local MySQL instance.
3. **Execute SQL Queries:**
   - Run [`customer_behavior_sql_queries.sql`](./customer_behavior_sql_queries.sql) in MySQL Workbench or your favorite SQL editor.
4. **Explore the Power BI Dashboard:**
   - Open [`customer_behavior_dashboard.pbix`](./customer_behavior_dashboard.pbix) in Power BI Desktop to interact with the visualizations.

---

## 👤 Author
- **Project by:** Kiruthika M
- **Role:** Data Analyst / BI Developer
- **Focus:** Python | SQL | Power BI | Data Modeling & Business Analytics

---
*If you find this project insightful, feel free to star ⭐ this repository!*

