# HR Analytics Dashboard 📊 — Employee Attrition

## **Overview**

This project is a comprehensive, portfolio-grade HR analytics solution focused on understanding **employee attrition**. It showcases end-to-end skills in **data cleaning**, **SQL-based analysis (PostgreSQL)**, and **interactive visualization** using **Tableau** and **Excel**. The deliverables are structured for real-world stakeholders—HR leadership, People Analytics, and Business Partners—to enable **data-driven retention strategies**.

---

## **Project Goals**

**Primary Objective:** Build an interactive dashboard that surfaces the **drivers of employee turnover** and answers core business questions:

* **What is our overall headcount and attrition rate?**
* **Which demographics, departments, and roles see disproportionate attrition?**
* **How do job satisfaction and work–life balance relate to attrition?**
* **What do performance and compensation patterns suggest about risk?**

By translating raw HR data into clear KPIs and visual narratives, the project enables targeted **retention** and **employee experience** initiatives.

---

## **Tech Stack**

* **Data Analysis & Modeling:** **SQL (PostgreSQL)**
* **Data Storage:** CSV, Excel
* **Visualization:** **Tableau**, **Excel** 

---

## **Dataset**

**Primary Source:** `Data/HR_Analytics_Dataset.csv`
**Validation Sample:** `Data/hr_analytics_validation_sample.csv` (used to test queries before scaling)

**Key Columns (selected):**

* **Attrition** *(Yes/No)* — target variable
* **Age** *(with derived CF\_age band)*
* **Department** *(e.g., Sales, R\&D, HR)*
* **JobRole**
* **JobSatisfaction** *(1–4 scale in many public HR datasets)*
* **YearsAtCompany**
* **MonthlyIncome**

> The project creates derived fields (e.g., **age bands**) and aggregates (e.g., **attrition rate**) to support dashboard visuals and consistent KPI definitions.

---

## **Repository Structure**

```
HR-Analytics-Dashboard/
├─ Data/
│  ├─ HR_Analytics_Dataset.csv
│  └─ hr_analytics_validation_sample.csv
├─ Excel/
│  └─ HR_Analytics_Dashboard.xlsx
├─ Sql/
│  └─ hr_analytics_dashboard_validation_queries.sql
└─ Tableau/
   └─ HR_Analytics_Dashboard_Tableau.twb(x)
```

* **Data**: Core CSVs (source + validation)
* **Excel**: Clickable dashboard & pivot outputs
* **Sql**: Clean, reproducible queries (development + validation)
* **Tableau**: Published workbook packaged with calculated fields and worksheets

---

## **KPIs & Business Logic**

**Core KPIs**

* **Headcount** = `COUNT(*)`
* **Attrition Count** = `SUM(CASE WHEN Attrition='Yes' THEN 1 ELSE 0 END)`
* **Attrition Rate** = `Attrition Count / Headcount`
* **Avg Age / Tenure / Income** = `AVG(Age)`, `AVG(YearsAtCompany)`, `AVG(MonthlyIncome)`
* **Job Satisfaction Score (Avg)** = `AVG(JobSatisfaction)`

**Segmentation Dimensions**

* **Demographics**: Age bands, gender (if available)
* **Org Structure**: Department, Job Role
* **Experience**: YearsAtCompany (bucketed)
* **Engagement/EX**: JobSatisfaction, WorkLifeBalance (if provided)
* **Compensation**: MonthlyIncome (bands/quantiles)

**Standardized Buckets (examples)**

```sql
-- Age Banding
CASE
  WHEN age BETWEEN 18 AND 24 THEN '18-24'
  WHEN age BETWEEN 25 AND 34 THEN '25-34'
  WHEN age BETWEEN 35 AND 44 THEN '35-44'
  WHEN age BETWEEN 45 AND 54 THEN '45-54'
  ELSE '55+'
END AS age_band;

-- Income Banding (example quantiles)
CASE
  WHEN MonthlyIncome < 3000 THEN '< 3k'
  WHEN MonthlyIncome BETWEEN 3000 AND 6000 THEN '3k-6k'
  WHEN MonthlyIncome BETWEEN 6001 AND 9000 THEN '6k-9k'
  ELSE '9k+'
END AS income_band;
```

> **Why standardize?** Consistent banding ensures visually comparable charts in Tableau/Excel and prevents skew from outliers.

---

## **Analytical Approach (SQL)**

The SQL in `Sql/hr_analytics_dashboard_validation_queries.sql` follows a **clean-room** approach:

1. **Validation First**

   * Run on `hr_analytics_validation_sample.csv` (ingested to a staging table) to confirm logic.
2. **Production Queries**

   * Scale the same logic to the full `HR_Analytics_Dataset.csv`.
3. **CTEs & Reusable Logic**

   * Use **CTEs** for clarity (cleaning, banding, KPIs) and to simplify downstream consumption by Tableau/Excel.

**Representative Query Patterns**

