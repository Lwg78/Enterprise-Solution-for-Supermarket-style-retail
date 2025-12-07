# 🛒 Enterprise Retail Sales Forecasting Pipeline (Armstrong Cycle Implementation)

## 1. Project Overview
This repository contains an End-to-End Machine Learning Pipeline designed to predict daily retail sales in a high-volume FMCG environment (modeled after **NTUC FairPrice** operations).

Unlike standard forecasting tools, this project addresses the specific challenge of "dirty" retail data and complex seasonality by combining **Automated ETL** with the **Martin Armstrong Economic Confidence Model (ECM)**.

### Key Capabilities
* **Automated Data Cleaning:** A "Universal Scraper" that handles inconsistent Excel headers and messy formats automatically.
* **Advanced Feature Engineering:** Custom Scikit-Learn Transformers that mathematically model long-term economic cycles (8.6 years) alongside short-term business seasonality.
* **Simulation Mode:** Built-in synthetic data generator to stress-test the model when historical client data is sparse.

---

## 2. Folder Structure
The project follows a modular production-grade structure:

* **`data`**:
    * `raw/`: Drop messy Excel files here (supports `.xls` and `.xlsx`).
    * `processed/`: Contains the merged CSVs and synthetic datasets.
* **`src`**:
    * `data_cleaner.py`: Smart header detection logic to ingest inconsistent Excel reports.
    * `preprocessor.py`: Aggregates transactional data into daily time-series formats.
    * `ArmstrongCycleTransformer.py`: **(Core IP)** Custom math logic for Pi-Cycle feature generation.
    * `datagen.py`: Generates 5-years of synthetic sales data for simulation.
* **`main.py`**: The orchestration script that runs the full pipeline.

---

## 3. Instructions for Execution

### Prerequisites
1.  Python 3.9+ installed.
2.  Install dependencies (Pandas, Scikit-Learn, Numpy):
    ```bash
    pip install pandas numpy scikit-learn openpyxl xlrd
    ```

### Running the Pipeline
The `main.py` script is the single entry point. It features a **toggle** to switch between Real Data and Simulation Mode.

**Step 1: Configure Mode**
Open `main.py` and set the toggle:
```python
# Set to True to test with generated data (5 years)
# Set to False to process raw Excel files in data/raw
USE_SYNTHETIC = True 
````

**Step 2: Execute**

```bash
python main.py
```

**Expected Output:**
The pipeline will output the training logs, the number of days processed, and the final **Mean Absolute Error (MAE)**.

-----

## 4\. Pipeline Logic & Data Flow

The system is orchestrated by `main.py` and follows a strict 4-stage process:

1.  **Ingestion (`data_cleaner.py`)**:

      * Scans `data/raw` for any Excel files.
      * **Smart Header Detection:** Instead of assuming Row 0 is the header, it scans the first 10 rows to find keywords like "Date" or "SKU". This handles files with title rows or empty space.
      * **Standardization:** Forces all columns to snake\_case (e.g., `Sales Amt` -\> `sales_amt`).

2.  **Preprocessing (`preprocessor.py`)**:

      * Aggregates granular transaction rows into **Daily Sales Totals**.
      * Ensures strict datetime sorting to prevent time-leakage in the model.

3.  **Feature Engineering (`ArmstrongCycleTransformer.py`)**:

      * This is the "Secret Sauce." It transforms a simple date into three cyclical features:
          * **Macro Pi Cycle:** $\sin(2\pi \times t / 3141)$ (8.6 years)
          * **Quarter Cycle:** $\sin(2\pi \times t / 785)$ (2.15 years)
          * **Business Cycle:** $\sin(2\pi \times t / 365.25)$ (Yearly Seasonality)

4.  **Modeling & Evaluation (`main.py`)**:

      * **Model:** Random Forest Regressor (`n_estimators=100`).
      * **Validation:** Time-series split (Train on past, Test on future).

-----

## 5\. Feature Processing Summary

| Feature Name | Type | Processing Steps | Rationale |
| :--- | :--- | :--- | :--- |
| **Date** | Datetime | **Armstrong Transformation** | Converts linear time into cyclical sine waves to capture seasonality. |
| **Sales Amount** | Numeric | Aggregation (Sum) | Retail prediction requires daily volume, not individual receipt data. |
| **Cycle Features** | Mathematical | Sine/Cosine Transformation | Allows tree-based models (Random Forest) to "see" periodic patterns. |

-----

## 6\. Model Selection & Results

### Why Random Forest?

We selected **Random Forest** over Linear Regression because retail sales data is highly non-linear (spikes during holidays, dips during stock-outs). Random Forest handles these irregularities and outliers significantly better than linear models.

### Performance Analysis

  * **Metric:** Mean Absolute Error (MAE) & Error Percentage.
  * **Current Baseline (Synthetic Data):** \~35% Error.
  * **Analysis:** The current error rate is driven by the synthetic data's linear growth trend (Year 1: $20k -\> Year 5: $40k).
  * **Limitation:** Random Forests cannot extrapolate trends outside their training range.
  * **Future Improvement:** The next iteration will implement a **Hybrid Model** (XGBoost + Linear Booster) or detrending the data before training to capture the growth component accurately.

-----

## 7\. Business Impact (The "So What?")

For a retail giant like NTUC FairPrice, accurate forecasting means:

1.  **Reduced Waste:** Fresh produce isn't over-ordered.
2.  **Stock Availability:** High-demand items (detecting the "Business Cycle") are stocked before the rush.
3.  **Automation:** Saving hours of manual Excel cleaning every week via `data_cleaner.py`.

<!-- end list -->

```
```
