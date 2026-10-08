# Teen Mental Health and Social Media Analytics

![Cover](/cover_banner.jpg)



##  Introduction

This project analyzes a dataset of **1,200 teenagers (ages 13–19)** to explore the relationship between social media behavior and mental health outcomes — including stress, anxiety, sleep, addiction, academic performance, and depression indicators.

The goal was to turn a raw survey-style dataset into a two-page, decision-ready Power BI report that a parent, school counselor, or youth-wellness program could actually use - not just a collection of charts, but a report built around specific questions worth answering.

---

##  Tools Used

| Tool | Purpose |
|---|---|
| **Excel** | Initial data inspection |
| **Power Query (Power BI)** | Data cleaning, text standardization, new column creation |
| **Power BI Desktop** | Data modeling, DAX measures, visuals, report design |
| **DAX** | Calculated columns, KPI measures, dynamic titles |
| **HTML Content (custom visual)** | Styled, icon-based KPI cards |
| **Chiclet Slicer (custom visual)** | Image-based platform slicer (Instagram / TikTok / Both) |

---

##  Business / Decision Questions

The report was designed to answer:

1. Does daily social media use correlate with depression, stress, or anxiety in teens?
2. Which platform (Instagram, TikTok, Both) is associated with the highest addiction and stress levels?
3. Does screen time before sleep reduce sleep duration, and does that relate to mental health outcomes?
4. Is there a relationship between physical activity / social interaction level and lower stress or depression risk?
5. How does academic performance vary with social media addiction level?
6. Are older or younger teens more at risk (age-group breakdown of stress/anxiety/depression)?
7. Do male and female teens differ in social media addiction, stress, or sleep patterns?
8. What proportion of teens fall into "high risk" (high stress + high anxiety) and what characterizes them?

---

##  Data Cleaning Process

The dataset arrived largely clean (no nulls, no duplicate rows), but a few steps were still needed in Power Query before modeling:

- **Enforced data types** - whole numbers for age/levels, decimals for hours
- **Standardized text casing** - `gender` and `social_interaction_level` were lowercase while `platform_usage` was Title Case; all three were normalized with `Text.Proper`
- **Flagged the class imbalance** - only 31 of 1,200 teens (2.6%) are flagged `depression_label = 1`; this is carried through the report as a visible caveat rather than cleaned away, since it's a real feature of the data, not an error

---

##  New Columns Created (Power Query / DAX)

| Column | Logic |
|---|---|
| **Age Group** | 13–15 → Early Teen, 16–17 → Mid Teen, 18–19 → Late Teen |
| **Risk Segment** | Stress ≥7 & Anxiety ≥7 → High Risk; either ≥4 → Moderate Risk; else → Low Risk |
| **Sleep Category** | <6h → Sleep Deficit, 6–8h → Adequate, 8h+ → Long Sleeper |
| **Usage Tier** | <3h → Light, 3–6h → Moderate, 6h+ → Heavy |
| **Social Media Band** | <3h / 3–6h / 6h+ buckets for charting |
| **Stress Band** | Low (1–3) / Medium (4–6) / High (7–10) |
| **Activity Level** | Physical activity score (0–2) mapped to Low / Medium / High |
| **Academic Band** | Academic performance score mapped to Below Average / Average / Above Average |

Example:
```DAX
Risk Segment = 
SWITCH(
    TRUE(),
    'Teen_Mental_Health'[stress_level] >= 7 && 'Teen_Mental_Health'[anxiety_level] >= 7, "High Risk",
    'Teen_Mental_Health'[stress_level] >= 4 || 'Teen_Mental_Health'[anxiety_level] >= 4, "Moderate Risk",
    "Low Risk"
)
```

---

##  DAX Measures

Core measures built to power the KPIs and charts:

- `Total Teens`, `Avg Sleep Hours`, `Avg Social Media Hours`, `Avg Stress Level`, `Avg Anxiety Level`, `Avg Addiction Level`, `Avg Academic Performance`, `Avg Screen Time Before Sleep`
- `Depression Rate %`, `High Risk %`, `Moderate Risk %`, `Low Risk %`, `Sleep Deficit %`, `Heavy Users %`
- `High Risk Teens`, `Heavy Users (6+ hrs)`

Example:
```DAX
Depression Rate % = 
DIVIDE(
    SUM('Teen_Mental_Health'[depression_label]),
    COUNTROWS('Teen_Mental_Health')
)
```

---

##  HTML-Rendered KPI Cards

All KPI cards use the **HTML Content** custom visual so each tile can carry its own icon, color, and typography instead of Power BI's default card style. Each measure returns a complete HTML string, e.g.:

