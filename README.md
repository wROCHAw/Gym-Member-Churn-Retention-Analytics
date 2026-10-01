# Gym Member Churn & Retention Analytics

## Executive Summary
This project delivers an end-to-end analytics solution to identify the root causes of member churn in a gym setting. By combining data modeling in **MySQL** with advanced visualization and DAX measures in **Power BI**, the analysis uncovered a critical retention bottleneck between **months 3 and 6**, directly linked to **weight loss progress stagnation** and **flexible monthly subscriptions**.

---

## Key Business Insights

* **Peak Churn Window (3–6 Months):** The highest churn rate occurs between the 3rd and 6th month of tenure (~28% churn rate), indicating a sharp decline in motivation after the initial 90 days.
* **Weight Loss Stagnation:** A scatter/trend analysis revealed that members experience a performance plateau in weight loss during this exact 3–6 month window, driving cancellation decisions.
* **Subscription Vulnerability:** Members on **Monthly plans** show significantly higher cancellation rates compared to Quarterly and Yearly subscribers due to low commitment/exit barriers.

---

## Tech Stack & Methodology

* **Database & ETL:** MySQL Workbench
  * Created clean SQL views using **CTEs** and `CASE WHEN` statements to segment tenure ranges dynamically.
  * Computed active tenure using `DATEDIFF` and handling null dates with `COALESCE`.
* **Data Modeling & DAX:** Power BI Desktop
  * Developed measures for **High Risk Members** (`CALCULATE`, `COUNTROWS`) and **High Risk Rate %** (`DIVIDE`).
  * Configured dynamic Gauge targets and custom slicers for cross-filtering.
* **Data Visualization:** High-contrast Dark Mode layout organized into a structured logical narrative (KPI Cards $\rightarrow$ Core Drivers $\rightarrow$ Granular Trends).

---

## SQL View Architecture

```sql
CREATE OR REPLACE VIEW vw_gym_churn_tenure_ranges AS
WITH base_data AS (
    SELECT 
        Member_ID,
        Gender,
        Weight_Loss,
        Churn,
        DATEDIFF(
            COALESCE(Cancel_Date, CURRENT_DATE), 
            Join_Date
        ) / 30.0 AS tenure_months
    FROM gym_churn
    WHERE Join_Date IS NOT NULL 
      AND TRIM(Join_Date) != ''
)
SELECT 
    Member_ID,
    Gender,
    Weight_Loss,
    Churn,
    tenure_months,
    CASE 
        WHEN tenure_months < 1 THEN '1. < 1 Month'
        WHEN tenure_months BETWEEN 1 AND 3 THEN '2. 1–3 Months'
        WHEN tenure_months BETWEEN 3.0001 AND 6 THEN '3. 3–6 Months'
        WHEN tenure_months BETWEEN 6.0001 AND 12 THEN '4. 6–12 Months'
        ELSE '5. > 12 Months'
    END AS tenure_range
FROM base_data;
```

---

##  Strategic Action Plan & Business Recommendations

1. **Proactive 90-Day Reassessment Program (Operational):** Implement an automated alert trigger for a mandatory fitness check-up at day 90 to re-evaluate workouts and nutrition before performance plateaus.
2. **Plan Migration Campaigns (Commercial):** Target active monthly members at month 2 with exclusive incentives to upgrade to long-term (Quarterly/Yearly) contracts.
3. **Member Educational Content & Progress Tracking:** Inform members about the common 90-day weight plateau, explaining that stalled scale weight often reflects muscle gain rather than a lack of body composition progress. Offer complimentary bioimpedance analysis as an incentive for plan upgrades, helping members visually track this subtle physical transformation.
---

## Repository Structure
```text
├── sql/
│   └── vw_gym_churn_tenure_ranges.sql
├── powerbi/
│   └── Gym_Churn_Analytics.pbix
├── images/
│   └── dashboard_preview.png
└── README.md
```
