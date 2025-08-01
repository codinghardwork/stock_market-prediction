
*   **Project Goal**: To **predict tomorrow's S&P 500 index price** (specifically, whether it will go up or down) using machine learning and Python, and to **backtest the model** on over 20 years of historical data to ensure confidence in its predictions.
*   **Tools Used**: The project utilizes **Jupyter Lab** (or Jupyter Notebook) and the **`yfinance` package** to download daily stock and index prices from the Yahoo Finance API.
*   **Data Acquisition**:
    *   The **S&P 500 index is represented by the `^GSPC` symbol**.
    *   Historical data is queried using the `history(period='max')` method of a Ticker class object.
    *   The downloaded data is a **Pandas DataFrame** where each row is a single trading day.
    *   Key columns include **Open, High, Low, Close, and Volume**.
    *   The DataFrame uses a **DateTime index** for easy indexing and slicing.
*   **Data Cleaning and Preprocessing**:
    *   **Irrelevant columns ('Dividends', 'Stock Splits') are removed** as they are more appropriate for individual stocks than an index.
    *   **Defining the Target**: The model aims to predict **directionality (price going up or down tomorrow)**, not the absolute price, because directional accuracy is crucial for trading profitability.
        *   A 'tomorrow' column is created by shifting the 'Close' price back one day.
        *   The 'target' column is set to **1 if 'tomorrow's price is greater than today's price (price goes up), and 0 otherwise** (converted to integer type for machine learning).
    *   **Data Filtering**: Data before January 1, 1990, is removed because older data might not be useful for future predictions due to fundamental market shifts. A `.copy()` method is used to avoid Pandas warnings when subsetting and assigning data.
*   **Initial Machine Learning Model**:
    *   A **Random Forest Classifier** is chosen as the default model for its **resistance to overfitting**, relatively fast execution, and ability to **capture non-linear relationships** in the data.
    *   Model parameters include:
        *   `n_estimators`: Number of decision trees (initially set low for speed, higher values can improve accuracy).
        *   `min_samples_split`: Helps prevent overfitting by controlling tree depth.
        *   `random_state`: Ensures **reproducible results** for consistency.
*   **Train/Test Split for Time Series Data**:
    *   **Standard cross-validation is avoided** because it can lead to **"leakage"** (using future data to predict the past), resulting in overly optimistic but unrealistic model performance.
    *   Initially, the **last 100 rows are used as the test set**, and the remaining historical data forms the training set, preserving the time series order.
    *   **Predictors** for the initial model are 'Close', 'Volume', 'Open', 'High', and 'Low'.
*   **Model Evaluation (Initial)**:
    *   The **precision score** is used as the primary accuracy metric, indicating **"what percentage of the time, when we said the market would go up, did it actually go up?"**. This is particularly relevant for buying stock.
    *   Initial model performance: **42% precision score**, which is not good and suggests trading against the model would be better.
*   **Robust Backtesting**:
    *   To build confidence, a **more robust backtesting approach is implemented** to test the algorithm across multiple years of data.
    *   A `predict_function` wraps the model fitting and prediction logic.
    *   A `backtest_function` simulates real-world trading by:
        *   Setting a `start` value (e.g., 2500 trading days for 10 years of initial training data) and a `step` value (e.g., 250 days for one year).
        *   **Iteratively trains the model on an increasing historical dataset** and makes predictions for the subsequent period (e.g., train on years 1-10, predict for year 11; then train on years 1-11, predict for year 12, and so on).
    *   Backtested precision score: **53% accurate** across about 6,000 trading days.
    *   **Benchmark**: The S&P 500 naturally went up 53.6% of the days, meaning the initial backtested model was slightly worse than just always predicting "up".
*   **Model Improvement with New Predictors**:
    *   **Rolling averages (Close Price Ratios)** are added, comparing today's closing price to the average closing price over various horizons (2, 5, 60, 250, 1000 trading days). These help identify if the market is due for a downturn or upswing.
    *   **Rolling trends** are added, representing the number of days the stock price went up within a given past horizon.
    *   `NaN` values, which occur when there isn't enough historical data to compute rolling averages/sums, are dropped, effectively starting the data from a later date (e.g., 1993).
    *   **Model parameters are updated**: `n_estimators` increased to 200, `min_sample_split` reduced to 50.
    *   **Prediction method changed to `predict_proba`**: This returns probabilities (0-down, 1-up) instead of just 0 or 1, allowing for more control.
    *   A **custom prediction threshold is set to 60%** (instead of the default 50%). This means the model only predicts "up" if it's at least 60% confident, **reducing the number of trades but aiming for higher accuracy on those trades**.
    *   **Absolute price predictors (Close, Open, High, Low, Volume) are removed** because ratio-based predictors are more informative.
*   **Improved Model Evaluation**:
    *   With the new predictors and threshold, the model made **fewer "up" predictions** (due to higher confidence requirement).
    *   The **precision score significantly improved to 57%**. This is better than the baseline (53% for previous model, 53.6% natural market up-days), indicating that the **model now has predictive value**.
*   **Next Steps for Further Improvement**:
    *   Incorporate data from **overnight exchanges**.
    *   Add **news articles** and **macroeconomic conditions** (e.g., interest rates, inflation).
    *   Include data on **key components of the S&P 500** (individual stocks or sectors).
    *   Increase data **resolution** (e.g., hourly, minute-by-minute, or tick data) for more granular predictions.