```DAX
KPI Card - Avg Sleep = 
"<div style='background:#FFFFFF;border-radius:14px;padding:18px;font-family:Segoe UI,sans-serif;box-shadow:0 2px 8px rgba(93,125,163,0.15);text-align:center;'>" &
  "<div style='width:46px;height:46px;border-radius:50%;background:#EAF0FA;display:flex;align-items:center;justify-content:center;font-size:22px;margin:0 auto 10px;'>🌙</div>" &
  "<div style='font-size:34px;font-weight:800;color:#5D7DA3;line-height:1.1;'>" & FORMAT([Avg Sleep Hours],"0.0") & "<span style='font-size:16px;font-weight:600;'>h</span></div>" &
  "<div style='font-size:14px;font-weight:700;color:#5D7DA3;opacity:0.85;margin-top:6px;'>AVG SLEEP HOURS</div>" &
"</div>"
```

Design system: white card background, `#5D7DA3` text, soft drop shadow, pastel icon badge per card, 34px bold value / 14px bold label for readability at a glance.

**Dynamic page titles** were built the same way, reflecting whatever Age Group / Gender / Platform filters are currently applied:
```DAX
Dynamic Title - Wellbeing = 
VAR SelAge = IF(ISFILTERED('Teen_Mental_Health'[age_group]), SELECTEDVALUE('Teen_Mental_Health'[age_group], "All Ages"), "All Ages")
VAR SelGender = IF(ISFILTERED('Teen_Mental_Health'[gender]), SELECTEDVALUE('Teen_Mental_Health'[gender], "All Genders"), "All Genders")
VAR SelPlatform = IF(ISFILTERED('Teen_Mental_Health'[platform_usage]), SELECTEDVALUE('Teen_Mental_Health'[platform_usage], "All Platforms"), "All Platforms")
RETURN
"<div style='background:#FFFFFF;padding:14px 20px;font-family:Segoe UI,sans-serif;'>" &
  "<div style='font-size:30px;font-weight:800;color:#5D7DA3;'>Digital Wellbeing &amp; Behavior Overview</div>" &
  "<div style='font-size:15px;font-weight:600;color:#5D7DA3;opacity:0.75;margin-top:4px;'>" & SelAge & " | " & SelGender & " | " & SelPlatform & " · " & FORMAT([Total Teens],"#,0") & " teens shown</div>" &
"</div>"
```

---

##  Image Slicer

The platform filter (Instagram / TikTok / Both) uses the **Chiclet Slicer** custom visual with platform logos instead of a text list, built from a small image-mapping table related to `platform_usage`.

---

## Digital Wellbeing & Behaviour Overview

![Dashboard 1 - Digital Wellbeing & Behaviour Overview](/dashboard1_wellbeing_overview.png)

---
##  Mental Health Risk & Correlation

![Dashboard 2 - Mental Health Risk & Correlation](/dashboard2_risk_correlation.png)

---

##  Key Insights

- **Sleep is the strongest protective factor** in the dataset - teens with lower sleep hours show the clearest association with elevated depression indicators.
- **Stress, anxiety, and daily social media hours** are the three variables most positively associated with depression flags, though all correlations are weak-to-moderate, not strong.
- **40% of teens are sleeping under 6 hours** a night, regardless of platform used.
- **~75% of teens fall into the Moderate Risk segment**, with roughly 15.5% High Risk and the remainder Low Risk - suggesting most of this population sits in a "watch" zone rather than clearly safe or clearly critical.
- Depression-flagged cases are **too few (31 of 1,200, ~2.6%)** to generalize confidently - the dashboard surfaces this directly rather than overstating the finding.

---

##  Recommendations

1. **Prioritize sleep hygiene interventions** over blanket screen-time bans - sleep duration showed a clearer relationship with depression indicators than raw social media hours did.
2. **Target the Moderate Risk segment**, not just High Risk - it's the largest group and the one most likely to shift with early intervention.
3. **Treat screen time before bed as a specific checkpoint**, separate from total daily usage - the data suggests timing matters, not just volume.
4. **Use the Depression Rate figures cautiously** in any report-out - the sample of flagged cases is small, and messaging should reflect that rather than presenting it as a firm population estimate.
5. **Expand the dataset** in future iterations to include more depression-flagged cases, which would allow firmer conclusions and potentially predictive modeling.

---

##  Limitations

- Severe class imbalance in `depression_label` (2.6% positive cases) limits statistical confidence in any depression-specific finding.
- `academic_performance` and `physical_activity` are provided as coded numeric scales (2–4 and 0–2) rather than raw units — exact scale definitions were not provided with the dataset.
- Dataset is cross-sectional (a single snapshot), so all relationships described are correlational, not causal.

---

## Here is the Link to assess the full project on PowerBI Service
https://app.powerbi.com/view?r=eyJrIjoiNjkwOGU2OTYtYTdhMi00YTBkLTg5MGEtYmJkZDRlNWEyMzU5IiwidCI6IjQ4NTkyZTczLTE2OTUtNGVmMy1hYzg3LWM0ZDNjMGVhNDYzMyJ9
