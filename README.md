AirAware - Milestone 3: Alert Logic & Trend Visualization
=========================================================

Module Overview
---------------
This milestone focuses on implementing Air Quality Index (AQI) calculations,
alert logic, and trend visualization for the AirAware system. The goal is
to provide users with insights on air quality, highlight high-risk days,
and generate visual trend reports to aid decision-making.

Features Implemented
-------------------

1. AQI Calculation Logic
------------------------
- Each pollutant (PM2.5, PM10, NO2, SO2, O3, CO, NH3) has defined thresholds
  based on Indian air quality standards.
- A simple AQI function computes the AQI value for each pollutant.
- The overall AQI is calculated as the maximum of individual pollutant AQIs.

Code Snippet:
-------------
def simple_aqi(val, pol):
    thresholds = {'PM2.5': 60, 'PM10': 100, 'NO2': 80, 'SO2': 80, 'O3': 100,
                  'CO': 10, 'NH3': 400, 'NO': 80, 'NOx': 100}
    return (val / thresholds.get(pol, 100)) * 100

latest_pollutants = latest[pollutants].dropna()
aqi_values = [simple_aqi(v, pol) for pol, v in latest_pollutants.items()]
overall_aqi = int(max(aqi_values))

2. AQI Category and Alerts
--------------------------
- Implemented AQI category mapping based on pollutant concentration.
- Categories: Good ✅, Satisfactory 🙂, Moderate 😐, Poor 😷, Very Poor 🤒, Severe ☠️.
- Alerts are generated when pollutant values exceed defined safe limits.

Code Snippet:
-------------
def aqi_category(pollutant, value):
    if pd.isna(value): return "No Data"
    if pollutant == "PM2.5":
        if value <= 30: return "Good ✅"
        elif value <= 60: return "Satisfactory 🙂"
        elif value <= 90: return "Moderate 😐"
        elif value <= 120: return "Poor 😷"
        elif value <= 250: return "Very Poor 🤒"
        else: return "Severe ☠️"
    # Similar logic for other pollutants

thresholds = {'PM2.5': 60, 'PM10': 100, 'NO2': 80, 'SO2': 80, 'O3': 100}
for pol in pollutants:
    latest_val = latest.get(pol)
    if latest_val and pol in thresholds and latest_val > thresholds[pol]:
        st.warning(f"{pol} exceeds safe limit! ({latest_val})")

3. Visual Trend Reports
-----------------------
- Daily Average Trends: Line plots showing pollutant concentration trends over time.
- Normalized Multi-Pollutant Comparison: All selected pollutants normalized to 0–1 range for comparison.
- Users can select multiple pollutants for dynamic visualization.

Code Snippet:
-------------
daily_avg = city_df.groupby('Date')[pollutants].mean().reset_index()
fig, ax = plt.subplots(figsize=(12,6))
for pol in multi_pollutants:
    ax.plot(daily_avg['Date'], daily_avg[pol], label=pol)
ax.set_title(f"Daily Average Pollutants in {city}")
ax.set_xlabel("Date")
ax.set_ylabel("Concentration")
ax.legend()
st.pyplot(fig)

norm_df = daily_avg.copy()
for pol in multi_pollutants:
    norm_df[pol] = (norm_df[pol] - norm_df[pol].min()) / (norm_df[pol].max() - norm_df[pol].min())

fig, ax = plt.subplots(figsize=(12,6))
for pol in multi_pollutants:
    ax.plot(norm_df['Date'], norm_df[pol], label=pol)
ax.set_title(f"Normalized Daily Trends in {city}")
ax.set_xlabel("Date")
ax.set_ylabel("Normalized Value (0-1)")
ax.legend()
st.pyplot(fig)

Key Components
--------------
1. AQI Calculation – Computes individual pollutant AQI and overall AQI.
2. Alert Logic – Notifies when pollutants exceed safe thresholds.
3. Trend Visualization – Provides line charts for daily average and normalized pollutant trends.

Usage Instructions
------------------
1. Load the cleaned dataset:
   df = pd.read_csv("clean_india_air_quality.csv")

2. Filter for a specific city:
   city_df = df[df['City'] == city]

3. Compute AQI values and display categories:
   aqi_results = {poll: aqi_category(poll, latest.get(poll, None)) for poll in pollutants}
   st.table(pd.DataFrame.from_dict(aqi_results, orient="index", columns=["AQI Category"]))

4. Generate trend plots using Matplotlib and Streamlit:
   st.pyplot(fig)

Outcome of Milestone 3
----------------------
- Users can see real-time AQI categories for all pollutants.
- High-risk pollutants trigger alerts.
- Trend visualizations allow for historical and comparative analysis.
- The foundation is set for forecasting and dashboard integration in Milestone 4.
