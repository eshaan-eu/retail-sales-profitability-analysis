# 📊 Retail Sales & Profitability Analysis

![Power BI Dashboard](screenshot/dashboard.png)

An end-to-end retail analytics project built using **Google Cloud Storage, BigQuery, SQL, dimensional modeling, Power BI, and DAX**. The project transforms four raw CSV datasets into a validated **star schema** and an interactive dashboard for analyzing sales, profitability, customers, products, regions, and returns.

---

## 🧾 Executive Summary (For Hiring Managers)

- ✅ **Project scope:** Built an end-to-end retail analytics pipeline from raw CSV files through cloud storage, BigQuery transformation, dimensional modeling, and Power BI visualization.
- ✅ **Data engineering:** Created a **raw → staging → warehouse** workflow and preserved the raw source layer for traceability.
- ✅ **Data quality:** Identified and handled duplicate records, inconsistent product categories, invalid dates, missing customer/product references, and return transactions.
- ✅ **Data modeling:** Designed a **star schema** with `fact_orders` and four dimensions using surrogate keys.
- ✅ **Analytics:** Created reusable DAX measures for sales, profit, orders, customers, returns, average order value, and profit margin.
- ✅ **Outcome:** Delivered a business dashboard covering overall performance, sales trends, category performance, regional contribution, and profitability.

### Key project numbers

| Metric | Result |
|---|---:|
| Unique orders | **25,000** |
| Customers | **2,000** |
| Products | **100** |
| Regions | **20** |
| Exact duplicate order rows identified | **50** |
| Exact duplicate customer rows identified | **10** |
| Missing customer references | **60** |
| Missing product references | **35** |
| Invalid order dates | **10** |
| Return transactions | **15** |

---

## 🧩 Problem & Context

A retail business has transaction-level data covering **orders, customers, products, and geographical regions**. Before analysis, the source data contained several issues that could distort business KPIs and dashboard results.

The objective was to build a trustworthy analytical layer that would allow the business to understand:

- 🎯 **Sales performance:** How much is being sold and how sales change over time.
- 💰 **Profitability:** Which product categories generate the strongest profit.
- 🌍 **Regional performance:** Which regions contribute the most sales.
- 👥 **Customer activity:** How many customers and orders are represented.
- ↩️ **Returns:** How negative-quantity transactions should be interpreted and preserved.
- 🧹 **Data quality:** How duplicates, invalid dates, inconsistent categories, and missing relationships can be handled without unnecessarily losing valid transactions.

---

## 🎯 Project Objectives

1. Profile the raw data before transformation.
2. Identify data-quality issues and define appropriate business rules.
3. Build cleaned staging tables in BigQuery.
4. Validate referential integrity between orders and dimensions.
5. Design an analytical star schema.
6. Create reusable DAX measures in Power BI.
7. Build a decision-focused dashboard.
8. Extract business insights from sales and profitability data.

---

## ☁️ Data Pipeline & Architecture

```text
Raw CSV Files
     ↓
Google Cloud Storage
     ↓
BigQuery Raw Layer
     ↓
BigQuery Staging / Cleaning
     ↓
BigQuery Gold / Star Schema
     ↓
Power BI + DAX
     ↓
Dashboard & Business Insights
```

The project uses a layered approach so that the **raw source remains preserved**, cleaning happens in staging, and Power BI consumes the analytical warehouse layer.

---

## 🧱 Data Model

![Star Schema](images/Data_Warehouse.png)

The final warehouse model is centered around `fact_orders` with four dimensions:

- **`fact_orders`** – transactional measures such as quantity, discount, sales, cost, and profit.
- **`dim_customer`** – customer attributes such as name, gender, age, and segment.
- **`dim_product`** – product attributes such as product name, category, sub-category, cost, and selling price.
- **`dim_region`** – geographic attributes including state, city, and region.
- **`dim_date`** – calendar attributes used for time-based analysis.

### Relationships

