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

## 📥 Project Files

| File | Description |
|---|---|
| [`Crime_Analytics.pbix`](./Crime_Analytics.pbix) | Power BI report — open in Power BI Desktop |
| [`Crime_Analytics.pdf`](./Crime_Analytics.pdf) | Static PDF export of all dashboard pages |
| [`ETL/Halifax_Crime.dtsx`](./ETL/Halifax_Crime.dtsx) | SSIS ETL package |
| [`Python/HRM_Clustering.ipynb`](./Python/HRM_Clustering.ipynb) | K-Means clustering notebook |

> 💡 **New to the project?** Start with the PDF for a quick visual overview, then open the `.pbix` file in Power BI Desktop to explore the interactive slicers and tooltips.

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

**Coverage period:** January 14, 2026 – March 2026 (68 days of data)

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

**Key observations from initial exploration:**
- The `OBJECTID` field uses a 1-byte signed integer (`i1`) — this caused a **datatype mismatch** for one record where the value exceeded the 1-byte range, producing the single error record visible in the ETL Operational dashboard
- Some physical locations appear multiple times with slightly different coordinates — 1,028 incidents map to only 795 unique coordinate points and 503 unique street addresses
- The `EVT_RT` field only ever contains `GO` (General Occurrence) — the data model was designed to accommodate future event types, but currently only one type is present

---

### Phase 3 — ETL Pipeline (SSIS + SQL Server)

The ETL pipeline is implemented as a single SSIS package (`Halifax_Crime.dtsx`) running against a SQL Server instance (`HRM_Crime` database).

**Step 1 — Enumerate Crime Data (Foreach Loop Container)**
A Foreach Loop iterates over all CSV files in the source folder, dynamically binding the file path to a package variable (`User::crimevariable`). This allows the pipeline to process multiple collected CSVs in a single execution without any hardcoded file paths.

**Step 2 — Staging (`stg.HalifaxCrime`)**
Each CSV is loaded into the staging table via a Flat File Source → OLE DB Destination data flow. Error rows are redirected rather than failing the pipeline — they land in an error output for review. The staging table preserves raw source data as the landing zone.

**Step 3 — Dimension Loading (`Dims` Sequence Container)**
Three dimension tables are loaded from staging using incremental logic — each uses a `NOT IN (SELECT key FROM dim.Table)` subquery to ensure only new, unseen records are inserted, making the pipeline safe to re-run.

- **`dim.Crime`** — `Crime_ID`, `Crime_Classification_Code`, `Crime_Category`, `Crime_Type`, `Crime_Subcategory`, `Crime_Severity_Level`
- **`dim.Event`** — `Event_ID`, `Event_Reporting_Type` (`GO`), `Event_Type_Description` (`General Occurrence`). Built with future event types in mind via a `lup.Event_ID` lookup table
- **`dim.Location`** — Location keys and street address information

**Step 4 — Fact Table Loading**
`fact.CrimeIncidents` stores one row per reported incident with foreign keys to all dimension tables, date key, coordinate values (x, y), and raw record identifier.

**Step 5 — Reporting Tables (`RPTS` Sequence Container)**
Pre-aggregated tables summarise the fact table by category, location, and severity — an optimised layer for Power BI consumption that reduces query load on the fact table for standard summary visuals.

**Step 6 — Calendar Table (DAX, Power BI)**
Rather than building a date dimension in SQL, a Calendar table spanning ~6 years was generated directly in Power BI using a DAX calculated table, providing full date intelligence (year, month, week, weekday/weekend flags) for all time-based analysis and forecasting.

---

### Phase 4 — Machine Learning (Python / Jupyter)

Geographic clustering was performed in Python to identify spatially coherent crime zones across HRM.

**Libraries:** `pandas`, `numpy`, `matplotlib`, `seaborn`, `plotly`, `scikit-learn`, `kneed`

**Process:**
1. Export from SQL Server → load into Pandas DataFrame
2. Select `x` and `y` coordinate columns as the only features — pure geographic clustering, no crime-type bias
3. Standardise with `StandardScaler` to prevent either axis dominating due to scale differences
4. Compute elbow curve (inertia across k=1–10) with `KneeLocator` from the `kneed` library → elbow confirmed **k=4**
5. Validate cluster quality with **silhouette score**
6. Fit `KMeans(n_clusters=4)` on scaled coordinates
7. Assign cluster labels back to the original DataFrame
8. Export CSV: `Date`, `Crime_Classification_Code`, `Crime_Category`, `location`, `x`, `y`, `GeoCluster`

