# US–EUR Yield Curve Differential Analysis

## 1. Project Overview

This project collects and compares US Treasury yields with euro-area AAA-rated government bond yields using Python. It focuses on the 2-year (2Y) and 10-year (10Y) maturities to analyse changes in the interest rate differential between the United States and the euro area.

The objective is to create a daily dataset that can help monitor relative interest rate movements and generate rule-based EUR/USD signals based on changes in the yield differentials.

The workflow is designed to be repeatable and can be extended to run automatically on a daily schedule.

## 2. Why This Project?

Interest rate differentials are an important macroeconomic indicator in foreign exchange analysis. Differences between US and euro-area yields can reflect changing monetary policy expectations, inflation expectations, and economic conditions.

This project aims to:

- Monitor the difference between US and euro-area 2Y and 10Y yields.
- Track how these differentials change over a one-week period.
- Generate simple, rule-based EUR/USD signals from changes in the differentials.
- Maintain a consistent daily dataset despite weekends and market holidays.
- Provide a foundation for further quantitative analysis, backtesting, and strategy development.

**Important:** Yield differentials alone do not determine exchange rate movements. The signals are analytical rules, not guaranteed trading recommendations.

## 3. Data Sources

### US Treasury yields

US nominal Treasury yield data is retrieved using the Python package `ustreasurycurve`.

The project uses:
- 2-year US Treasury yield
- 10-year US Treasury yield
- Observation date

The data is matched to a business-day calendar. Where no new observation is available, the most recent available observation can be carried forward.

### Euro-area AAA yield curve

Euro-area AAA-rated government bond yields are retrieved from the European Central Bank (ECB) Data API.

The project uses:
- 2-year euro-area AAA yield
- 10-year euro-area AAA yield
- Observation date

The ECB series are selected by maturity and reshaped into a table with one column per maturity. The data is aligned to a business-day calendar, with missing observations handled through forward-filling where appropriate.

The ECB curve represents AAA-rated euro-area central government bonds, rather than a single country's government bonds.

## 4. Methodology

### Step 1: Retrieve the data

The Python workflow downloads the relevant US Treasury and ECB yield observations for the selected date range.

### Step 2: Align the dates

Both datasets are converted to a consistent datetime format and aligned by date.

Weekends are excluded from the business-day calendar. Missing weekday observations, including dates affected by market holidays, may be filled using the latest available observation.

A one-observation lag is also applied in the individual data preparation steps where specified. This is intended to avoid using the latest observation immediately and should be interpreted consistently across both datasets.

### Step 3: Combine the datasets

The US and euro-area data are merged into one DataFrame, `df_combined`, containing:

- `date`
- `US_2Y`
- `US_10Y`
- `EUR_2Y`
- `EUR_10Y`

### Step 4: Calculate the yield differentials

The differentials are calculated as US yields minus euro-area yields:

**2-year differential**

`US_EUR_2Y_Diff = US_2Y - EUR_2Y`

**10-year differential**

`US_EUR_10Y_Diff = US_10Y - EUR_10Y`

A positive differential means the US yield is higher than the corresponding euro-area yield. A negative differential means the euro-area yield is higher.

### Step 5: Calculate the one-week change

The project measures how much each differential has changed compared with the corresponding observation five business-day rows earlier:

`US_EUR_2Y_1W_Change = US_EUR_2Y_Diff - US_EUR_2Y_Diff.shift(5)`

`US_EUR_10Y_1W_Change = US_EUR_10Y_Diff - US_EUR_10Y_Diff.shift(5)`

This approximates a same-weekday comparison when the dataset contains a complete weekday calendar. It may not correspond exactly to seven calendar days around market holidays.

The first five rows do not have sufficient historical observations to calculate the change. These rows are excluded from the final output.

### Step 6: Generate rule-based signals

Signals are based on the **one-week change in the yield differential**, not on the absolute level of the differential.

| Condition | Signal |
|---|---|
| One-week change > +0.05 percentage points | SELL EUR/USD |
| One-week change < -0.05 percentage points | BUY EUR/USD |
| One-week change between -0.05 and +0.05, inclusive | NEUTRAL |

