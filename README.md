# Customer Churn Analysis

An exploratory retail analytics project that uses customer-level purchasing behaviour to identify customers who may be at risk of churning. The notebook combines sales analysis, RFM-style customer summaries, and a Random Forest classifier to produce a prioritised customer list.

## Project contents

- `MainAnalysis.ipynb` — data cleaning, exploratory analysis, customer feature engineering, churn modelling, visualisations, and export.
- `Online Retail.xlsx` — transaction-level retail data used by the notebook.
- `high_priority_at_risk_customers.xlsx` — an example/exported list of customers flagged for review by the analysis.
- `README.md` — project overview and instructions.

> Keep the dataset and notebook in the same directory, or update the input path in the notebook to match your setup.

## Goals

- Explore sales patterns by time, country, customer, and product.
- Summarise customer behaviour using recency, frequency, and monetary value.
- Define a simple churn label based on customer inactivity.
- Train a Random Forest classifier and estimate churn risk.
- Export a list of higher-priority customers for retention follow-up.

## Data

The workbook contains transaction-level fields including:

| Column | Description |
| --- | --- |
| `InvoiceNo` | Invoice identifier; invoices beginning with `C` are excluded as cancellations/returns in the notebook. |
| `StockCode` | Product code. |
| `Description` | Product description. |
| `Quantity` | Number of units on the transaction line. |
| `InvoiceDate` | Date and time of the transaction. |
| `UnitPrice` | Unit price. |
| `CustomerID` | Customer identifier. |
| `Country` | Customer's country. |

The notebook removes rows with missing values and calculates line-level sales as `Quantity × UnitPrice`. Customer summaries are built from the remaining transaction records.

## Method

### 1. Data preparation and exploration

The notebook cleans missing values, excludes invoice numbers beginning with `C`, converts invoice dates to datetime, and calculates sales. It explores revenue over time, top customers and products, sales by country, and purchasing patterns by hour and weekday.

### 2. Customer features

For each customer, the notebook calculates:

- **Recency** — days since their most recent purchase, measured relative to one day after the latest invoice date in the dataset.
- **Frequency** — count of transaction rows for the customer (not necessarily the count of distinct orders).
- **Monetary** — total calculated sales for the customer.
- **Average order value** — calculated in the notebook as monetary value divided by frequency. Since frequency is a transaction-row count, this is an average value per transaction row rather than a true order-level average.

### 3. Churn definition and model

The notebook labels a customer as `Churned = 1` when recency is greater than 90 days; otherwise, `Churned = 0`. A Random Forest classifier is trained using `frequency`, `monetary`, and `AvgOrderValue`, and its predicted class-1 probability is saved as `churn_probability`.

The notebook's saved run reports **69% accuracy** for the Random Forest on its test split. This is an initial result, not evidence that the model is production-ready. The saved classification report shows weaker recall for the churn class (0.39), so some churn-labelled customers may be missed.

### 4. Priority customer export

The notebook selects customers whose predicted churn probability is greater than 0.5 and whose rule-based label is `Churned = 0`, then sorts them by predicted risk and exports up to 50 rows to `high_priority_at_risk_customers.xlsx`.

The export is intended as a review list for possible retention outreach, not as a definitive prediction of future behaviour.

## Requirements

The notebook uses Python and the following packages:

- `pandas`
- `numpy`
- `matplotlib`
- `scikit-learn`
- `openpyxl` (for Excel file input/output)

Install dependencies with:

```bash
python -m pip install pandas numpy matplotlib scikit-learn openpyxl jupyter
```

## How to run

1. Download or clone the project files.
2. Place the notebook and `Online Retail(1).xlsx` in the same directory.
3. Install the dependencies listed above.
4. Open `MainAnalysis(1).ipynb` in Jupyter Notebook, JupyterLab, or VS Code.
5. Run the notebook cells from top to bottom.
6. Check the generated charts and metrics, then review `high_priority_at_risk_customers.xlsx` in the working directory.

If your dataset has a different filename, edit the `pd.read_excel(...)` path near the beginning of the notebook. The output workbook is written to the notebook's current working directory.

## Important limitations and next steps

- **The churn label is a heuristic.** Inactivity for more than 90 days is used as a proxy for churn; it is not a confirmed business outcome.
- **The features and label need further validation.** The label is defined from recency, while the model uses purchasing-value features that may only indirectly reflect inactivity. Consider including recency in a clearly designed baseline, using a time-based train/test split, and evaluating performance against a future observation window.
- **Check the modelling code before relying on results.** In the current notebook, the Logistic Regression model is fitted on unscaled training data but predicts on scaled test data. Its reported metric should not be treated as reliable until this is corrected. The Random Forest is trained and tested on the unscaled features.
- **Avoid interpreting probabilities as calibrated probabilities.** The output is a model score and has not been shown to be calibrated.
- **Validate the customer list.** Review the selected customers and thresholds with business stakeholders before using the list for campaigns.
- **Currency and data provenance.** The notebook labels sales axes as dollars, but the workbook itself does not establish a currency in the columns shown. Confirm the source dataset's currency before reporting financial totals.

## Output fields

The exported customer list includes:

- `CustomerID`
- `recency`
- `frequency`
- `monetary`
- `Churned`
- `AvgOrderValue`
- `churn_probability`


No license or data-source attribution is specified in the files provided with this project. Add the applicable dataset citation and license terms before redistributing the data or repository.
