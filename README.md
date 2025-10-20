# 🍽️ Samba Restaurant SQL Analytics Portfolio Project (Snowflake)

### 📊 Advanced Business Intelligence & Data Analytics Project  
**By:** [Your Name]  
**Tools:** Snowflake SQL | Power BI | Python | GitHub | Data Modeling | ERD | ELT | SQL Analytics  

---

## 🧭 Project Overview

The **Samba Restaurant SQL Analytics Portfolio Project** demonstrates end-to-end analytics capability — from data modeling and pipeline design (ELT) to advanced SQL insights and BI reporting — for a **multi-branch restaurant chain** operating across **five Kenyan cities** from **2009 to 2022**.

This project forms part of the **Samba Data Engineering and Analytics Portfolio**, where the engineering phase established clean, modeled data in Snowflake, and this analytics phase focuses on deriving **data-driven business insights**.

---

## 🎯 Business Case

Samba Restaurants operates in Kenya’s fast-growing hospitality sector, offering a variety of menu items across branches in **Nairobi, Mombasa, Kisumu, Nakuru, and Eldoret**.  
With over **500,000 historical sales transactions**, management needed a clear analytics solution to answer:

- How have **revenues** and **sales volumes** evolved over the years?  
- Which **products and categories** drive profitability across cities?  
- Which **branches outperform or underperform** compared to peers?  
- How do **public holidays** and **seasons** affect sales?  
- Which **staff members and cities** contribute most to operational efficiency?

This SQL-based BI solution addresses these needs through structured analysis in **Snowflake**, leading to dashboards and visual insights in **Power BI**.

---

## 🧱 Data Architecture & Schema

All data resides in the `PRODUCTION` schema within Snowflake.  
It follows a **star schema** with one central fact table and multiple dimensions:

```
PRODUCTION.FACT_SALES
PRODUCTION.DIM_DATE
PRODUCTION.DIM_PRODUCT
PRODUCTION.DIM_CITY
PRODUCTION.DIM_BRANCH
PRODUCTION.DIM_STAFF
PRODUCTION.DIM_KENYA_PUBLIC_HOLIDAYS
```

- **FACT_SALES**: Core sales transactions with measures (revenue, quantity, cost, etc.)
- **DIM_PRODUCT**: Menu items and categories
- **DIM_BRANCH / DIM_CITY**: Geographical hierarchy
- **DIM_DATE**: Calendar dimension (year, month, quarter)
- **DIM_STAFF**: Employee and shift-level data
- **DIM_KENYA_PUBLIC_HOLIDAYS**: National holidays for seasonal analysis

📘 *Refer to:* [`ERD_Diagram/Samba_ERD_Diagram.png`](./02_Data_Analytics_Phase/ERD_Diagram/Samba_ERD_Diagram.png)

---

## 🧩 Analytical Objectives

1. **Revenue Growth Analysis** – Assess trends and YoY growth across years and cities.  
2. **Product & Category Insights** – Identify high-performing menu categories.  
3. **Branch Benchmarking** – Compare branches against peers and city averages.  
4. **Seasonality & Public Holiday Impact** – Evaluate demand patterns on Kenyan holidays.  
5. **Staff Productivity** – Measure efficiency using revenue per staff-hour (where data available).

---

## 📈 Key Deliverables

| Deliverable | Description |
|--------------|-------------|
| **Problem_Statement.md** | Executive problem framing and analytical scope |
| **SQL_Solutions.md** | 15 curated Snowflake SQL queries (Simple → Advanced) |
| **ERD Diagram** | Visual schema overview |
| **Power BI Dashboard** | (Optional) Visual insights and KPI tracking |
| **Data Samples** | CSV extracts for schema understanding |
| **README.md** | Project documentation (this file) |

---

## 🧠 SQL Query Categories

| Level | Example Focus | Description |
|--------|----------------|-------------|
| 🟢 **Simple (5)** | Annual revenue, top products, average ticket | Foundational analysis |
| 🟡 **Intermediate (5)** | Branch ranking, category share, city benchmarks | Aggregations, joins |
| 🔴 **Advanced (5)** | Rolling averages, product affinity, YoY growth, branch clustering | Window functions, advanced analytics |

📄 *See full query documentation in:*  
[`SQL_Solutions.md`](./02_Data_Analytics_Phase/SQL_Solutions.md)

---

## 💡 Example Insights (Highlights)

- **Revenue grew 34% YoY (2021 → 2022)**, driven by expansion in Nairobi and Mombasa.  
- **Top-selling categories:** Grilled meats and beverages contributed 58% of total sales.  
- **Holiday uplift:** National holidays like *Jamhuri Day* and *Madaraka Day* showed 15–20% higher sales vs regular weekdays.  
- **Branch benchmarking:** Nairobi branches outperformed peers; Eldoret underperformed relative to city averages.  
- **Staff productivity:** Revenue per staff-hour correlated positively with shift length consistency and city-level footfall.

---

## 🧰 Tech Stack

| Tool | Purpose |
|------|----------|
| **Snowflake SQL** | Data warehousing & analytics |
| **Power BI** | Data visualization & storytelling |
| **Python / Pandas** | Data exploration and transformations |
| **GitHub** | Version control & portfolio presentation |
| **ERD / Draw.io / Mermaid** | Schema and relationship visualization |

---

## 📂 Repository Structure

```
Samba_Restaurant_SQL_Analytics_Portfolio_Project/
│
├── README.md
│
├── 01_Data_Engineering_Phase/
│   ├── Snowflake_Table_DDLs/
│   ├── ELT_Pipeline_Overview.md
│   └── Data_Modeling_ERD.png
│
├── 02_Data_Analytics_Phase/
│   ├── Problem_Statement.md
│   ├── SQL_Solutions.md
│   └── ERD_Diagram/Samba_ERD_Diagram.png
│
├── 03_BI_Dashboard_and_Insights/
│   ├── Dashboard_Design.md
│   ├── Visuals/
│   └── Key_Insights_Summary.md
│
├── data_samples/
│   ├── FACT_SALES_sample.csv
│   ├── DIM_DATE_sample.csv
│   └── DIM_KENYA_PUBLIC_HOLIDAYS_sample.csv
│
├── notebooks/
│   ├── SQL_Exploration_and_EDA.ipynb
│   └── Snowflake_Connection_Template.ipynb
│
└── LICENSE
```

---

## 🚀 How to Use

1. Clone the repository:
   ```bash
   git clone https://github.com/founa67/Samba_Restaurant_SQL_Analytics_Portfolio_Project.git
   ```
2. Open the Snowflake SQL console or VS Code with Snowflake connector.  
3. Execute queries from `02_Data_Analytics_Phase/SQL_Solutions.md`.  
4. Optionally connect Power BI to Snowflake for visualization.  
5. Review dashboards and export insights.

---

## 🧾 License & Attribution

This project is open-sourced under the **MIT License**.  
Data used is **synthetic / anonymized** for educational and portfolio demonstration purposes.  

---

## ✨ Author

**[Your Full Name]**  
Snowflake SQL | BI Analytics | Data Engineering | Kenya Market Analytics  
📧 [frankouna96@gmail.com]  
🌐 [[LinkedIn / Portfolio Website](https://www.linkedin.com/in/frankouna/)]

---

## ⭐ Acknowledgment

This project is part of the **Samba Data Engineering & Analytics Portfolio Series**, focused on demonstrating full-lifecycle data solutions — from **data ingestion and modeling (ELT)** to **analytics, BI, and executive decision insights**.

---