**Cluster naming (manual, based on HRM geographic knowledge):**

| Cluster Label | Geographic Area | Crime Profile |
|---|---|---|
| Urban Core | Downtown Halifax/Dartmouth peninsula | All 5 crime types, highest density |
| Suburban | Bedford, Clayton Park, suburban Dartmouth | Property crime more prominent |
| Rural North | Fall River, Sackville corridor | Low volume, assault-dominant |
| Remote East | Eastern Passage, Lawrencetown | Isolated incidents, 3 crime types only |

The cluster CSV was imported into Power BI alongside the SQL Server data and modelled relationally for the Geo-Cluster Analysis pages.

---

### Phase 5 — Power BI Reporting

Power BI connects directly to SQL Server for the warehouse data and imports the Python clustering CSV. Relationships were modelled in the data model view, and DAX measures were written for all KPIs, trend calculations, severity percentages, month-over-month comparisons, 7-day moving average, and Q2 projections with confidence bounds.

---

## 📊 Dashboard Summary

> 💡 **Tip:** Open [`Crime_Analytics.pbix`](./Crime_Analytics.pbix) in Power BI Desktop to interact with slicers and tooltips. Use [`Crime_Analytics.pdf`](./Crime_Analytics.pdf) for a quick static walkthrough of all pages.

---

### Page 1 — ETL Operational Reporting

A technical health dashboard for pipeline monitoring. Displays record counts across all key tables and surfaces any error records or lookup validation issues. **This page is for the data engineering team, not end users** — if the numbers here look off, check the pipeline logs before investigating the analytical pages.

**Key metrics:** 1,028 crime records · 12 crime types · 5 categories · 795 unique coordinate points · 503 unique locations · 68 days of data · 1 error record (OBJECTID datatype overflow, redirected and logged)

---

### Page 2 — Summary Overview

The primary executive-facing page. Three KPI cards at the top show Total Crimes, % Change vs Previous Month, and % High Severity. Below, a category bar chart, a monthly trend bar chart, and a top-5 locations bar chart all update dynamically based on the Month and Crime Category slicers on the left panel.

---

#### All Months (Combined View)

![Summary Overview — All Months](./Screenshots/01_summary_overview_all.png)

**1,028 total crimes** across Jan–Mar 2026. Assault dominates at 560 incidents (54.5%) — more than double the next category. Month-over-month growth is sustained and unbroken across all three months. Gloria McCluskey Ave leads all locations at 25 incidents, followed by Gottingen St (23) and Mumford Rd (21) — all urban corridor streets, not residential areas.

| Metric | Value |
|---|---|
| Total Crimes | 1,028 |
| vs Previous Month (Feb) | +30.5% |
| % High Severity | 7.2% |

---

#### January — Filtered View

![Summary Overview — January](./Screenshots/02_summary_overview_january.png)

The lowest-volume month at **242 crimes**. Assault already dominant at 122 (50% of January total). Gloria McCluskey Ave (9) and Gottingen St (8) establish themselves as the top hotspots from the very first month. The "vs Previous Month" KPI is blank — no prior month exists in the dataset, which is correct and expected behaviour.

| Metric | Value |
|---|---|
| Total Crimes | 242 |
| vs Previous Month | — |
| % High Severity | 6.6% |

---

#### February — Filtered View

![Summary Overview — February](./Screenshots/03_summary_overview_february.png)

The **sharpest month-over-month jump** in the dataset at +40.9%, bringing total crimes to 341. Notably, high severity *decreased* to 6.2% despite more overall crimes — more incidents were reported but proportionally fewer were serious. Gloria McCluskey Ave surges to 11 incidents; Spring Garden Rd enters the top 5 for the first time with 6. Mumford Rd drops to just 2, suggesting its January incidents did not persist.

| Metric | Value |
|---|---|
| Total Crimes | 341 |
| vs Previous Month (Jan) | +40.9% |
| % High Severity | 6.2% |

---

#### March — Filtered View

![Summary Overview — March](./Screenshots/04_summary_overview_march.png)