The rules are applied independently to the 2Y and 10Y differentials, producing separate signals for each maturity.

The thresholds assume yields are expressed in percentage points. A change of 0.05 percentage points is equivalent to 5 basis points.

These signals are mechanical indicators. Their economic interpretation and predictive value must be tested before being used in a live trading strategy.

## 5. Main Output

The final combined dataset, `df_combined`, includes:

| Column | Description |
|---|---|
| `date` | Observation date |
| `US_2Y` | US 2-year Treasury yield |
| `US_10Y` | US 10-year Treasury yield |
| `EUR_2Y` | Euro-area AAA 2-year yield |
| `EUR_10Y` | Euro-area AAA 10-year yield |
| `US_EUR_2Y_Diff` | US minus euro-area 2Y yield |
| `US_EUR_10Y_Diff` | US minus euro-area 10Y yield |
| `US_EUR_2Y_1W_Change` | One-week change in the 2Y differential |
| `US_EUR_10Y_1W_Change` | One-week change in the 10Y differential |
| `Signal_2Y` | Rule-based 2Y signal |
| `Signal_10Y` | Rule-based 10Y signal |

Numerical values are rounded to four decimal places for presentation.

The final table can be exported to CSV or Excel for further analysis, visualisation, or backtesting.

## 6. Technologies Used

- **Python** — core programming language
- **pandas** — data cleaning, date alignment, merging, and calculations
- **NumPy** — rule-based signal generation
- **requests** — HTTP requests to the ECB API
- **ustreasurycurve** — retrieval of US Treasury yield data
- **Jupyter Notebook / Google Colab / VS Code** — interactive development and execution
- **pyarrow** — compatibility support for pandas conversion where required by the Treasury data package

## 7. How to Run the Project

1. Open the notebook in Jupyter, VS Code, or Google Colab.
2. Install the required dependencies in the active Python environment.
3. Run the data-import and configuration cells.
4. Run the US Treasury data retrieval code.
5. Run the ECB data retrieval and calendar-alignment code.
6. Merge the datasets and calculate the yield differentials.
7. Calculate the one-week changes and generate the signals.
8. Review the final DataFrame and optionally export it to CSV or Excel.

Example dependency installation:

```bash
pip install pandas numpy requests ustreasurycurve pyarrow
```

Depending on the environment, a kernel restart may be required after installing packages.

## 8. Automation

The workflow can be scheduled to run daily rather than requiring manual execution.

Possible approaches include:
- Windows Task Scheduler for a local Windows computer.
- `cron` for Linux or macOS.
- A cloud notebook or scheduled cloud job for execution without relying on a personal computer being switched on.

For reliable automation, the workflow should save the output dataset, log execution errors, and verify that the latest data was retrieved successfully.

Google Colab does not automatically schedule a notebook to run daily simply because it is open or has been executed once.

## 9. Limitations and Considerations

- Yield differentials are only one of many factors affecting EUR/USD.
- The five-row comparison approximates a one-week change and can be affected by differing US and ECB holiday calendars.
- Forward-filling creates a continuous weekday dataset but does not represent a new market observation on every date.
- Data publication times and market closing times can differ between the sources.
- A one-observation lag should be applied consistently and documented to avoid unintended timing differences.
- The thresholds have not been demonstrated to be profitable merely by generating signals.
- Historical testing should account for transaction costs, slippage, data availability, and look-ahead bias.

## 10. Potential Future Improvements

- Backtest the 2Y and 10Y signals against historical EUR/USD exchange rates.
- Compare alternative thresholds and lookback periods.
- Add yield curve slope indicators, such as 10Y minus 2Y.
- Create charts showing yield differentials and EUR/USD movements.
- Export daily results automatically and maintain a historical signal log.
- Add error handling, data validation, and alerts when an API download fails.
- Refine date matching to handle US and ECB holidays explicitly.

## Summary

This project provides a reproducible Python workflow for comparing US Treasury and euro-area AAA yields, calculating 2Y and 10Y yield differentials, measuring their weekly changes, and generating simple EUR/USD signals.

Its main purpose is to turn daily interest rate data into a structured analytical dataset that can be used for monitoring, research, and future strategy testing.
