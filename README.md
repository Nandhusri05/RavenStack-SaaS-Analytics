# RavenStack SaaS Analytics

An end-to-end SaaS analytics project built with **Microsoft Fabric, SQL, Python/PySpark, and Power BI** to analyze recurring revenue, customer growth, subscription behavior, product usage, support performance, and churn patterns.

> **Note:** The RavenStack dataset is fully synthetic. It is intended for educational and portfolio use.

---

## 📌 Project Overview

RavenStack is a synthetic SaaS company with customer, subscription, product-usage, support-ticket, and churn-event data.

The objective of this project was to build an analytics workflow that moves from **raw multi-table data → data-quality validation → business analysis → behavioral analysis → semantic modeling → interactive Power BI reporting**.

### Business Questions

The analysis focuses on:

- How is recurring revenue growing?
- Which subscription plans contribute the most revenue?
- How is the customer base distributed across plans and industries?
- Which acquisition channels bring customers?
- How frequently do customers upgrade or downgrade?
- How does product usage vary across plans?
- Which product features have higher error rates?
- How does support workload vary by priority and plan?
- What are the most frequently recorded churn reasons?
- Which customer segments show different churn patterns?
- How much recurring revenue is associated with churn?
- Are product usage, support activity, MRR, seats, or tenure clearly associated with churn?

---

## 🛠️ Tech Stack

| Area            | Tools                                  |
| --------------- | -------------------------------------- |
| Data Platform   | Microsoft Fabric                       |
| Storage         | Fabric Lakehouse                       |
| SQL Analysis    | Spark SQL / SQL                        |
| Python Analysis | Python, PySpark, pandas-style analysis |
| Data Modeling   | Power BI Semantic Model                |
| Visualization   | Power BI                               |
| Dashboard       | Microsoft Fabric Power BI Report       |
| Version Control | Git / GitHub                           |

---

## 🗂️ Dataset

The project uses the **RavenStack SaaS Subscription & Churn Analytics Dataset** created by **River @ Rivalytics**.

The dataset is synthetic and contains no real customer PII.

### Tables

| Table             |   Rows | Purpose                                                 |
| ----------------- | -----: | ------------------------------------------------------- |
| `accounts`        |    500 | Customer/account information                            |
| `subscriptions`   |  5,000 | Subscription, MRR, ARR, plan and lifecycle information  |
| `feature_usage`   | 25,000 | Product feature usage and error activity                |
| `support_tickets` |  2,000 | Support workload, response, resolution and satisfaction |
| `churn_events`    |    600 | Churn reasons, refunds and lifecycle events             |

### Data Relationships

```text
accounts
   │
   ├── subscriptions
   │       │
   │       └── feature_usage
   │
   ├── support_tickets
   │
   └── churn_events
```

`feature_usage` is connected to customers through:

```text
feature_usage.subscription_id
        ↓
subscriptions.subscription_id
        ↓
subscriptions.account_id
        ↓
accounts.account_id
```

### Data Source

Kaggle: **RavenStack SaaS Subscription & Churn Analytics Dataset**

https://www.kaggle.com/datasets/rivalytics/saas-subscription-and-churn-analytics-dataset

Dataset attribution: **River @ Rivalytics**

---

# 🔍 Project Workflow

## 1. Data Quality Validation — SQL

Before performing business analysis, the five source tables were validated in Microsoft Fabric.

### Checks performed

- Row-count validation
- Primary-key uniqueness
- Duplicate-key investigation
- Full-row duplicate validation
- Missing-value checks
- Date and numerical business-rule validation
- Foreign-key / referential-integrity validation
- Duplicate record validation

### Important Data-Quality Finding

`feature_usage` contains:

- 25,000 total records
- 24,979 unique `usage_id` values
- 21 duplicated `usage_id` values

Investigation showed that these were **different usage events sharing the same ID**, rather than exact duplicate rows.

Therefore, the records were preserved rather than incorrectly deleted.

This was an important data-quality decision because removing them would have resulted in loss of valid usage events.

---

## 2. Business Analysis — SQL

SQL analysis was used to establish the business baseline and answer revenue, subscription, acquisition, and churn questions.

### Key analysis areas

- Overall SaaS KPIs
- MRR and ARR analysis
- MRR contribution by plan
- Month-over-month MRR growth
- Subscription status and churn
- Churn by plan
- Churn by industry
- Churn by referral source
- Subscription duration
- Revenue associated with churn
- Customer acquisition and growth
- Billing frequency
- Upgrade and downgrade activity