```text
DIM_CUSTOMER  1 ───────── * FACT_ORDERS
DIM_PRODUCT   1 ───────── * FACT_ORDERS
DIM_REGION    1 ───────── * FACT_ORDERS
DIM_DATE      1 ───────── * FACT_ORDERS
```

The modeling principle is:

> **FACT = what happened**  
> **DIMENSIONS = who, what, where, and when**

---

## 🔍 Exploratory Data Analysis & Data Quality

Before creating the warehouse layer, the raw datasets were profiled for **row counts, uniqueness, missing values, invalid values, numeric ranges, formatting inconsistencies, and referential integrity**.

### Key issues identified

**Customers**

- 2,010 raw rows compared with 2,000 unique customer IDs.
- 10 exact duplicate customer rows were identified.

**Orders**

- 25,050 raw rows compared with 25,000 unique order IDs.
- 50 exact duplicate order rows were identified.
- 60 orders had a missing customer reference.
- 35 orders had a missing product reference.
- 10 records contained an invalid order date value.
- 15 records had negative quantities and matching negative sales, indicating return-like transactions.
- 586 transactions had negative profit; these were preserved because a negative profit can be a legitimate business outcome.
- No orphan customer, product, or region IDs were found among the non-null references.

**Products**

- Category values contained inconsistent capitalization and whitespace.
- Product categories were standardized before warehouse loading.

**Regions**

- The source file was initially ingested with generic `string_field_*` column names.
- The CSV header was incorrectly loaded as a data row and was corrected during staging.

---

## 🧹 Data Cleaning & Transformation

The cleaning layer was designed to preserve the raw data and create clean analytical tables separately.

### Customers

- Removed exact duplicate records.
- Trimmed text fields.
- Converted signup dates into proper date values.

### Products

- Removed exact duplicate records.
- Trimmed text fields.
- Standardized category and sub-category capitalization.

### Regions

- Corrected the incorrectly ingested header row.
- Renamed the generic source fields to meaningful business columns.
- Standardized text fields.

### Orders

- Removed exact duplicate records.
- Converted datetime strings into BigQuery `DATETIME` values.
- Preserved invalid-date transactions by representing their parsed date as `NULL`.
- Classified negative quantities as `RETURN` transactions.
- Preserved source financial values rather than deleting suspicious-looking rows.

---

## 🔐 Handling Missing Dimension References

A key modeling decision involved the **60 orders without a customer ID** and **35 orders without a product ID**.

Instead of deleting those orders, the warehouse contains explicit **Unknown** dimension members with key `0`.

This means:

```text
Missing customer → customer_key = 0
Missing product  → product_key  = 0
```

The transaction remains available for analysis while the missing descriptive information is made explicit.

The same principle can be applied to other dimensions when necessary.

---

## 📐 Surrogate Keys

The warehouse dimensions use separate surrogate keys:

- `customer_key`
- `product_key`
- `region_key`
- `date_key`

The original source identifiers such as `customer_id` and `product_id` remain available as business keys.

This keeps the **source identity** separate from the **warehouse relationship key** and provides a clean foundation for dimensional modeling.

---

## 💻 SQL Analysis & Transformation

All BigQuery SQL used in the project is consolidated into a single file:

### [`SQL_Queries.sql`](./SQL_Queries.sql)

The file is organized linearly into these sections:

1. **Data Profiling** – row counts, duplicates, NULL checks, invalid dates, numeric checks, and category inspection.
2. **Data Cleaning / Silver Staging** – creation of the four staging tables and standardization rules.
3. **Staging Validation** – duplicate validation, return checks, and foreign-key/orphan checks.
4. **Gold / Star Schema** – creation of dimensions, date dimension, unknown members, and the fact table.
5. **Final Validation** – fact-table integrity, unknown-key checks, uniqueness checks, and dimension counts.

This keeps the SQL workflow easy to reproduce while documenting the complete transformation process in one place.

---

## 📊 Power BI & DAX

