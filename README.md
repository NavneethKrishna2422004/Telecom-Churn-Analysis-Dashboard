# 📊 Telecom Customer Churn Analysis

A Power BI project analyzing customer churn behavior for a telecom company. The report identifies churn patterns, revenue impact, and key risk drivers — culminating in actionable retention recommendations.

**Author:** Navneeth Krishna  
**LinkedIn:** [navneethkrishna2004](https://www.linkedin.com/in/navneethkrishna2004)  
**Data Source:** [Telco Customer Churn — Kaggle](https://www.kaggle.com/datasets/blastchar/telco-customer-churn)

---

## 🔄 Project Pipeline

```
Kaggle CSV
    ↓
Jupyter (Python) — Split into 3 tables
    ↓
PostgreSQL — Create tables, import CSVs, clean & validate
    ↓
Power BI — Connect to PostgreSQL, build dashboard
```

---

## 📁 Project Structure

```
telecom-churn-analysis/
│
├── ChurnAnalysis.pbix                  # Power BI report file
├── Churn_Analysis_Report.pdf           # Exported dashboard (3 pages)
├── split_tables.ipynb                  # Jupyter notebook — table splitting
├── queries/
│   └── create_tables.sql               # PostgreSQL table creation
│   └── data_cleaning.sql               # Data cleaning & validation
│   └── exploratory_analysis.sql        # Exploratory SQL queries
├── screenshots/
│   ├── dashboard_overview.png
│   ├── dashboard_churn_drivers.png
│   └── dashboard_retention.png
└── README.md
```

---

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| Python (Jupyter) | Splitting the raw CSV into 3 separate tables |
| PostgreSQL | Table creation, data import, cleaning & validation |
| Power BI Desktop | Data modeling, DAX measures, and dashboard creation |
| DAX | Calculated measures and columns |

---

## ⚙️ How to Run

### 1. Download the Dataset
Download `WA_Fn-UseC_-Telco-Customer-Churn.csv` from [Kaggle](https://www.kaggle.com/datasets/blastchar/telco-customer-churn).

### 2. Split into 3 Tables (Jupyter)
Run `split_tables.ipynb` to generate `customers.csv`, `services.csv`, and `billing.csv`.

### 3. Set Up PostgreSQL
- Create a database in PostgreSQL
- Run `create_tables.sql` to create the 3 tables
- Import the 3 CSVs into their respective tables
- Run `data_cleaning.sql` to clean and validate the data

### 4. Open in Power BI
- Open `ChurnAnalysis.pbix` in Power BI Desktop
- Update the data source connection to your PostgreSQL instance if needed (`Home → Transform Data → Data Source Settings`)
- Refresh and the visuals will populate automatically

---

## 🐍 Python — Table Splitting

The original Kaggle dataset is a single CSV. This script splits it into 3 normalized tables:

```python
import pandas as pd

df = pd.read_csv("WA_Fn-UseC_-Telco-Customer-Churn.csv")

# Table 1 - Customers
customers = df[["customerID", "gender", "SeniorCitizen", 
                "Partner", "Dependents", "tenure", "Churn"]]

# Table 2 - Services
services = df[["customerID", "PhoneService", "MultipleLines",
               "InternetService", "OnlineSecurity", "OnlineBackup",
               "DeviceProtection", "TechSupport", "StreamingTV", 
               "StreamingMovies"]]

# Table 3 - Billing
billing = df[["customerID", "Contract", "PaperlessBilling",
              "PaymentMethod", "MonthlyCharges", "TotalCharges"]]

# Save as separate CSVs
customers.to_csv("customers.csv", index=False)
services.to_csv("services.csv", index=False)
billing.to_csv("billing.csv", index=False)
```

---

## 🗄️ PostgreSQL — Table Creation

```sql
CREATE TABLE customers (
  CustomerID    VARCHAR,
  gender        VARCHAR,
  SeniorCitizen INT,
  Partner       VARCHAR,
  Dependents    VARCHAR,
  tenure        INT,
  Churn         VARCHAR
);

CREATE TABLE services (
  CustomerID        VARCHAR,
  PhoneService      VARCHAR,
  MultipleLines     VARCHAR,
  InternetService   VARCHAR,
  OnlineSecurity    VARCHAR,
  OnlineBackup      VARCHAR,
  DeviceProtection  VARCHAR,
  TechSupport       VARCHAR,
  StreamingTV       VARCHAR,
  StreamingMovies   VARCHAR
);

CREATE TABLE billing (
  CustomerID       VARCHAR,
  Contract         VARCHAR,
  PaperlessBilling VARCHAR,
  PaymentMethod    VARCHAR,
  MonthlyCharges   FLOAT,
  TotalCharges     VARCHAR
);
```

---

## 🧹 PostgreSQL — Data Cleaning & Validation

### Check for Nulls
```sql
SELECT 
  COUNT(*) - COUNT(customerid)      AS null_customerid,
  COUNT(*) - COUNT(tenure)          AS null_tenure,
  COUNT(*) - COUNT(monthlycharges)  AS null_monthlycharges
FROM customers;
```

### Check for Duplicate Customers
```sql
SELECT customerid, COUNT(*) 
FROM customers
GROUP BY customerid
HAVING COUNT(*) > 1;
```

### Verify Row Counts Across Tables
```sql
SELECT
  (SELECT COUNT(*) FROM customers) AS customer_count,
  (SELECT COUNT(*) FROM services)  AS services_count,
  (SELECT COUNT(*) FROM billing)   AS billing_count;
```

### Check Distinct Churn Values
```sql
SELECT DISTINCT churn FROM customers;
```

### Fix Blank TotalCharges (Whitespace → NULL)
```sql
-- Check how many blank values exist
SELECT COUNT(*) 
FROM billing
WHERE totalcharges = ' ';

-- Replace blanks with NULL
UPDATE billing
SET totalcharges = NULL
WHERE totalcharges = ' ';
```

> **Note:** `TotalCharges` in the raw Kaggle dataset contains whitespace strings instead of proper NULLs for customers with no charges. This step converts them to proper NULLs.

---

## 📊 PostgreSQL — Exploratory Analysis

### Churn Rate by Status
```sql
SELECT Churn, 
       COUNT(*) AS total,
       ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(), 2) AS percentage
FROM customers
GROUP BY Churn;
```

### Churn by Contract Type
```sql
SELECT b.Contract, c.Churn, COUNT(*) AS total
FROM customers c
JOIN billing b ON c.customerid = b.customerID
GROUP BY b.Contract, c.Churn
ORDER BY b.Contract;
```

### Avg Monthly Charges by Churn Status
```sql
SELECT c.Churn, 
       ROUND(AVG(b.MonthlyCharges::numeric), 2) AS avg_monthly
FROM customers c
JOIN billing b ON c.customerID = b.customerID
GROUP BY c.churn;
```

### Churn by Tenure Group
```sql
SELECT 
  CASE 
    WHEN tenure <= 12 THEN '0-12 months'
    WHEN tenure <= 24 THEN '13-24 months'
    WHEN tenure <= 48 THEN '25-48 months'
    ELSE '48+ months'
  END AS tenure_group,
  Churn,
  COUNT(*) AS total
FROM customers
GROUP BY tenure_group, Churn
ORDER BY tenure_group;
```

### Churn by Internet Service Type
```sql
SELECT s.InternetService, c.Churn, COUNT(*) AS total
FROM customers c
JOIN services s ON c.customerid = s.customerid
GROUP BY s.InternetService, c.Churn
ORDER BY s.InternetService;
```

---

## 📐 DAX Measures

### Core KPIs

```dax
Total Customers = 
COUNTROWS('public customers')

Churned Customers = 
COUNTROWS(FILTER('public customers', 'public customers'[churn] = "yes"))

Retained Customers = 
COUNTROWS(FILTER('public customers', 'public customers'[churn] = "no"))

Churn Rate % = 
DIVIDE(
    COUNTROWS(FILTER('public customers', 'public customers'[churn] = "yes")),
    COUNTROWS('public customers'),
    0
)

Retention Rate % = 
DIVIDE(
    COUNTROWS(FILTER('public customers', 'public customers'[churn] = "no")),
    COUNTROWS('public customers'),
    0
)
```

### Revenue

```dax
Total Revenue = SUM('public billing'[totalcharges])

Revenue Lost to Churn = 
CALCULATE(
    SUM('public billing'[totalcharges]),
    'public customers'[churn] = "yes"
)
```

### Monthly Charges

```dax
Avg Monthly Charges = AVERAGE('public billing'[monthlycharges])

Avg Monthly Charge (Churned) = 
CALCULATE(
    AVERAGE('public billing'[monthlycharges]),
    'public customers'[churn] = "yes"
)

Avg Monthly Charge (Retained) = 
CALCULATE(
    AVERAGE('public billing'[monthlycharges]),
    'public customers'[churn] = "no"
)
```

### Tenure

```dax
Avg Tenure (Churned) = 
CALCULATE(
    AVERAGE('public customers'[tenure]),
    'public customers'[churn] = "yes"
)

Avg Tenure (Retained) = 
CALCULATE(
    AVERAGE('public customers'[tenure]),
    'public customers'[churn] = "no"
)
```

### Risk & Segmentation

```dax
High-Risk Customers = 
CALCULATE(
    'public customers'[Total Customers],
    'public billing'[contract] = "Month-to-month",
    'public services'[onlinesecurity] = "No"
)

Customers without Online Security = 
CALCULATE(
    'public customers'[Total Customers],
    'public services'[onlinesecurity] = "No"
)
```

### Calculated Columns

```dax
-- Groups customers by tenure length
Tenure Group = 
SWITCH(
    TRUE(),
    [tenure] <= 12, "0-12 Months",
    [tenure] <= 24, "1-2 Years",
    [tenure] <= 48, "2-4 Years",
    "4+ Years"
)

-- Converts churn flag to readable label
Churn Status = IF(
    'public customers'[churn] = "Yes",
    "Churned",
    "Retained"
)
```

> **Note:** `Tenure Group` and `Churn Status` are calculated columns added directly to the data model in Power BI, not measures.

---

## 📋 Dashboard Pages

### Page 1 — Telecom Customer Churn Overview

| Metric | Value |
|---|---|
| Total Customers | 7,000 |
| Churn Rate | 26.54% |
| Retention Rate | 73.46% |
| Revenue Lost to Churn | $3M |

---

### Page 2 — Churn Driver Analysis

| Metric | Value |
|---|---|
| Avg Monthly Charges (Churned) | $74.44 |
| Revenue Lost to Churn | $3M |
| Customers without Online Security | 3,000 |

---

### Page 3 — Customer Retention Strategy

| Metric | Value |
|---|---|
| Avg Monthly Charges (Retained) | $61.27 |
| Avg Tenure (Churned) | 17.98 months |
| High-Risk Customers | 3,000 |

**Key Retention Recommendations:**
- Promote long-term contracts
- Bundle online security services
- Encourage automatic payment methods

---

## 📌 Data Model

Three tables joined on `customerID`:

| Table | Key Columns |
|---|---|
| `public customers` | CustomerID, gender, SeniorCitizen, Partner, Dependents, tenure, Churn |
| `public billing` | CustomerID, Contract, PaperlessBilling, PaymentMethod, MonthlyCharges, TotalCharges |
| `public services` | CustomerID, PhoneService, MultipleLines, InternetService, OnlineSecurity, OnlineBackup, DeviceProtection, TechSupport, StreamingTV, StreamingMovies |

---

## 🔍 Key Findings

- **Month-to-month contracts** account for the majority of churned customers (1,655) and the highest revenue loss ($1.93M).
- **New customers (0–12 months)** churn the most — 1,037 customers — suggesting an early engagement problem.
- **Fiber optic users** churn significantly more than DSL users (1,297 vs 459), likely due to higher monthly costs.
- **Customers without online security** are at far higher churn risk (1,461 vs 295 with security).
- **Churned customers pay more on average** ($74/month) than retained ones ($61/month), indicating price sensitivity.
- **Retained customers stay ~2x longer** — avg tenure of 37.57 months vs 17.98 months for churned customers.