### Selected Findings

- The dataset contains **500 customers** and **5,000 subscriptions**.
- **4,514 subscriptions** are active at the subscription level.
- Subscription-level churn is **9.72%** based on 486 churned subscriptions.
- Enterprise subscriptions generate substantially more recurring revenue per subscription than Pro and Basic.
- Churn rates across Basic, Pro, and Enterprise subscriptions are relatively close, ranging from **9.49% to 9.98%**.
- Churned subscriptions have a substantially shorter average subscription duration than active subscriptions.
- Enterprise accounts represent the largest share of recurring revenue associated with churned subscriptions.
- Customer acquisition and subscription activity increase strongly toward the later part of the observed period.

> **Metric definition note:** Customer-level churn and subscription-level churn are intentionally treated as different metrics because the dataset contains multiple subscriptions per account. The dashboard uses an account-level customer churn view, while the SQL business analysis also evaluates subscription-level churn.

---

# 🐍 3. Python / PySpark Analysis

Python/PySpark was used for deeper behavioral analysis that was less convenient to express through SQL alone.

### Analysis areas

1. Feature usage vs. churn
2. Support activity and satisfaction vs. churn
3. Customer engagement patterns
4. Customer segmentation
5. Churned vs. retained customer comparison
6. Potential churn-risk patterns
7. Product usage by plan
8. Feature error rates
9. Beta feature adoption
10. Support workload and resolution by plan
11. Churn reasons
12. Customer lifecycle changes
13. Final churn-risk pattern analysis

### Key Findings

#### Product Usage vs. Churn

Product usage differences between churned and retained customers were small.

The analysis therefore did **not** identify product usage alone as a strong churn indicator.

#### Support vs. Churn

Support activity, resolution time, and satisfaction were also broadly similar between churned and retained customers.

This suggests that support metrics alone were not strong churn differentiators in this dataset.

#### Customer Segmentation

At the account level:

- DevTools: **30.97% churn**
- FinTech: **22.32% churn**
- HealthTech: **21.88% churn**
- EdTech: **16.46% churn**
- Cybersecurity: **16.00% churn**

Plan-level customer churn was approximately **22% across Basic, Pro, and Enterprise**, showing relatively small differences between plans at the account level.

#### Churn Reasons

Among 600 recorded churn events:

| Reason     | Events |  Share |
| ---------- | -----: | -----: |
| Features   |    114 | 19.00% |
| Support    |    104 | 17.33% |
| Budget     |    104 | 17.33% |
| Unknown    |     95 | 15.83% |
| Competitor |     92 | 15.33% |
| Pricing    |     91 | 15.17% |

The **Features** category was the most frequently recorded churn reason.

The relatively large **Unknown** category also indicates an opportunity to improve churn-reason tracking.

#### Feature Error Rates

The highest observed feature error rates were:

- `feature_4`: **6.56%**
- `feature_9`: **6.51%**
- `feature_26`: **6.45%**

These are areas that could be investigated further by product or engineering teams.

#### Lifecycle Events

Of 600 recorded churn events:

- **123 (20.5%)** were preceded by an upgrade.
- **53 (8.83%)** were preceded by a downgrade.
- **61 (10.17%)** were associated with reactivation events.

These are associations observed in the dataset and should not be interpreted as causal relationships.

---

# 📊 4. Power BI Dashboard

The final dashboard was built using a Power BI semantic model in Microsoft Fabric.

### Semantic Model

The model contains five related tables:

```text
accounts
subscriptions
feature_usage
support_tickets
churn_events
```

Relationships:

```text
accounts 1 ─── * subscriptions
subscriptions 1 ─── * feature_usage
accounts 1 ─── * support_tickets
accounts 1 ─── * churn_events
```

### DAX Measures

Key measures include:

- Total MRR
- Total ARR
- Total Customers
- Active Customers
- Churned Customers
- Customer Churn Rate
- ARPU
- Active Subscriptions
- Churned Subscriptions
- Total Subscriptions
- Upgrade Subscriptions
- Downgrade Subscriptions

---

# 📈 Dashboard Pages

## Page 1 — Executive Overview

**Key insights into revenue, growth, and customer retention**

Includes:

- Total MRR
- Total ARR
- Active Customers
- Customer Churn Rate
- ARPU
- Revenue by Plan
- MRR by Start Month
- Customer Status
- Customer Signups by Month