The highest-volume month at **445 crimes** and the most concerning combination in the dataset: both volume (+30.5%) and severity (8.3%) rise simultaneously — the only month where both indicators move in the same direction. Mumford Rd rockets from 2 to 17 incidents, the sharpest single-location surge in the dataset. Gloria McCluskey Ave drops to just 5, suggesting a geographic dispersal of activity away from the traditional downtown hotspots.

| Metric | Value |
|---|---|
| Total Crimes | 445 |
| vs Previous Month (Feb) | +30.5% |
| % High Severity | 8.3% |

---

### Page 3 — Time & Pattern

![Time and Pattern](./Screenshots/05_time_and_pattern.png)

Three visuals examining the temporal structure of crime across the observation window.

**Monthly Trend (top left):** Confirms the consistent linear climb Jan → Feb → Mar. The slope is steady — not a spike but a sustained acceleration, which from a policing perspective is more concerning than a one-time surge because it suggests a structural shift rather than an isolated event.

**Weekday vs Weekend (top right):** Approximately **72% of crimes occur on weekdays** vs 28% on weekends. This is counterintuitive — most people assume weekends carry more risk. The likely explanation is that many property crimes (vehicle theft, break & enter) are *discovered and reported* during business hours on weekdays rather than occurring exclusively then. This has direct implications for patrol resource allocation.

**7-Day Moving Average (bottom):** The richest temporal view in the report. Key data points: 13 (mid-Jan start, partial week) → 132 (late Jan peak) → 44 (trough) → 101 → 51 → 119 → 76 → 128 → 71 → 128 (end of March, rising). Two patterns are visible: a roughly **2–3 week wave cycle** of peaks and troughs, and a **rising floor** across each trough — even the quietest periods are getting busier.

