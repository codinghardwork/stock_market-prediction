 
# Project: Forecasting Tomorrow’s S\&P 500 Index Movement

## Purpose

To forecast whether the S\&P 500 index price will **increase or decrease tomorrow** using machine learning and Python, and to **backtest the model** on over **20 years** of historical data to ensure prediction reliability.

---

## Tools Used

* Jupyter Lab / Jupyter Notebook
* Python packages:

  * `yfinance` – to fetch stock/index prices via the Yahoo Finance API
  * `pandas`, `scikit-learn`, `numpy` – for data manipulation and modeling

---

## Data Acquisition

* **Ticker symbol**: `^GSPC` (S\&P 500 Index)
* **Method** to retrieve maximum historical data:

  ```python
  Ticker("^GSPC").history(period="max")
  ```
* **Data**: A `pandas.DataFrame`

  * Each row represents one trading day
  * Important columns: `Open`, `High`, `Low`, `Close`, `Volume`
  * Index: `DateTimeIndex` for easy slicing and filtering

---

## Data Cleaning and Preprocessing

* **Dropped irrelevant columns**: `Dividends` and `Stock Splits` (not useful for index-level forecasting)

* **Target Definition**:

  * Objective: Predict direction (up/down), not actual price
  * Created a `tomorrow` column: the next day’s `Close` value
  * Created a `target` column:

    ```python
    target = 1 if tomorrow > today else 0
    ```
  * Converted to integer type for compatibility with machine learning models

* **Filtered historical range**:

  * Dropped all data before January 1, 1990
  * Used `.copy()` when subsetting to avoid Pandas warnings

---

## Initial Machine Learning Model

### Model: Random Forest Classifier

* **Reasons for selection**:

  * Resistant to overfitting
  * Fast training and prediction
  * Handles non-linear patterns well

* **Initial parameters**:

  ```python
  RandomForestClassifier(
      n_estimators=100,        # number of trees (can be increased later)
      min_samples_split=10,    # controls tree depth
      random_state=1           # ensures reproducibility
  )
  ```

---

## Train/Test Split for Time Series

* **No standard cross-validation**:

  * To prevent data leakage, where future data influences past predictions

* **Split strategy**:

  * Last 100 rows used as test set
  * Remaining historical data used for training
  * Time order preserved

* **Initial predictors**:

  ```python
  ['Close', 'Volume', 'Open', 'High', 'Low']
  ```

---

## Model Evaluation (Initial)

* **Evaluation metric**: Precision score

  * Measures: “Of all the times the model predicted the market would go up, how often was it correct?”

* **First result**:

  * Precision: **42%**
  * This is below chance (market goes up \~53.6% of days), meaning an inverse strategy may outperform

---

## Strong Backtesting

### Purpose: Simulate real-world use and test over decades

* **Functions created**:

  * `predict_function`: wraps training and prediction
  * `backtest_function`: simulates production-like forecasting

* **Backtesting strategy**:

  * Initial training set: first \~2500 trading days (≈10 years)
  * Forecast horizon: next 250 days (≈1 year)
  * Repeat: extend training window and forecast forward

    * Train years 1–10 → forecast year 11
    * Train years 1–11 → forecast year 12, etc.

* **Backtesting result**:

  * Accuracy: **53%** over \~6,000 trading days
  * Baseline: Market went up **53.6%** of days
  * Conclusion: Initial model is only slightly worse than a naïve "always up" forecast

---

## Model Enhancement Using New Predictors

* **Added features**:

  * Rolling average ratios (e.g., today's Close ÷ 5-day average)

    * Windows: 2, 5, 60, 250, 1000 days
  * Rolling trends: Count of "up" days in previous `n` days

* **Handling missing values**:

  * Early rows with insufficient data for long rolling windows are dropped
  * Data effectively starts from \~1993

* **Parameter updates**:

  ```python
  RandomForestClassifier(
      n_estimators=200,
      min_samples_split=50,
      random_state=1
  )
  ```

* **Prediction strategy**:

  * Switched to `predict_proba()` to return probabilities
  * Set custom threshold:

    ```python
    predict "up" only if probability > 60%
    ```
  * Reduces trade frequency in favor of higher-confidence trades

* **Removed predictors**:

  * Absolute values (`Close`, `Open`, `High`, `Low`, `Volume`) removed
  * Only ratio-based and trend-based predictors retained

---

## Improved Model Evaluation

* **New threshold impact**:

  * Fewer "up" predictions due to confidence filter
  * But those predictions are more accurate

* **Improved precision**:

  * **57%** precision vs

    * 42% from initial model
    * 53.6% market baseline
  * Demonstrates **real predictive power**

 
