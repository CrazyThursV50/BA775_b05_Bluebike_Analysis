# **Bluebikes Demand Forecasting with Weather and MBTA Proximity for Operational Planning**

Team B05: Mohamad Gong, Marcus Shi, Tianqi Sun, Tzu-Jen Chen, Arshdeep Oberoi, Xiaoqing Ye

This repository contains the code and documentation for Team B05’s BA775 project  
**“Bluebikes Demand Forecasting with Weather and MBTA Proximity for Operational Planning.”**  
All analysis is implemented in BigQuery and a single Jupyter notebook.

---

## **1. Project Overview**

Bluebikes has become a high-volume component of Boston’s mobility network.  
Our goal is to give Bluebikes stakeholders a repeatable, data-driven view of where and when to focus docks, bikes, and staff by:

- Profiling system and station-level usage across time, user type, and geography  
- Quantifying how climate conditions influence daily ridership  
- Forecasting “high-demand” days for the upcoming month  
- Assessing how proximity to MBTA rapid-transit stations supports first- and last-mile connectivity  

The notebook follows the structure:

1. Data Model and Dashboard Overview  
2. Data Sources  
3. Data Importing and Cleaning  
4. Analysis and Findings  
5. Conclusions, Risks, and Next Steps  
6. Challenges, References, and Appendix  

---

## **2. Data Model and Tools**

Most data engineering and modeling is done directly in **Google BigQuery**, then surfaced into:

- A **core data model** for trips, stations, climate, and MBTA proximity  
- Aggregated tables for Looker Studio dashboards (for example  
  `dashboard.dashboard_page1_network_trips`,  
  `dashboard.dashboard_page2_daily_demand`)  
- BigQuery ML tables for daily high-demand classification and prediction  

Key technologies:

- **BigQuery SQL** for cleaning, feature engineering, and aggregations  
- **BigQuery ML logistic regression** model  
  `results.daily_high_demand_lag1_lg` for high-demand prediction  
- **Jupyter / Colab** notebook to orchestrate and document the workflow  

---

## **3. Data Sources**

The notebook integrates four main sources:

1. **Bluebikes trips**  
   - `ba775-fall25-b05.bluebikes.trips`  
   - Raw ride-level data (start/end times, station ids, duration, bike type, user type)  

2. **Bluebikes stations**  
   - `ba775-fall25-b05.bluebikes.stations`  
   - Station metadata and coordinates used to build `trips_clean` and the station–MBTA proximity table  

3. **Climate / weather data**  
   - Daily NOAA–style climate table transformed into  
     `ba775_fall25_b05.weather_daily_summary`  
   - Includes average temperature, precipitation, snowfall, snow depth, and wind measures  

4. **MBTA rapid-transit stations**  
   - `mbta.mbta_stations` plus a derived table  
     `results.stations_mbta_proximity`  
   - Stores nearest MBTA stop, line, access segment, and distance in meters for each Bluebikes station  

---

## **4. Data Importing and Cleaning**

The notebook creates a series of clean, analysis-ready tables, including:

- **`ba775_fall25_b05.trips_clean`**  
  - Cleans raw trips, standardizes station ids, derives trip duration in seconds, and flags member vs casual users.  

- **`ba775_fall25_b05.weather_daily_summary`**  
  - Casts string climate fields to numeric, builds daily temperature and hazard metrics.  

- **`results.daily_trips_temp_hazard`**  
  - Joins daily ridership with weather to bucket days into temperature ranges  
    (Below freezing / Cold / Mild / Warm / Hot) and hazard vs no-hazard days.  

- **`results.stations_mbta_proximity`**  
  - Computes straight-line distance from each Bluebikes station to its nearest MBTA rapid-transit stop and classifies access segments such as “walkable_to_mbta”.  

- **`results.daily_features_lag1`**  
  - Builds modeling features like daily trip counts, previous-day (“lag 1”) trips, temperature, day-of-week, month, and weekend flag.  

These tables feed both the analysis sections and the dashboard datasets.

---

## **5. Analysis and Findings**

### **4.1 Sizing and Segmenting Bluebikes Demand**

We size demand across:

- Date, hour of day, and day of week  
- Member vs casual riders  
- Active start stations  

The table `dashboard.dashboard_page2_daily_demand` supports daily KPIs such as:

- Total trips and member share  
- Average trip duration in minutes  
- Number of active start stations  
- Distribution across weather buckets and hazard flags  

### **4.2 Quantifying Climate Sensitivity of Ridership**

Using `results.daily_trips_temp_hazard` and related tables, we:

- Compare average ridership across temperature buckets  
  (Below freezing, Cold, Mild, Warm, Hot)  
- Contrast hazard vs no-hazard days  
- Visualize how extreme cold and hazardous conditions sharply depress demand, while mild and warm days lift ridership above median levels  

### **4.3 High-Demand Outlook for the Next Month**

We define **high-demand days** as those with total trips above the median, then build a BigQuery ML logistic regression model:

- Model: `results.daily_high_demand_lag1_lg`  
- Features:

  - `trips_lag_1` (previous-day trip count)  
  - Temperature (`temp`)  
  - Day of week (`dow`)  
  - Month  
  - Weekend indicator (`is_weekend`)  

The model is trained with `data_split_method = 'NO_SPLIT'` on `results.daily_features_lag1` and used to generate:

- **`results.daily_high_demand_october_predictions`**  
  - For each date in the forecast horizon, the model outputs:  
    - `predicted_high_demand` (0/1)  
    - `predicted_high_demand_prob` (probability of high demand)  

This supports a practical “high-demand calendar” for the upcoming month.

### **4.4 Assessing MBTA Proximity and First- and Last-Mile Opportunities**

Using `results.stations_mbta_proximity` and aggregated trip data, we:

- Classify stations into MBTA access segments such as “walkable_to_mbta”.  
- Build `dashboard.dashboard_page1_network_trips` that labels trips as:  

  - First mile to MBTA  
  - Last mile from MBTA  
  - MBTA to MBTA  
  - Not MBTA linked  

- Quantify the share of trips that connect to the MBTA network and identify corridors where Bluebikes is heavily used as a first- and last-mile connector.  

These outputs are used in a network view on the dashboard to highlight important transit-adjacent clusters.

---

## **6. Dashboards**

The cleaned and aggregated tables back a set of Looker Studio dashboard pages (as specified in the notebook):

1. **Network and MBTA Connectivity View**  
   - Station-level map, MBTA access segments, and first/last-mile trip roles.  

2. **Daily Demand and Climate View**  
   - Time series of daily trips, member vs casual mix, temperature and hazard overlays, and high-demand labels.  

3. **High-Demand Forecast View**  
   - Predicted high-demand days and probabilities for the forecast horizon.  

---

## **7. How to Run**

This repository currently includes the main analysis notebook:

```text
├── B05-Bluebikes-Demand-Forecasting-with-Weather-and-MBTA-Proximity-for-Operational-Planning.ipynb
└── README.md