> ⚠️ Sharp dips in the moving average on Jan 24–25, Feb 5–6, Feb 13, and Mar 4/10/16–17 reflect **missing collection days**, not genuine crime reductions. See [Data Limitations](#-data-limitations--known-gaps).

---

### Page 4 — Geo-Cluster Analysis

#### Spatial Distribution & Cluster Boundaries

![Geo-Cluster Spatial Analysis](./Screenshots/06_geo_cluster_spatial.png)

**Left plot — Raw spatial distribution:** All 795 crime coordinate points plotted in projected coordinate space. The density mass collapses into a tight vertical band around the downtown Halifax/Dartmouth peninsula, with sparse outliers extending north and east. Bubble sizes represent incident count per coordinate point — the giant bubbles in the lower-left confirm extreme concentration at specific downtown addresses.

**Right plot — K-Means cluster boundaries:** The same points annotated by the 4 clusters with hand-drawn boundary overlays. The Urban Core and Rural North clusters overlap spatially, reflecting Halifax's compact geography where the urban-to-rural transition is abrupt.

The clusters were identified using **K-Means (k=4) on X/Y coordinates only** — no crime type information was fed into the model. The fact that criminologically distinct profiles emerged from purely geographic grouping confirms that in HRM, **geography alone is a strong predictor of crime type**.

---

#### Crime Category Mix Per GeoCluster

![Category Mix Per GeoCluster](./Screenshots/07_geo_cluster_category_mix.png)

> ⚠️ **The y-axis is logarithmic.** This is an intentional design choice — on a linear scale, the Urban Core's assault count (406) would render all other clusters visually invisible. The log scale enables meaningful cross-cluster comparison without distorting the relative patterns within each cluster.

| Cluster | Assault | Break & Enter | Robbery | Theft from Vehicle | Theft of Vehicle | Total |
|---|---|---|---|---|---|---|
| Urban Core | 406 | 75 | 47 | 189 | 69 | **786** |
| Suburban | 107 | 16 | 2 | 17 | 6 | **148** |
| Rural North | 13 | 3 | 0 | 2 | 3 | **21** |
| Remote East | 6 | 2 | 0 | 0 | 4 | **12** |

Robbery is present only where people and victims concentrate in proximity — it drops to near-zero outside the Urban Core. Assault appears in all 4 clusters, confirming it is not a purely urban phenomenon. Property crimes (Theft from Vehicle, Break & Enter) show suburban creep — growing as a share of the crime mix as you move outward.

---

### Page 5 — Q2 Projection

All three views share the same chart structure: **solid blue area = actual observed data · dashed yellow line = point forecast · olive shaded cone = confidence interval**. The cone widens with time — an honest representation of uncertainty that grows with distance from the observed data.

---

#### Full Dataset Projection (Jan–Jun 2026)

![Q2 Projection — All Data](./Screenshots/08_q2_projection_all.png)

With all months included, the projection extends from the observed plateau (~397–434 in late March/early April) through to June 2026. By June, the **upper bound approaches ~1,400 crimes per month** while the lower bound sits near ~200 — an extremely wide cone reflecting the inherent uncertainty of projecting 3 months ahead from only 68 days of training data. This view is an honest communication of what we do and don't yet know.

---

#### February Baseline Projection

![Q2 Projection — February Baseline](./Screenshots/09_q2_projection_feb_baseline.png)

With only February selected, the projection starts from a lower, more optimistic base. The tooltip at **March 31, 2026** shows: Projected **486** · Upper **560** · Lower **412**. A teammate seeing only this view in February would have projected controlled, moderate Q2 growth — which March's actual data then exceeded.

---

#### March Baseline Projection *(most current and actionable)*

![Q2 Projection — March Baseline](./Screenshots/10_q2_projection_march_baseline.png)

With March selected, actual data shows values in the 348–422 range before the projection takes over. The tooltip at **May 1, 2026** shows: Projected **433** · Upper **482** · Lower **384**. The cone is notably tighter than the full-dataset view, and the relatively flat trajectory suggests a potential **plateauing in the 400–480 crimes/month range** — the most actionable view for current operational planning. The lower bound never drops below March's observed floor, treating March's activity level as the new normal minimum.

---

## 💡 Key Insights

1. **Assault is the dominant crime type in HRM, consistently accounting for ~54% of all reported incidents.** It appears in all 4 spatial clusters, confirming it is not geographically bounded to the urban core.

2. **Crime volume increased every month of the observation window** — January (242) → February (341, +40.9%) → March (445, +30.5%). The growth rate is slowing but absolute volume continues to rise.

3. **March 2026 recorded both the highest crime volume and the highest severity rate (8.3%)** — more crimes AND more serious crimes in the same month is the most concerning combination in the dataset.

4. **72% of reported crimes occur on weekdays.** Resource allocation strategies that over-index on weekend coverage may be misaligned with the actual distribution of reported incidents.

5. **The Urban Core accounts for the majority of every crime category** — 72% of all assaults (406/560), 85% of all theft from vehicle (189/223), and 96% of all robberies (47/49). Geographically targeted intervention in the downtown peninsula would have outsized impact.

6. **Robbery is exclusively an urban phenomenon in HRM** — zero robbery incidents were recorded in Rural North or Remote East clusters across the full observation period.

7. **Mumford Road surged from 2 incidents in February to 17 in March** — the sharpest location-level change in the dataset, potentially signalling geographic displacement of criminal activity.

8. **The top 5 crime locations are all urban corridor streets, not residential areas.** Crime concentrates around high-traffic thoroughfares rather than neighbourhoods.

9. **The 7-day moving average reveals a roughly 2–3 week cyclical pattern with rising troughs** — even the quieter periods are becoming more active, reinforcing the upward trend independently of monthly totals.

10. **K-Means clustering on coordinates alone produced criminologically coherent clusters.** Distinct crime type profiles emerged without any crime-type information fed into the model — confirming that geography is a strong predictor of crime type in HRM.

---

## 🎯 Purpose & Audience

**Primary audience:**
- Data analysts and engineers reviewing this as a portfolio project
- Teammates or collaborators joining the project at any stage
- Public safety researchers interested in HRM crime patterns

**Secondary audience:**
- Municipal stakeholders or policymakers interested in data-driven approaches to local crime
- Fellow international students or newcomers to Halifax wanting an evidence-based picture of the city's safety landscape

**What this project is NOT:**
- This is not a criminology study. No causal claims are made about why crimes occur where they do.
- This is not a real-time system. Data collection is manual and subject to gaps.
- This is not a law enforcement tool. It analyses publicly available reported crime data only.

---

## 🎓 What I Learned

**Technical:**
- Designing a full star schema data warehouse from scratch — staging, lookup, dimension, fact, and reporting layers — and understanding why each layer exists and what problem it solves
- Writing incremental SSIS data flows that are safe to re-run without producing duplicate records
- The practical difference between a datatype defined in a flat file source versus what SQL Server expects — and how to handle it gracefully with error redirection rather than pipeline failure
- Applying `StandardScaler` before K-Means and understanding why unscaled coordinates would bias clustering toward whichever axis has larger absolute values
- Using the elbow method and silhouette scoring together to make a defensible, evidence-based choice of k
- Building DAX measures for moving averages, period-over-period comparisons, and forecast confidence intervals
- Connecting Power BI to multiple data sources (SQL Server + Python CSV) and managing relationships across them in the data model

**Analytical:**
- How to interpret a logarithmic axis and when it is the right choice over a linear scale
- That weekday/weekend crime distributions are not intuitive — reported crime follows reporting behaviour, not just occurrence behaviour
- That spatial clustering without crime-type labels can still produce criminologically meaningful segments
- How to communicate uncertainty honestly in a forecast rather than presenting a single point estimate as fact

**Personal:**
- Starting a project with a personal question — *is this city safe for me?* — and following it through to a rigorous analytical answer is one of the most motivating ways to learn a technical stack
- Manual data collection teaches you to respect data provenance in a way that automated pipelines never do
- Documentation written for a future teammate is the best documentation — it forces a clarity that notes-to-self never require

---

## 🚀 Future Improvements

- **Automate data collection** — Replace manual CSV downloads with a scheduled task or API call hitting the HRM Open Data Hub daily, eliminating collection gaps entirely
- **Move Calendar to SQL** — Migrate the DAX Calendar table into a proper `dim.Date` SQL table populated by a stored procedure, for better performance and portability outside Power BI
- **Add weather data** — Join daily Halifax weather records (temperature, precipitation, daylight hours) to investigate correlations between environmental conditions and crime volume
- **Time-of-day analysis** — The current dataset includes date only. If HRM publishes hourly data, a time dimension would unlock peak-hour analysis
- **Severity scoring model** — Replace the binary High/Other severity classification with a multi-tier scoring model trained on classification codes and historical patterns
- **Predictive modelling** — Move beyond trend forecasting to ARIMA or Prophet as more months of data accumulate
- **Publish to Power BI Service** — Make the report shareable via browser without requiring Power BI Desktop
- **Fix the "vs Previous Month" label** — Update the DAX measure to dynamically reflect whichever month is being compared rather than always displaying "February"
- **Robbery micro-location analysis** — Given robbery is almost entirely concentrated in the Urban Core, proximity analysis could identify specific micro-locations driving the cluster

---

## ⚠️ Data Limitations & Known Gaps

| Limitation | Impact | Mitigation |
|---|---|---|
| 7-day rolling source window | Cannot retroactively fill missed collection days | Missing dates documented in this README |
| Missing collection days | Artificial dips in 7-day moving average | Flagged throughout Time & Pattern section |
| No time-of-day field | Cannot analyse peak crime hours | Flagged as future improvement |
| Single event type (`GO`) | Cannot distinguish sub-event types | Data model built to accommodate future types |
| 1 error record (OBJECTID overflow) | Excluded from analysis | Surfaced in ETL Operational dashboard |
| Coordinate duplication | 1,028 incidents → 795 unique coordinate points | Acknowledged; documented in data understanding |
| Short observation window (68 days) | Wide forecast confidence intervals | Honest cone representation; collection continues |
| No demographic data | Cannot analyse victim or offender profiles | Outside scope; public dataset limitation |

---

## 📁 Repository Structure

```
HRM-Crime-Analytics/
│
├── ETL/
│   └── Halifax_Crime.dtsx               # SSIS package (Visual Studio)
│
├── Python/
│   └── HRM_Clustering.ipynb             # K-Means clustering notebook
│
├── Crime_Analytics.pbix                 # Power BI report file
├── Crime_Analytics.pdf                  # Static dashboard export
│
├── Data/
│   └── *.csv                            # Raw collected CSVs (gitignored if sensitive)
│
├── Exports/
│   └── cluster_output.csv               # Python → Power BI bridge file
│
├── Screenshots/
│   ├── 01_summary_overview_all.png
│   ├── 02_summary_overview_january.png
│   ├── 03_summary_overview_february.png
│   ├── 04_summary_overview_march.png
│   ├── 05_time_and_pattern.png
│   ├── 06_geo_cluster_spatial.png
│   ├── 07_geo_cluster_category_mix.png
│   ├── 08_q2_projection_all.png
│   ├── 09_q2_projection_feb_baseline.png
│   └── 10_q2_projection_march_baseline.png
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