```sql
-- Overall KPIs
WITH base AS (
  SELECT
    Attrition,
    Age,
    Department,
    JobRole,
    JobSatisfaction,
    YearsAtCompany,
    MonthlyIncome
  FROM hr_analytics
),
kpis AS (
  SELECT
    COUNT(*)::int AS headcount,
    SUM(CASE WHEN Attrition='Yes' THEN 1 ELSE 0 END)::int AS attrition_count,
    ROUND(
      SUM(CASE WHEN Attrition='Yes' THEN 1 ELSE 0 END)::numeric
      / NULLIF(COUNT(*), 0), 4
    ) AS attrition_rate,
    ROUND(AVG(Age)::numeric, 2) AS avg_age
  FROM base
)
SELECT * FROM kpis;
```

```sql
-- Attrition by Age Band
WITH banded AS (
  SELECT
    CASE
      WHEN Age BETWEEN 18 AND 24 THEN '18-24'
      WHEN Age BETWEEN 25 AND 34 THEN '25-34'
      WHEN Age BETWEEN 35 AND 44 THEN '35-44'
      WHEN Age BETWEEN 45 AND 54 THEN '45-54'
      ELSE '55+'
    END AS age_band,
    Attrition
  FROM hr_analytics
)
SELECT
  age_band,
  COUNT(*) AS total,
  SUM(CASE WHEN Attrition='Yes' THEN 1 ELSE 0 END) AS attritions,
  ROUND(
    SUM(CASE WHEN Attrition='Yes' THEN 1 ELSE 0 END)::numeric / NULLIF(COUNT(*),0),
    4
  ) AS attrition_rate
FROM banded
GROUP BY age_band
ORDER BY age_band;
```

```sql
-- Department / Job Role Cuts
SELECT
  Department,
  JobRole,
  COUNT(*) AS total,
  SUM(CASE WHEN Attrition='Yes' THEN 1 ELSE 0 END) AS attritions,
  ROUND(
    SUM(CASE WHEN Attrition='Yes' THEN 1 ELSE 0 END)::numeric / NULLIF(COUNT(*),0),
    4
  ) AS attrition_rate,
  ROUND(AVG(JobSatisfaction)::numeric,2) AS avg_satisfaction,
  ROUND(AVG(MonthlyIncome)::numeric,2) AS avg_income
FROM hr_analytics
GROUP BY Department, JobRole
ORDER BY attritions DESC;
```

---

## **Dashboards**

### **Tableau (Public)**

**Live Dashboard:**
[HR Analytics Dashboard (Tableau)](https://public.tableau.com/views/HR_Analytics_Dashboard_Tableau_17548358462780/HRAnalyticsDashboard?:language=en-GB&:sid=&:display_count=n&:origin=viz_share_link)

**Screenshot (Tableau):** <img width="1295" height="730" alt="Screenshot 2025-08-11 at 22 43 58" src="https://github.com/user-attachments/assets/226634b7-16a7-4786-9bef-4a6efc555728" />

---

### **Excel Dashboard**

`Excel/HR_Analytics_Dashboard.xlsx`

**Screenshot (Excel):** <img width="1164" height="587" alt="Screenshot 2025-08-10 at 12 23 17" src="https://github.com/user-attachments/assets/a9930c6b-67df-49e8-845b-99c1b42eef15" />

---

## **Key Findings (from this dataset)**

* **Attrition by Age Group:** Highest attrition counts in **25–34**, followed by **35–44**.
* **Attrition by Department:** **R\&D** and **Sales** account for the largest attrition volumes.
* **Education & Attrition:** Employees with a **Bachelor’s Degree** show the highest attrition counts.
* **Attrition by Role:** **Laboratory Technician**, **Sales Executive**, and **Research Scientist** are the most impacted roles.
* **Job Satisfaction:** Average satisfaction surfaces as a **leading indicator**—teams/roles with lower averages tend to show **higher attrition rates**.
* **Overall Metrics:** KPIs include **headcount**, **attrition count**, **attrition rate**, and **average age**, exposed via SQL and visualized in Tableau/Excel.

> These insights help HR prioritize interventions (e.g., role-specific retention plans, career pathing for early-career cohorts, and targeted EX improvements).

---

## **How to Reproduce**

1. **Clone & Open**

   * Clone the repository and review the folder structure above.
2. **Load Data**

   * Import `Data/HR_Analytics_Dataset.csv` into PostgreSQL (e.g., table `hr_analytics`).
   * Optionally load `hr_analytics_validation_sample.csv` into a staging table for dry runs.
3. **Run SQL**

   * Execute `Sql/hr_analytics_dashboard_validation_queries.sql`.
   * Export result sets if needed for Excel or directly connect Tableau to PostgreSQL/CSV extracts.
4. **Open Dashboards**

   * **Tableau:** Open the workbook in `Tableau/`, or use the **Public** link above.
   * **Excel:** Open `Excel/HR_Analytics_Dashboard.xlsx` and refresh pivots/slicers.

---

## **Screens & Interactions**

* **Filters:** Age band, Department, Job Role, Tenure band, Income band
* **Drilldowns:** Department → Role → KPIs & Attrition Rate
* **Comparisons:** Satisfaction vs. Attrition; Income vs. Attrition; Tenure vs. Attrition

> Designed so HR partners can quickly **slice** and **compare** risk hotspots and prioritize interventions.
