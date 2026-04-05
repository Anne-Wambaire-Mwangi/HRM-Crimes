# 🔍 HRM Crime Analytics
### *A full-stack data analytics project on general occurrence crimes in Halifax Regional Municipality, Nova Scotia*

> **Audience:** This README is written for the whole team — whether you're picking this up for the first time or returning after months away, everything you need to understand, reproduce, and extend this project is documented here.

---

## Badges

![SQL Server](https://img.shields.io/badge/SQL%20Server-CC2927?style=for-the-badge&logo=microsoft-sql-server&logoColor=white)
![SSIS](https://img.shields.io/badge/SSIS-Visual%20Studio-5C2D91?style=for-the-badge&logo=visual-studio&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?style=for-the-badge&logo=scikit-learn&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white)
![NumPy](https://img.shields.io/badge/NumPy-013243?style=for-the-badge&logo=numpy&logoColor=white)
![Plotly](https://img.shields.io/badge/Plotly-3F4F75?style=for-the-badge&logo=plotly&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white)
![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=power-bi&logoColor=black)
![Status](https://img.shields.io/badge/Status-Active%20Collection-brightgreen?style=for-the-badge)
![Data](https://img.shields.io/badge/Records-1028%20Incidents-blue?style=for-the-badge)

---

## 📖 Introduction

This project analyses reported general occurrence crimes in the **Halifax Regional Municipality (HRM)**, Nova Scotia, Canada. It was initiated by **Anne Wambaire Mwangi**, an international student at the time with no prior familiarity with Halifax's safety landscape — a perspective that turned curiosity into a structured analytical investigation.

The original motivation was to search for human trafficking data after hearing rumours that Halifax was a hotspot. That data was not publicly available, but the HRM Open Data Hub publishes a rolling 7-day general occurrence crimes dataset. Rather than walk away empty-handed, the decision was made to collect, store, and analyse whatever was available — and to build infrastructure capable of answering real public safety questions as data accumulated.

What started as a personal safety inquiry became a complete data engineering and analytics pipeline spanning **ETL automation, a relational data warehouse, unsupervised machine learning, and an interactive Power BI report**.

---

## 📌 Table of Contents

1. [Introduction](#-introduction)
2. [The Data](#-the-data)
3. [Tech Stack & Architecture](#-tech-stack--architecture)
4. [The Process (How It Was Built)](#-the-process-how-it-was-built)
5. [Dashboard Summary](#-dashboard-summary)
6. [Key Insights](#-key-insights)
7. [Purpose & Audience](#-purpose--audience)
8. [What I Learned](#-what-i-learned)
9. [Future Improvements](#-future-improvements)
10. [Data Limitations & Known Gaps](#-data-limitations--known-gaps)

---

## 📦 The Data

**Source:** [HRM Open Data Hub](https://catalogue.hrm.opendata.arcgis.com/)

The dataset reports **general occurrence crimes** — the broadest category of police-reported incidents in HRM. It is published as a **7-day rolling window**, meaning at any given time only the last 7 days of data are publicly accessible. There is no historical archive.

To build a longitudinal dataset, data was **manually collected every other day** starting January 14, 2026, and loaded through an automated SSIS pipeline into a SQL Server data warehouse.

**Raw source columns:**

| Column | Description |
|---|---|
| `OBJECTID` | Source record identifier |
| `EVT_RT` | Event reporting type code (e.g. `GO` = General Occurrence) |
| `EVT_RIN` | Event record identification number |
| `EVT_DATE` | Date of the reported incident |
| `LOCATION` | Street address or intersection |
| `RUCR` | Crime classification code (numeric) |
| `RUCR_EXT_D` | Crime type description |
| `x` | Longitude coordinate (projected) |
| `y` | Latitude coordinate (projected) |

**Coverage period:** January 14, 2026 – March 2026 (68 days of data across a calendar span of 1,864 days modelled for projections)

**⚠️ Missing collection days** *(data was not collected on these dates due to the 7-day rolling window constraint)*:
- January: Jan 1–13 (pre-project), Jan 24, Jan 25
- February: Feb 5, Feb 6, Feb 13
- March: Mar 4, Mar 10, Mar 16, Mar 17

These gaps should be kept in mind when interpreting trend dips in the 7-day moving average chart. A sharp drop on those dates reflects missing collection, not a genuine crime reduction.

---

## 🛠 Tech Stack & Architecture

```
HRM Open Data Hub (CSV, 7-day rolling)
        │
        ▼
┌─────────────────────────────┐
│   SSIS Package (Visual      │
│   Studio / SQL Server IS)   │
│   Halifax_Crime.dtsx        │
│                             │
│  ┌─────────────────────┐    │
│  │ Enumerate Crime Data│    │  ← Foreach Loop over CSV files
│  │ → Staging Crime Data│    │  ← Flat File Source → stg.HalifaxCrime
│  └─────────────────────┘    │
│  ┌─────────────────────┐    │
│  │ Dims Container      │    │
│  │ → dim.Crime         │    │  ← Incremental load (NOT IN subquery)
│  │ → dim.Event         │    │
│  │ → dim.Location      │    │
│  └─────────────────────┘    │
│  ┌─────────────────────┐    │
│  │ Fact Load           │    │  ← fact.CrimeIncidents
│  └─────────────────────┘    │
│  ┌─────────────────────┐    │
│  │ RPTS Container      │    │  ← Pre-aggregated reporting tables
│  │ → rpt.ByCategory    │    │
│  │ → rpt.ByLocation    │    │
│  │ → rpt.BySeverity    │    │
│  └─────────────────────┘    │
└─────────────────────────────┘
        │
        ▼
┌─────────────────────────┐
│  SQL Server (SSMS)      │
│  Database: HRM_Crime    │
│  Star Schema            │
└─────────────────────────┘
        │
        ├──────────────────────────────────────┐
        ▼                                      ▼
┌──────────────────┐                 ┌──────────────────────┐
│  Python /        │                 │  Power BI            │
│  Jupyter         │                 │  (Direct query from  │
│  Notebook        │                 │   SSMS + Python CSV) │
│                  │                 │                      │
│  K-Means         │                 │  5 Report Pages      │
│  Clustering      │                 │  DAX Measures        │
│  (k=4, X/Y only) │                 │  DAX Calendar Table  │
│                  │                 │  Forecasting         │
│  → GeoCluster    │────────────────▶│                      │
│    CSV export    │                 └──────────────────────┘
└──────────────────┘
```

---

## ⚙️ The Process (How It Was Built)

### Phase 1 — Business Understanding *(CRISP-DM)*

The project began with a real-world question: *Is Halifax safe?* As an international student arriving in a new city, understanding the local crime landscape was both personally relevant and analytically interesting. The absence of human trafficking data redirected focus to the broader general occurrence dataset, which offered sufficient richness to support meaningful analysis.

**Analytical goals defined:**
- Understand the volume, type, and distribution of reported crimes in HRM
- Identify geographic hotspots and spatial clusters
- Detect temporal patterns (monthly trends, weekday vs weekend)
- Project Q2 2026 crime volumes based on observed trends
- Build infrastructure to continue growing the dataset over time

---

### Phase 2 — Data Understanding & Collection

The HRM Open Data Hub exposes crime data as a downloadable CSV, refreshed on a 7-day rolling basis. Because no historical data is retained by the source, a **manual collection protocol** was established: download the CSV approximately every other day, name it consistently, and store it in the pipeline input folder.

Each raw file contains up to 9 fields covering event type, date, location, crime classification code, crime description, and geographic coordinates.

**Key observations from initial exploration:**
- The `OBJECTID` field uses a 1-byte signed integer (`i1`) — this caused a **datatype mismatch** for one record where the value exceeded the 1-byte range, producing the single error record visible in the ETL Operational dashboard
- Some physical locations appear multiple times with slightly different coordinates, meaning 1,028 incidents map to only 795 unique coordinate points and 503 unique street addresses
- The `EVT_RT` field only ever contains `GO` (General Occurrence) — the data model was designed to accommodate future event types, but currently only one type is present

---

### Phase 3 — ETL Pipeline (SSIS + SQL Server)

The ETL pipeline is implemented as a single SSIS package (`Halifax_Crime.dtsx`) running against a SQL Server instance (`HRM_Crime` database). The package runs in sequence:

**Step 1 — Enumerate Crime Data (Foreach Loop Container)**
A Foreach Loop iterates over all CSV files in the source folder, dynamically binding the file path to a package variable (`User::crimevariable`) which is then used by the Flat File Connection Manager. This allows the pipeline to process multiple collected CSVs in a single execution without any hardcoded file paths.

**Step 2 — Staging (`stg.HalifaxCrime`)**
Each CSV is loaded into the staging table via a Flat File Source → OLE DB Destination data flow. Error rows (e.g. the OBJECTID overflow record) are redirected rather than failing the entire pipeline — they land in an error output for review. The staging table preserves raw source data with minimal transformation, acting as the landing zone.

**Step 3 — Dimension Loading (`Dims` Sequence Container)**
Three dimension tables are loaded from staging using incremental logic — each uses a `NOT IN (SELECT key FROM dim.Table)` subquery to ensure only new, unseen records are inserted. This prevents duplicate dimension members across multiple pipeline runs.

- **`dim.Crime`** — Contains `Crime_ID`, `Crime_Classification_Code`, `Crime_Category`, `Crime_Type`, `Crime_Subcategory`, `Crime_Severity_Level`. Severity levels were derived and assigned during the SQL transformation layer based on crime classification codes.
- **`dim.Event`** — Contains `Event_ID`, `Event_Reporting_Type` (`GO`), `Event_Type_Description` (`General Occurrence`). Built with future event types in mind via a `lup.Event_ID` lookup table.
- **`dim.Location`** — Contains location keys and street address information derived from the `LOCATION` field.

**Step 4 — Fact Table Loading**
The fact table (`fact.CrimeIncidents`) stores one row per reported incident, with foreign keys to all dimension tables plus the date key, coordinate values (x, y), and the raw record identifier. This is the central table from which all reporting and analysis draws.

**Step 5 — Reporting Tables (`RPTS` Sequence Container)**
Pre-aggregated reporting tables are populated via OLE DB Source → OLE DB Destination flows using SQL queries that summarise the fact table:
- Crimes by category
- Crimes by location (top locations)
- Crimes by severity

These reporting tables serve as an optimised layer for Power BI consumption, reducing query load on the fact table for standard summary visuals.

**Step 6 — Calendar Table (DAX, Power BI)**
Rather than building a date dimension in SQL, a Calendar table spanning 2,229 days (~6 years) was generated directly in Power BI using a DAX calculated table. This provides full date intelligence (year, month, week, weekday/weekend flags) for all time-based analysis and forecasting.

---

### Phase 4 — Machine Learning (Python / Jupyter)

Geographic clustering was performed in Python to identify spatially coherent crime zones across HRM.

**Libraries used:**
`pandas`, `numpy`, `matplotlib`, `seaborn`, `plotly`, `scikit-learn`, `kneed`

**Process:**

1. Data was exported from SQL Server and loaded into a Pandas DataFrame
2. The `x` and `y` coordinate columns were selected as the only features — the goal was pure geographic clustering, not crime-type-driven segmentation
3. Coordinates were **standardised using `StandardScaler`** before clustering to ensure neither axis dominated due to scale differences
4. An **elbow curve** was computed using inertia values across k=1 to k=10, with `KneeLocator` from the `kneed` library identifying the optimal elbow point
5. The elbow confirmed **k=4** as the appropriate number of clusters
6. **Silhouette score** was computed to validate cluster quality
7. `KMeans(n_clusters=4)` was fitted on the scaled coordinates
8. Cluster labels were assigned back to the original DataFrame and exported as a CSV with columns: `Date`, `Crime_Classification_Code`, `Crime_Category`, `location`, `x`, `y`, `GeoCluster`

**Cluster naming (manual, based on HRM geography):**

| Cluster Label | Geographic Area | Characteristics |
|---|---|---|
| Urban Core | Downtown Halifax/Dartmouth peninsula | Highest density, all 5 crime types present |
| Suburban | Bedford, Clayton Park, suburban Dartmouth | Property crime more prominent |
| Rural North | Fall River, Sackville corridor | Low volume, assault-dominant |
| Remote East | Eastern Passage, Lawrencetown area | Isolated incidents, 3 crime types only |

The cluster CSV was imported into Power BI alongside the SQL Server data and modelled relationally for the Geo-Cluster Analysis pages.

---

### Phase 5 — Power BI Reporting

Power BI connects directly to SQL Server (SSMS) for the warehouse data and imports the Python clustering CSV. Relationships were modelled in the data model view, and DAX measures were written for all KPIs, trend calculations, severity percentages, month-over-month comparisons, the 7-day moving average, and Q2 projections with confidence bounds.

The report contains **5 pages**:

1. ETL Operational Reporting
2. Summary Overview
3. Time & Pattern
4. Geo-Cluster Analysis (2 sub-views)
5. Q2 Projection

---

## 📊 Dashboard Summary

### Page 1 — ETL Operational Reporting
A technical health dashboard for pipeline monitoring. Displays record counts across key tables: total crime records ingested (1,028), distinct crime types (12), crime categories (5), unique coordinate points (795), crime locations (503), event types (1), days of actual data (68), and calendar days modelled (1,864). Also surfaces error records and lookup table validation status. This page is for the data engineering team, not end users.

---

### Page 2 — Summary Overview
The primary executive-facing page. Three KPI cards at the top show **Total Crimes**, **% Change vs Previous Month**, and **% High Severity**. Below, a bar chart breaks crimes down by category across all 5 types, a monthly bar chart shows the Jan–Mar progression, and a horizontal bar chart ranks the **top 5 crime locations** by incident count.

**Slicers:** Month (Jan / Feb / Mar) and Crime Category — all visuals cross-filter dynamically.

| Month | Total Crimes | vs Prev Month | % High Severity |
|---|---|---|---|
| January | 242 | — | 6.6% |
| February | 341 | +40.9% | 6.2% |
| March | 445 | +30.5% | 8.3% |
| **All** | **1,028** | **+30.5%** | **7.2%** |

**Top 5 locations (all months):** Gloria McCluskey Ave (25), Gottingen St (23), Mumford Rd (21), Spring Garden Rd (19), Barrington St (17)

Notable: **Mumford Rd surged from 2 incidents in February to 17 in March** — the most dramatic location-level shift in the dataset, suggesting a geographic spread of activity away from traditional downtown hotspots.

---

### Page 3 — Time & Pattern
Three visuals examining temporal structure:

**Monthly Trend Line** — Confirms a consistent linear climb from Jan to Mar with no sign of plateauing in the observed window.

**Weekday vs Weekend Comparison** — Approximately **72% of crimes occur on weekdays** vs 28% on weekends. This counterintuitive finding likely reflects that many property crimes (vehicle theft, break & enter) are *discovered and reported* during business hours rather than occurring exclusively then.

**7-Day Moving Average** — The most granular temporal view. Peaks and troughs follow a roughly 2–3 week wave cycle. Key values: 13 (mid-Jan start) → 132 (late Jan peak) → 44 (trough, overlaps missing data) → 101 → 51 → 119 → 76 → 128 → 71 → 128 (end of March, rising). The floor of each trough rises over time — even quiet periods are getting busier.

---

### Page 4 — Geo-Cluster Analysis

**Spatial Scatter Plot (left):** All 795 coordinate points plotted in projected coordinate space. The density mass collapses into a tight vertical band around the downtown peninsula, with sparse outliers to the north and east.

**Crime Category within GeoClusters (right):** The same points annotated by the 4 K-Means clusters with boundary overlays. The Urban Core and Rural North clusters overlap spatially, which reflects Halifax's compact geography — the transition from dense urban to semi-rural happens within a short distance.

**Crime Category Mix Per GeoCluster (log scale bar charts):**

> ⚠️ The y-axis is **logarithmic** — this is intentional. On a linear scale, the Urban Core's assault count (406) would render all other clusters visually invisible. The log scale enables meaningful cross-cluster comparison.

| Cluster | Assault | Break & Enter | Robbery | Theft from Vehicle | Theft of Vehicle |
|---|---|---|---|---|---|
| Urban Core | 406 | 75 | 47 | 189 | 69 |
| Suburban | 107 | 16 | 2 | 17 | 6 |
| Rural North | 13 | 3 | 0 | 2 | 3 |
| Remote East | 6 | 2 | 0 | 0 | 4 |

**Key finding:** Robbery is essentially absent outside the Urban Core — it requires victim proximity that rural/suburban geography naturally suppresses. Assault appears in all clusters, confirming it is not purely an urban phenomenon.

---

### Page 5 — Q2 Projection

A Power BI native forecast built using DAX measures. The solid blue area chart shows actual observed data; the dashed yellow line is the point forecast; the olive shaded cone is the confidence interval (upper and lower bounds).

**Three slicer views available:**

- **All data:** Full trajectory Jan–Jun 2026. Upper bound approaches ~1,400 by June — extreme uncertainty at 3-month range with 68 days of training data.
- **February baseline:** Tooltip at Mar 31 → Projected 486, Upper 560, Lower 412. More optimistic trajectory.
- **March baseline (most current):** Tooltip at May 1 → Projected 433, Upper 482, Lower 384. Tighter cone. Suggests a potential **plateauing in the 400–480 range** rather than continued steep growth — the most actionable view for current operational planning.

The widening confidence cone is an honest methodological choice — it does not oversell forecast precision. More collection days will dramatically narrow the bounds.

---

## 💡 Key Insights

1. **Assault is the dominant crime type in HRM, consistently accounting for ~54% of all reported incidents across all months and all geographic areas.** It is the only crime category present in all 4 spatial clusters, confirming it is not geographically bounded to the urban core.

2. **Crime volume increased every month of the observation window** — January (242) → February (341, +40.9%) → March (445, +30.5%). The rate of growth is slowing but the absolute volume continues to rise.

3. **March 2026 recorded both the highest crime volume and the highest severity rate (8.3%)** — more crimes AND more serious crimes in the same month is the most concerning combination in the dataset.

4. **72% of reported crimes occur on weekdays.** Resource allocation strategies that over-index on weekend coverage may be misaligned with the actual distribution of reported incidents.

5. **The Urban Core accounts for the vast majority of every crime category** — 406 of 560 assaults (72%), 189 of 223 theft from vehicle incidents (85%), and 47 of 49 robberies (96%). Geographically targeted intervention in the downtown peninsula would have outsized impact.

6. **Robbery is exclusively an urban phenomenon in HRM** — zero robbery incidents were recorded in the Rural North or Remote East clusters over the full observation period.

7. **Mumford Road surged from 2 incidents in February to 17 in March**, the sharpest location-level change in the dataset. This may indicate geographic displacement of criminal activity — worth monitoring closely in continued data collection.

8. **The top 5 crime locations (Gloria McCluskey Ave, Gottingen St, Mumford Rd, Spring Garden Rd, Barrington St) are all urban corridor streets** — not residential areas. This suggests crime concentrates around high-traffic thoroughfares rather than residential neighbourhoods.

9. **The 7-day moving average reveals a roughly 2–3 week cyclical pattern** with rising troughs over time — meaning even the quieter periods are becoming more active, reinforcing the upward trend.

10. **K-Means clustering on coordinates alone produced criminologically coherent clusters** — distinct crime type profiles emerged from geographic grouping without any crime-type information fed into the model. This confirms that in HRM, geography is a strong predictor of crime type.

---

## 🎯 Purpose & Audience

**Primary audience:**
- Data analysts and engineers reviewing this as a portfolio project
- Teammates or collaborators joining the project at any stage
- Public safety researchers interested in HRM crime patterns

**Secondary audience:**
- Municipal stakeholders or policymakers interested in data-driven approaches to understanding local crime
- Fellow international students or newcomers to Halifax who want an evidence-based picture of the city's safety landscape

**What this project is NOT:**
- This is not a criminology study. No causal claims are made about why crimes occur where they do.
- This is not a real-time system. Data collection is manual and subject to gaps.
- This is not a law enforcement tool. It analyses publicly available reported crime data only.

---

## 🎓 What I Learned

**Technical:**
- Designing and implementing a full star schema data warehouse from scratch — staging, lookup, dimension, fact, and reporting layers — and understanding why each layer exists and what problem it solves
- Writing incremental SSIS data flows that are safe to re-run without producing duplicate records
- The practical difference between a datatype defined in a flat file source versus what SQL Server expects — and how to handle it gracefully with error redirection rather than pipeline failure
- Applying `StandardScaler` before K-Means and understanding why unscaled coordinates would bias the clustering toward whichever axis has larger absolute values
- Using the elbow method and silhouette scoring together to make a defensible choice of k — rather than guessing
- Building DAX measures for moving averages, period-over-period comparisons, and forecast confidence intervals — and understanding the difference between calculated columns and measures
- Connecting Power BI to multiple data sources (SQL Server + Python CSV) and managing relationships across them

**Analytical:**
- How to interpret a logarithmic axis and when it is the right choice versus a linear scale
- That weekday/weekend crime distributions are not intuitive — reported crime follows reporting behaviour, not just occurrence behaviour
- That spatial clustering without crime-type labels can still produce criminologically meaningful segments — the geography tells the story
- How to communicate uncertainty honestly in a forecast rather than presenting a single point estimate as fact

**Personal:**
- Starting a project with a personal question — *is this city safe for me?* — and following it through to a rigorous analytical answer is one of the most motivating ways to learn a technical stack
- Manual data collection is tedious but teaches you to respect data provenance and understand exactly what your pipeline is ingesting
- Documentation written for a future teammate is the best documentation — it forces clarity that "notes to self" never require

---

## 🚀 Future Improvements

- **Automate data collection** — Replace manual CSV downloads with a scheduled task or web scraper that hits the HRM Open Data Hub API on a daily cadence, eliminating collection gaps entirely
- **Expand the date dimension** — Move the Calendar table from DAX into the SQL Server warehouse as a proper `dim.Date` table, populated via a SQL stored procedure, for better performance and portability
- **Add weather data** — Join daily Halifax weather records (temperature, precipitation, daylight hours) to investigate correlations between environmental conditions and crime volume
- **Time-of-day analysis** — The current dataset does not include time of day, only date. If HRM ever publishes hourly data, a time dimension would unlock peak-hour analysis
- **Severity scoring model** — Replace the binary High/Other severity classification with a multi-tier scoring model trained on crime classification codes and historical severity patterns
- **Predictive modelling** — Move beyond trend forecasting to a supervised regression or time-series model (ARIMA, Prophet) trained on the growing dataset as more months of data accumulate
- **Interactive web dashboard** — Publish the Power BI report to Power BI Service for shareable, browser-accessible reporting, or rebuild key visuals in a Python Dash or Streamlit app for full open-source portability
- **Robbery network analysis** — Given that robbery is concentrated almost entirely in the Urban Core, graph-based analysis of robbery incident proximity could identify specific micro-locations driving the cluster
- **Fix the "vs Previous Month" label** — Update the DAX measure so the KPI card dynamically reflects whichever month is being compared, rather than always displaying "February"

---

## ⚠️ Data Limitations & Known Gaps

| Limitation | Impact | Mitigation |
|---|---|---|
| 7-day rolling source window | Cannot retroactively fill missed collection days | Document missing dates; flag affected moving average dips |
| Manual collection cadence | Gaps in Jan 24–25, Feb 5–6, Feb 13, Mar 4, Mar 10, Mar 16–17 | Noted in this README; visible as artificial dips in 7-day MA |
| No time-of-day field | Cannot analyse peak crime hours | Flagged as future improvement |
| Single event type (`GO`) | Cannot distinguish between sub-event types | Data model built to accommodate future types |
| 1 error record (OBJECTID overflow) | Excluded from analysis | Surfaced in ETL Operational dashboard |
| Coordinate duplication | 1,028 incidents → 795 unique points | Acknowledged; some locations cluster multiple incidents |
| Short observation window (68 days) | Wide forecast confidence intervals | Honest communication of uncertainty; collection continues |
| No demographic data | Cannot analyse victim or offender profiles | Outside scope; public dataset limitation |

---

## 📁 Repository Structure

```
HRM-Crime-Analytics/
│
├── ETL/
│   └── Halifax_Crime.dtsx          # SSIS package (Visual Studio)
│
├── Python/
│   └── HRM_Clustering.ipynb        # K-Means clustering notebook
│
├── PowerBI/
│   └── Crime_Analytics.pbix        # Power BI report file
│
├── Data/
│   └── *.csv                       # Raw collected CSVs (gitignored if sensitive)
│
├── Exports/
│   └── cluster_output.csv          # Python → Power BI bridge file
│
├── Screenshots/
│   ├── etl_operational.png
│   ├── summary_overview.png
│   ├── time_and_pattern.png
│   ├── geo_cluster_analysis.png
│   └── q2_projection.png
│
└── README.md
```

---

## 👩🏾‍💻 Author

**Anne Wambaire Mwangi**
GitHub: [@Anne-Wambaire-Mwangi](https://github.com/Anne-Wambaire-Mwangi)

*International student, data analyst, and the person who turned "is Halifax safe?" into a full data engineering project.*

---

## 📄 License

This project uses publicly available data from the [HRM Open Data Hub](https://catalogue.hrm.opendata.arcgis.com/). All analysis, code, and documentation are original work by Anne Wambaire Mwangi.

---

*Last updated: April 2026 | Data collection ongoing*
