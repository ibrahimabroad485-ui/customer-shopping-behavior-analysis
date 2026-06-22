<div align="center">

<!-- HEADER BANNER -->
<img src="dashboard/dashboard-image.png" alt="Customer Behavior Dashboard" width="100%"/>

<br/>

# 🛍️ Customer Shopping Behavior Analysis

### End-to-end retail analytics — from raw CSV to business recommendations

<br/>

![Python](https://img.shields.io/badge/Python-3.12-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Cleaning-150458?style=for-the-badge&logo=pandas&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-18-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Power BI](https://img.shields.io/badge/Power%20BI-Dashboard-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![Jupyter](https://img.shields.io/badge/JupyterLab-Notebook-F37626?style=for-the-badge&logo=jupyter&logoColor=white)

<br/>

![Status](https://img.shields.io/badge/Status-Complete-22C55E?style=flat-square)
![Records](https://img.shields.io/badge/Records-3%2C900-8B1A1A?style=flat-square)
![Queries](https://img.shields.io/badge/SQL%20Queries-10-4169E1?style=flat-square)
![Features](https://img.shields.io/badge/Features-18-E8A000?style=flat-square)

</div>

---

## ❓ The Business Problem

> *"How can the company leverage consumer shopping data to identify trends, improve customer engagement, and optimize marketing and product strategies?"*

A retail company with **3,900 customer records** needed answers. Management noticed shifting purchasing patterns across demographics, product categories, and channels — but had no structured analysis to act on. This project delivers that analysis: from a raw CSV to a fully interactive Power BI dashboard backed by Python and SQL.

---

## 📊 Dashboard Preview

<div align="center">
<img src="dashboard/dashboard-image.png" alt="Power BI Dashboard" width="90%"/>
<br/>
<em>Interactive Power BI dashboard with slicers for Subscription Status, Gender, Category, and Shipping Type</em>
</div>

---

## 🔢 Key Numbers at a Glance

<div align="center">

| Metric | Value |
|:--|:--|
| 👥 Total Customers | **3,900** |
| 💵 Average Purchase Amount | **$59.76** |
| ⭐ Average Review Rating | **3.75 / 5.0** |
| 🏆 Loyal Customers (11+ purchases) | **80% — 3,116 customers** |
| 🔓 Unsubscribed Repeat Buyers | **2,518 — largest revenue opportunity** |
| 📦 Top Revenue Category | **Clothing (~$100K)** |
| 👨 Male vs Female Revenue | **$157,890 vs $75,191** |

</div>

---

## 🔄 Analytics Pipeline

```
Raw CSV  ──▶  Python (JupyterLab)  ──▶  PostgreSQL (pgAdmin)  ──▶  Power BI  ──▶  Report & Presentation
  │                  │                          │                       │
  │           Clean · Engineer            10 SQL Queries          Interactive           PDF + PPTX
  │           Load to DB                  CTE · Window Fn         Dashboard             Deliverables
  └──────────────────────────────────────────────────────────────────────────────────────────────────▶
```

---

## 🐍 Phase 1 — Python / JupyterLab

All data preparation was done in JupyterLab using **Pandas**. The cleaned DataFrame was then exported directly to PostgreSQL via SQLAlchemy.

| Step | What Was Done | Detail |
|:--|:--|:--|
| **1. Load** | Imported raw dataset | `pd.read_csv()` → `df.head()`, `df.info()`, `df.describe()` |
| **2. Null Handling** | 37 missing values in `review_rating` | Filled with **category-level median** (not global avg) |
| **3. Column Standardization** | Renamed all columns to `snake_case` | `purchase_amount_(usd)` → `purchase_amount` |
| **4. Feature Engineering** | Created `age_group` | `pd.qcut(df["age"], q=4)` → Young Adult / Adult / Middle-aged / Senior |
| **5. Feature Engineering** | Created `purchase_frequency_days` | Mapped text labels (e.g. "Weekly" → 7, "Monthly" → 30) |
| **6. Redundancy Check** | `discount_applied` == `promo_code_used` — 100% identical | Dropped `promo_code_used` |
| **7. DB Export** | Loaded to PostgreSQL | `df.to_sql("customer", engine, if_exists="replace")` |

```python
# Missing value imputation — category-level median (smarter than global average)
df["review_rating"] = df.groupby("Category")["review_rating"].transform(
    lambda x: x.fillna(x.median())
)

# Age group feature engineering
labels = ["young Adult", "Adult", "Middle-aged", "Senior"]
df["age_group"] = pd.qcut(df["age"], q=4, labels=labels)

# PostgreSQL load
from sqlalchemy import create_engine
engine = create_engine("postgresql+psycopg2://user:pass@localhost:5432/customer_behavior")
df.to_sql("customer", engine, if_exists="replace", index=False)
```

---

## 🗄️ Phase 2 — SQL Analysis (PostgreSQL / pgAdmin)

Ten business queries were written in pgAdmin, ranging from basic aggregations to **CTEs** and **window functions**.

<details>
<summary><b>📋 View All 10 Queries</b></summary>

<br/>

**Q1 — Revenue by Gender** `GROUP BY + SUM`
```sql
SELECT gender, SUM(purchase_amount) AS revenue
FROM customer
GROUP BY gender;
-- Result: Female $75,191 | Male $157,890
```

**Q2 — High-Spend Discount Users** `Subquery`
```sql
SELECT customer_id, purchase_amount
FROM customer
WHERE discount_applied = 'Yes'
  AND purchase_amount >= (SELECT AVG(purchase_amount) FROM customer);
-- Result: 839 customers (~22% of base)
```

**Q3 — Top 5 Products by Review Rating** `AVG + ORDER BY`
```sql
SELECT item_purchased, ROUND(AVG(review_rating::numeric), 2) AS average
FROM customer
GROUP BY item_purchased
ORDER BY average DESC
LIMIT 5;
-- Result: Gloves 3.86 | Sandals 3.84 | Boots 3.82 | Hat 3.80 | Skirt 3.78
```

**Q4 — Spend by Shipping Type** `Filter + ROUND(AVG)`
```sql
SELECT shipping_type, ROUND(AVG(purchase_amount), 2) AS average
FROM customer
WHERE shipping_type IN ('Standard', 'Express')
GROUP BY shipping_type;
-- Result: Standard $58.46 | Express $60.48
```

**Q5 — Subscriber vs Non-Subscriber** `Multiple Aggregations`
```sql
SELECT subscription_status,
       ROUND(AVG(purchase_amount), 2) AS avg_spend,
       SUM(purchase_amount) AS total_revenue,
       COUNT(customer_id) AS total_customers
FROM customer
GROUP BY subscription_status
ORDER BY total_revenue, avg_spend DESC;
-- Result: Sub $59.49 avg / $62,645 total | Non-sub $59.87 avg / $170,436 total
```

**Q6 — Products with Highest Discount Rates** `CASE WHEN + SUM/COUNT`
```sql
SELECT item_purchased,
       ROUND(100 * SUM(CASE WHEN discount_applied = 'Yes' THEN 1 ELSE 0 END) / COUNT(*), 2) AS discount_rate
FROM customer
GROUP BY item_purchased
ORDER BY discount_rate DESC
LIMIT 5;
-- Result: Hat 50% | Sneakers 49% | Coat 49% | Sweater 48% | Pants 47%
```

**Q7 — Customer Loyalty Segmentation** `CTE + CASE WHEN`
```sql
WITH customer_type AS (
    SELECT customer_id, previous_purchases,
           CASE WHEN previous_purchases = 1 THEN 'New'
                WHEN previous_purchases BETWEEN 2 AND 10 THEN 'Returning'
                ELSE 'Loyal' END AS customer_segment
    FROM customer
)
SELECT customer_segment, COUNT(customer_segment) AS number_of_customers
FROM customer_type
GROUP BY customer_segment
ORDER BY number_of_customers DESC;
-- Result: Loyal 3,116 | Returning 701 | New 83
```

**Q8 — Top 3 Products per Category** `Window Function: ROW_NUMBER()`
```sql
WITH item_counts AS (
    SELECT category, item_purchased, COUNT(customer_id) AS total_orders,
           ROW_NUMBER() OVER (PARTITION BY category ORDER BY COUNT(customer_id) DESC) AS item_rank
    FROM customer
    GROUP BY category, item_purchased
)
SELECT item_rank, category, item_purchased, total_orders
FROM item_counts
WHERE item_rank <= 3;
```

**Q9 — Repeat Buyer Subscription Likelihood** `Filter + GROUP BY`
```sql
SELECT subscription_status, COUNT(customer_id) AS repeat_buyers
FROM customer
WHERE previous_purchases > 5
GROUP BY subscription_status;
-- Result: Not subscribed 2,518 | Subscribed 958
```

**Q10 — Revenue by Age Group** `GROUP BY derived feature`
```sql
SELECT age_group, SUM(purchase_amount) AS revenue
FROM customer
GROUP BY age_group
ORDER BY revenue DESC;
-- Result: Young Adult $62,143 | Middle-aged $59,197 | Adult $55,978 | Senior $55,763
```

</details>

---

## 📈 Phase 3 — Power BI Dashboard

Built an interactive dashboard with **6 slicers** and **6 chart types** for non-technical stakeholders.

| Component | Description |
|:--|:--|
| 📌 KPI Cards | 3.9K Customers · $59.76 Avg Purchase · 3.75★ Avg Rating |
| 🍩 Donut Chart | Subscription split — 73% unsubscribed vs 27% subscribed |
| 📊 Revenue by Category | Clothing ~$100K · Accessories ~$70K · Footwear · Outerwear |
| 📊 Sales by Category | Clothing ~1,750 transactions leads all categories |
| 📉 Revenue by Age Group | Nearly even across Young Adult · Middle-aged · Adult · Senior |
| 🎛️ Slicers | Subscription Status · Gender · Category · Shipping Type |

---

## 💡 Key Findings

<table>
<tr>
<td width="50%" valign="top">

**Revenue & Demographics**
- 👨 Male customers: **68% of total revenue** ($157,890 vs $75,191)
- 📊 Revenue is **nearly equal** across all 4 age groups
- 👕 **Clothing** leads in both revenue AND transaction volume

</td>
<td width="50%" valign="top">

**Pricing & Discount Behavior**
- 🏷️ **839 customers** (22%) used discounts yet spent above average
- 💳 Subscribers & non-subscribers: **nearly identical avg spend**
- 🚚 Express shipping users spend **$2 more** than Standard

</td>
</tr>
<tr>
<td width="50%" valign="top">

**Product Performance**
- ⭐ Top rated: **Gloves (3.86), Sandals (3.84), Boots (3.82)**
- 🛒 Top sellers: Blouse, Pants, Jewelry, Sandals, Jacket
- ⚠️ **Top-rated ≠ Top-selling** — major positioning opportunity

</td>
<td width="50%" valign="top">

**Loyalty & Retention**
- 🏆 **80% of customers are Loyal** (11+ purchases)
- 🆕 Only **83 customers (2%)** are brand new
- 🎯 **2,518 repeat buyers are NOT subscribed** — highest-ROI target

</td>
</tr>
</table>

---

## 🚀 Business Recommendations

| Priority | Recommendation | Based On |
|:--|:--|:--|
| 🔴 **Highest** | Target 2,518 unsubscribed repeat buyers with exclusive offers | Q5 + Q9 |
| 🟠 **High** | Audit female ad reach; build Accessories & Footwear campaigns | Q1 |
| 🟡 **Quick Win** | Feature top-rated products (Gloves, Sandals) on homepage | Q3 |
| 🟢 **Strategic** | Replace blanket discounts with tiered basket-size incentives | Q2 + Q6 |
| 🔵 **Operational** | Offer free Express shipping above $75+ to grow basket size | Q4 |
| ⚪ **Long-term** | Launch Silver / Gold / Platinum loyalty tier program | Q7 |

---

## 📁 Repository Structure

```
📂 dashboard/
│   ├── Customer_Behavior_Dashboard.pbix     # Power BI dashboard file
│   └── dashboard-image.png                  # Dashboard preview
📂 data/
│   └── customer_shopping_behavior.csv       # Raw dataset (3,900 records)
📂 presentation/
│   └── Customer_Shopping_Behavior_Presentation.pptx  # Slide deck (14 slides)
📂 python/
│   └── Customer_Shopping_Behavior_Analysis.ipynb  # Full EDA & cleaning notebook
📂 report/
│   └── Customer_Shopping_Behavior_Report.pdf  # Full analytical report
📂 sql/
│   └── SQL-Queries.sql                      # All 10 business queries

LICENSE
README.md

```

---

## 🛠️ Technical Skills Demonstrated

<div align="center">

| Category | Skills |
|:--|:--|
| **Python** | Pandas · Feature Engineering (`pd.qcut`, `.map`) · Null Imputation · SQLAlchemy |
| **SQL** | Aggregations · Subqueries · CTEs (`WITH`) · Window Functions (`ROW_NUMBER OVER PARTITION BY`) |
| **Power BI** | Data Modeling · DAX Measures · KPI Cards · Interactive Slicers |
| **Analytics** | EDA · Customer Segmentation · Revenue Attribution · Loyalty Classification |
| **Communication** | 17-page Report · 14-slide Presentation · Business Storytelling |

</div>

---

## 📄 Project Deliverables

| Deliverable | Description | Link |
|:--|:--|:--|
| 📓 Python Notebook | Full EDA, cleaning, and feature engineering | `python/` |
| 🗄️ SQL Queries | All 10 business queries with comments | `sql/` |
| 📊 Power BI Dashboard | Interactive dashboard with slicers | `dashboard/` |
| 📋 Analytical Report | 17-page PDF with findings and visualizations | `report/` |
| 🎞️ Presentation | 14-slide PPTX with live charts | `presentation/` |

---

<div align="center">

---

**Ibrahim Jamil**

Data Science & Computer Science · Luther College · Class of 2028

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)]([https://linkedin.com/in/YOUR_USERNAME](https://www.linkedin.com/in/ibrahim-jamil-5784a9395/?skipRedirect=true))
[![GitHub](https://img.shields.io/badge/GitHub-Follow-181717?style=for-the-badge&logo=github&logoColor=white)]([https://github.com/YOUR_USERNAME](https://github.com/ibrahimabroad485-ui))

*Sophomore targeting Data Analytics Internship — Summer 2027*

</div>