The Gold-layer tables were connected to Power BI using the star-schema relationships.

The core DAX measures used in the dashboard include:

- **Total Sales**
- **Total Profit**
- **Total Orders**
- **Total Customers**

Additional analytical measures were prepared for:

- Total Quantity
- Total Cost
- Return Orders
- Return Rate
- Average Order Value
- Profit Margin

The complete DAX definitions are stored here:

### [`measures.dax`](./dax/measures.dax)

---

## 📈 Power BI Dashboard

![Power BI Dashboard](images/PowerBI_Dashboard_User.png)

The dashboard was designed as an executive-style overview rather than a collection of unrelated charts.

### KPI Cards

- Total Sales
- Total Profit
- Total Orders
- Total Customers

### Visual Analysis

- **Sales trend over time** – tracks movement in sales across the available date range.
- **Sales by category** – compares revenue contribution by product category.
- **Sales by region** – shows regional contribution to total sales.
- **Profit by category** – compares profitability across categories.

The dashboard layout is intentionally focused on a small number of high-value visuals to keep the report readable and useful for decision-making.

---

## 💡 Key Business Insights

### Sales Performance

The final model produced approximately **29.50M in net sales** across **25,000 unique orders**.

### Profitability

Approximately **6.76M in profit** was recorded across the transaction set. Category-level analysis helps distinguish high-revenue categories from categories that contribute strongly to profitability.

### Regional Performance

Regional analysis shows noticeable differences in sales contribution across the five regions, allowing management to identify stronger and weaker geographic markets.

### Returns

The model preserved **15 return transactions** instead of deleting them. This keeps the analytical dataset closer to the underlying business activity and makes future return-rate analysis possible.

### Data Quality

The project demonstrates that data quality is part of analytics itself: the final KPIs were only produced after resolving duplicates, cleaning categorical values, handling invalid dates, validating relationships, and explicitly modeling missing dimension references.

---

## 🧰 Tech Stack

- ☁️ **Google Cloud Storage** – source data storage
- 🐘 **Google BigQuery** – cloud data warehouse and SQL transformation engine
- 🧮 **SQL** – profiling, cleaning, validation, and dimensional modeling
- 🧱 **Star Schema** – analytical warehouse design
- 📊 **Microsoft Power BI** – dashboard and visualization
- 📐 **DAX** – analytical measures
- 🔧 **Git / GitHub** – version control and portfolio delivery

---

## 📂 Repository Structure

```text
retail-sales-profitability-analysis/
│
├── README.md
├── sql/
│  └──SQL_Queries.sql
│
├── dax/
│   └── measures.dax
│
├── images/
│   ├── Data_Warehouse.png
│   └── PowerBI_Dashboard_User.png
│
├── powerbi/
│   └── retail_sales_dashboard.pbix
│
└── screenshots/
    ├── dashboard.png
    └── dashboard_raw.jpg
```

---

## ✅ Final Validation

The final warehouse was validated to confirm:

- 25,000 fact rows
- 25,000 unique orders
- No NULL customer keys after Unknown-member handling
- No NULL product keys after Unknown-member handling
- No NULL region keys
- 10 transactions with NULL date keys due to invalid source dates
- 60 transactions mapped to the Unknown Customer member
- 35 transactions mapped to the Unknown Product member
- No duplicate order IDs in the final fact table

---

## 🚀 Future Enhancements

- Add monthly and year-over-year growth measures.
- Add customer segmentation and RFM analysis.
- Add product-level return-rate analysis.
- Add a dedicated returns dashboard page.
- Add automated scheduled BigQuery transformations.
- Connect the pipeline directly to Cloud Storage for repeatable ingestion.
- Introduce data-quality tests as part of a production-style ELT workflow.

---

## 👤 Project Focus

This project demonstrates an end-to-end workflow for turning raw operational data into a business-ready analytical solution using **cloud storage, SQL, data warehousing, dimensional modeling, DAX, and Power BI**.
