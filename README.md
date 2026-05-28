# 📊 Telecom Customer Churn Analysis

A Power BI project analyzing customer churn behavior for a telecom company. The report identifies churn patterns, revenue impact, and key risk drivers — culminating in actionable retention recommendations.

**Author:** Navneeth Krishna  
**LinkedIn:** [navneethkrishna2004](https://www.linkedin.com/in/navneethkrishna2004)  
**Data Source:** [Telco Customer Churn — Kaggle](https://www.kaggle.com/datasets/blastchar/telco-customer-churn)

---

## 📁 Project Structure

```
telecom-churn-analysis/
│
├── ChurnAnalysis.pbix              # Power BI report file
├── Churn_Analysis_Report.pdf       # Exported dashboard (3 pages)
├── queries/
│   └── churn_queries.sql           # PostgreSQL queries for data exploration
├── screenshots/
│   ├── dashboard_overview.jpg
│   ├── dashboard_churn_drivers.jpg
│   └── dashboard_retention.jpg
└── README.md
```

---

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| PostgreSQL | Data exploration and initial analysis |
| Power BI Desktop | Data modeling, DAX measures, and dashboard creation |
| DAX | Calculated measures and columns |

---

## ⚙️ How to Run

### 1. Set Up the Database (PostgreSQL)

1. Download the dataset from [Kaggle](https://www.kaggle.com/datasets/blastchar/telco-customer-churn)
2. Create a PostgreSQL database and import the CSV
3. Run the SQL scripts in `queries/churn_queries.sql` to explore the data

### 2. Open in Power BI

1. Open `ChurnAnalysis.pbix` in **Power BI Desktop**
2. Update the data source connection to point to your PostgreSQL instance if needed (`Home → Transform Data → Data Source Settings`)
3. Refresh the data and the visuals will populate automatically

---

## 📋 Dashboard Pages

### Page 1 — Telecom Customer Churn Overview
![Churn Overview](screenshots/dashboard_overview.jpg)

High-level KPIs and churn breakdown by contract type and customer tenure.

| Metric | Value |
|---|---|
| Total Customers | 7,000 |
| Churn Rate | 26.54% |
| Retention Rate | 73.46% |
| Revenue Lost to Churn | $3M |

**Key Visuals:**
- Customers Churned by Contract Type
- Revenue Lost by Contract Type
- Customers Churned by Tenure Group
- Avg Monthly Charges by Churn Status

---

### Page 2 — Churn Driver Analysis
![Churn Driver Analysis](screenshots/dashboard_churn_drivers.jpg)

Explores which service features correlate most strongly with churn.

| Metric | Value |
|---|---|
| Avg Monthly Charges (Churned) | $74.44 |
| Revenue Lost to Churn | $3M |
| Customers without Online Security | 3,000 |

**Key Visuals:**
- Churned Customers by Online Security (No: 1,461 vs Yes: 295)
- Churned Customers by Internet Service (Fiber optic: 1,297 — highest)
- Churned Customers by Online Backup
- Churned Customers by Tech Support
- Churned Customers by Paperless Billing

---

### Page 3 — Customer Retention Strategy
![Customer Retention Strategy](screenshots/dashboard_retention.jpg)

Focuses on retained customers and actionable strategies to reduce churn.

| Metric | Value |
|---|---|
| Revenue Lost to Churn | $3M |
| Avg Monthly Charges (Retained) | $61.27 |
| Avg Tenure (Churned) | 17.98 months |
| High-Risk Customers | 3,000 |

**Key Retention Recommendations:**
- Promote long-term contracts
- Bundle online security services
- Encourage automatic payment methods

---

## 🗄️ SQL Queries

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

## 📌 Data Model

Three tables joined on `customerID`:

| Table | Key Columns |
|---|---|
| `public customers` | customerID, churn, tenure |
| `public billing` | customerID, contract, monthlycharges, totalcharges |
| `public services` | customerID, internetservice, onlinesecurity, onlinebackup, techsupport, paperlessbilling |

---

## 🔍 Key Findings

- **Month-to-month contracts** account for the majority of churned customers (1,655) and the highest revenue loss ($1.93M).
- **New customers (0–12 months)** churn the most — 1,037 customers — suggesting an early engagement problem.
- **Fiber optic users** churn significantly more than DSL users (1,297 vs 459), likely due to higher monthly costs.
- **Customers without online security** are at far higher churn risk (1,461 vs 295 with security).
- **Churned customers pay more on average** ($74/month) than retained ones ($61/month), indicating price sensitivity.
- **Retained customers stay ~2x longer** — avg tenure of 37.57 months vs 17.98 months for churned customers.