Global slicers:

- Plan Tier
- Industry

![Executive Overview](./Images/01%20Executive%20Overview.png)

---

## Page 2 — Customer & Subscription Analysis

**Customer trends, subscription patterns, and acquisition insights**

Includes:

- Customers by Plan
- Customers by Industry
- Customer Acquisition Channels
- Upgrades vs Downgrades by Plan
- Billing Frequency
- Subscriptions by Month

![Customer & Subscription Analysis](./Images/02%20Customer%20&%20Subscription%20Analysis.png)

---

## Page 3 — Product & Support

**Product engagement and support performance insights**

Includes:

- Feature Adoption
- Usage by Plan
- Errors by Feature
- Support Tickets by Priority
- Resolution Time by Priority
- Customer Satisfaction by Priority

![Product & Support](./Images/03%20Product%20&%20Support.png)

---

## Page 4 — Churn & Retention Analysis

**Customer churn patterns, retention trends, and subscription risk insights**

Includes:

- Churn Rate by Plan
- Churn Rate by Industry
- Churn Reasons
- Customer Value vs Seats by Plan
- Support Tickets: Churned vs Active
- Churned MRR Contribution by Plan

![Churn & Retention Analysis](./Images/04%20Churn%20&%20Retention%20Analysis.png)

---

# 💡 Key Business Takeaways

1. **Enterprise subscriptions are the major recurring-revenue contributor.**
2. **Subscription-level churn is relatively similar across the three plan tiers.**
3. **Customer-level churn patterns differ across industries.**
4. **Features, support, and budget are among the most frequently recorded churn reasons.**
5. **Enterprise churn carries a larger recurring-revenue impact because of higher subscription value.**
6. **Product usage and support metrics do not show strong standalone relationships with churn in this dataset.**
7. **Higher feature error rates can highlight areas for additional product investigation.**
8. **Lifecycle events such as upgrades, downgrades, and reactivations are useful signals for further churn analysis.**
9. **Churn analysis should distinguish account-level churn from subscription-level churn because the dataset contains multiple subscriptions per account.**

---

# 📁 Repository Structure

```text
RavenStack-SaaS-Analytics/
│
├── README.md
│
├── SQL/
│   ├── 01_SQL_Data_Quality.ipynb
│   └── 02_SQL_Business_Analysis.ipynb
│
├── Python/
│   └── 03_Python_Analysis.ipynb
│
├── Power BI/
│   └── RavenStack SaaS Analytics Dashboard.pbix
│
├── Images/
│   ├── 01 Executive Overview.png
│   ├── 02 Customer & Subscription Analysis.png
│   ├── 03 Product & Support.png
│   └── 04 Churn & Retention Analysis.png
│
└── Dataset/
    ├── dataset.zip
```

---

# ▶️ How to Reproduce the Analysis

1. Obtain the RavenStack dataset from Kaggle.
2. Create a Microsoft Fabric workspace.
3. Create a Lakehouse.
4. Load the five CSV files into the Lakehouse.
5. Run `01_SQL_Data_Quality.ipynb`.
6. Review and document data-quality findings.
7. Run `02_SQL_Business_Analysis.ipynb`.
8. Run `03_Python_Analysis.ipynb`.
9. Create the semantic model using the five Lakehouse tables.
10. Add the required DAX measures.
11. Build the four-page Power BI report.
12. Add and synchronize the Plan Tier and Industry slicers across the report.

---

# 🎯 Project Outcome

This project demonstrates an end-to-end analytics workflow rather than only a dashboard:

**Data ingestion → Data quality → SQL analysis → Python/PySpark analysis → Data modeling → DAX → Power BI visualization → Business insights**

It demonstrates practical skills in:

- SQL
- Python
- PySpark
- Microsoft Fabric
- Lakehouse architecture
- Data quality validation
- Relational data modeling
- DAX
- Power BI
- SaaS metrics
- Customer analytics
- Churn analysis
- Business storytelling

---

## 📚 Dataset Attribution

**RavenStack Synthetic SaaS Dataset**

Created by **River @ Rivalytics**.

The dataset documentation states that it is synthetic, contains no PII, and is intended for educational and portfolio use with attribution.

Source: https://www.kaggle.com/datasets/rivalytics/saas-subscription-and-churn-analytics-dataset

---

## 👤 Author

**Nandhusri Rajaraman**

Aspiring Data Analyst | Python | SQL | Power BI | Data Visualization | Business Analytics
