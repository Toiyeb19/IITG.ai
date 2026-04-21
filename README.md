# IITG.ai

Predictive Paradox: Grid Demand Forecasting
Text Summary Report
This project predicts the next hour's electricity demand (demand_mw at t+1) on the grid using only information available at time t.

In strict adherence to the project constraints,a simple classical model (XGBoost) is applied.
Dataset Description

The model integrates three distinct data sources, aligning multiple frequencies into a unified hourly structure:

    Grid Demand Data (PGCB_date_power_demand.xlsx): Hourly power generation and demand.

    Weather Data (weather_data.xlsx): Hourly environmental factors (Temperature, Apparent Temperature, Humidity, Precipitation).

    Macroeconomic Data (economic_full_1.csv): Annual World Bank indicators (GDP growth, Power consumption per capita).

Exploratory Data Analysis (EDA)

(Visualizing the relationships between weather, time of day, and human consumption patterns.)
1. Demand Volatility and Daily Seasonality

<img width="985" height="328" alt="image" src="https://github.com/user-attachments/assets/4d8a8fca-cff3-46c5-b747-e041a1112a6b" />

    Description: This plot demonstrates the inherent daily peaks in grid consumption and the variance across different seasons.

2. Weather Correlation Heatmap

<img width="777" height="689" alt="image" src="https://github.com/user-attachments/assets/d96584c9-e6d9-40f5-b7c0-0ebac609e14d" />

    Description: Displays the Pearson correlation between environmental factors (like apparent_temp and humidity) and electricity demand.

3. Weekday vs. Weekend Hourly Peaks

<img width="827" height="410" alt="image" src="https://github.com/user-attachments/assets/3d8e9f16-76bd-48b5-8e44-0f8de93b3ed3" />

    Description: Demonstrates that grid demand is not solely based on the time of day, but heavily relies on the interaction between the time of day and the type of day (weekend vs. weekday).

Methodology & Pipeline
1. Anomaly Handling & Data Cleaning

    Spike Detection: Handled severe, undocumented spikes in raw demand using a mathematically sound 3-sigma rolling bound strategy.Any data that was more than 3 standard deviations away from the mean was removed

    Missing Data: Imputed missing values safely, strictly dropping initial NaN rows caused by lag generation rather than backfilling (bfill), which would have introduced lookahead bias.

2. Temporal Feature Engineering

Since classical ML models treat rows independently, the concept of time was engineered tabularly to give the model a "memory" of grid behavior:

    Short-Term Lags: t-1h and t-2h to capture immediate momentum.

    Seasonal Lags: t-24h and t-168h to capture daily and weekly cycles.

    Rolling Aggregates: 3-hour and 24-hour rolling means and standard deviations to teach the model about recent volatility and consumption trends.

    Calendar Encoding: Extracted explicit features like hour, month, and is_weekend.

3. Macroeconomic Integration

Integrated wide-format annual economic data into the long-format hourly dataset. To ensure zero data leakage, the economic indicators were mapped using a lagged year (merge_year = year + 1), ensuring the model only uses the previous year's economic baseline to predict the current year.
4. Leakage-Free Chronological Split

The dataset was sorted chronologically before shifting the target variable (target_demand_next_hour = shift(-1)). The train-test split was performed strictly on a date boundary (Train: Pre-2024, Test: 2024) to simulate real-world grid operations and prevent future data leakage.
Modeling & Results
Algorithm: XGBoost Regressor

Chosen for its robust handling of non-linear relationships (e.g., how extreme heat and extreme cold both cause demand spikes) and its ability to natively categorize continuous features like the hour of the day using Information Gain.

    Hyperparameters: n_estimators=500, learning_rate=0.05, max_depth=6, objective='reg:squarederror'

Evaluation Metric

The model was evaluated on the chronological 2024 hold-out set using Mean Absolute Percentage Error (MAPE).

    Final Test MAPE (2024): ~2.58%

Feature Importance Interpretation

<img width="820" height="544" alt="image" src="https://github.com/user-attachments/assets/37395a54-5fb0-423a-93bd-4224be12841e" />

Key Drivers of Power Demand:

    Recent History: demand_lag_1h and demand_lag_24h act as the strongest baseline predictors. Power demand carries immense momentum.

    Apparent Temperature: The "feels like" temperature proved to be a highly critical external driver, dictating heavy HVAC and air conditioning loads on the grid.

    Time Interactions: The model relied heavily on the hour feature, successfully identifying the evening residential peaks naturally without manual binning.
