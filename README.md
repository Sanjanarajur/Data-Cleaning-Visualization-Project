# Data-Cleaning-Visualization-Project
An end-to-end data analytics project demonstrating data preprocessing, cleaning, exploratory data analysis (EDA), and visualization techniques on the Bank Marketing dataset using Python, Pandas, Matplotlib, and Seaborn.
# Data Cleaning & Visualization Project

## Objective
Work on a raw dataset to clean, process, and visualize insights — covering data
inspection, handling of missing values / duplicates / outliers, and building a
set of clear, well-labeled visualizations that summarize key patterns in the
data.

## Dataset Used
**Bank Marketing Dataset** — `data/bankmarketing(4).csv`

The dataset contains **41,188 records and 21 columns** describing a
Portuguese bank's direct marketing campaigns (phone calls) to sell term
deposits. It includes client demographics (age, job, marital status,
education), campaign details (contact type, month, duration, number of
contacts), economic indicators (`emp.var.rate`, `euribor3m`, etc.), and the
target outcome column `y` (whether the client subscribed to a term deposit).

## Libraries Used
- **Pandas** – data loading, inspection, and cleaning
- **NumPy** – numerical operations (IQR outlier bound calculations)
- **Matplotlib** – core plotting
- **Seaborn** – statistical visualizations (count plots, box plots, heatmap)

## Data Cleaning Steps
1. **Load & Inspect** — loaded the CSV with Pandas and inspected it using
   `.head()`, `.info()`, and `.describe()`.
2. **Missing Values** — the dataset has no explicit `NaN` values, but several
   categorical columns (e.g. `job`, `education`, `default`, `housing`,
   `loan`) use the string `"unknown"` to represent missing/unrecorded
   information. These were identified and quantified. Rather than dropping
   these records (which would discard a large portion of the data and bias
   the results), `"unknown"` was retained as its own valid category.
3. **Duplicate Records** — identified and removed **12 exact duplicate
   rows** using `duplicated()` / `drop_duplicates()`.
4. **Outliers** — applied the **IQR (Interquartile Range) method** to
   `age`, `campaign`, `duration`, and `previous`. Instead of deleting rows,
   outlier values were **capped (winsorized)** to the IQR bounds to reduce
   the influence of extreme values while preserving all records. The
   `pdays` column's sentinel value (`999` = "not previously contacted")
   was intentionally left untouched, since it's a coded value rather than a
   true outlier.
5. **Before/After Summary** — row count, missing values, and duplicate
   counts were compared before and after cleaning to confirm the cleaning
   was effective.

## Visualizations Included
All charts are saved in the `images/` folder:

| File | Chart Type | Description |
|---|---|---|
| `histogram.png` | Histogram | Distribution of client age |
| `barchart_job_subscription.png` | Bar chart | Subscription rate (%) by job |
| `countplot_subscription.png` | Count plot | Count of clients by subscription outcome |
| `countplot_marital.png` | Count plot | Count of clients by marital status |
| `boxplot_age_subscription.png` | Box plot | Age distribution by subscription outcome |
| `boxplot.png` | Box plot | Call duration spread (after outlier capping) |
| `heatmap.png` | Correlation heatmap | Correlations among numeric features |
| `piechart_contact.png` | Pie chart | Share of contact type (cellular vs. telephone) |
| `dashboard_report.png` | Combined dashboard | All charts arranged in a single visual report |

Every chart includes a clear title, labeled axes, and consistent formatting.

## Key Findings
1. **Age distribution** is right-skewed and concentrated between 30–50
   years, with fewer clients at the younger and older ends.
2. **Subscription rate by job** varies noticeably — students and retired
   clients show a higher term-deposit subscription rate than blue-collar
   or services workers.
3. **Overall subscription outcome** is heavily imbalanced: the large
   majority of clients did **not** subscribe to a term deposit.
4. **Marital status** shows most clients are married, followed by single,
   with divorced clients being the smallest group.
5. **Age vs. subscription** — clients who subscribed show a slightly wider
   age spread, including more older clients, compared to those who did not
   subscribe.
6. **Call duration** remains strongly right-skewed even after capping,
   indicating most calls are short, with a smaller number of longer calls.
7. **Correlation heatmap** shows `euribor3m`, `emp.var.rate`, and
   `nr.employed` are strongly positively correlated with each other
   (shared macroeconomic movement), while `duration` shows little linear
   correlation with other numeric features.
8. **Contact type** — cellular contact was used far more frequently than
   telephone, suggesting a shift toward mobile-based outreach.

## Project Structure
```
Data-Cleaning-Visualization-Project/
│
├── data/
│   └── bankmarketing(4).csv
│
├── notebooks/
│   └── Data_Cleaning_Visualization.ipynb
│
├── images/
│   ├── histogram.png
│   ├── barchart_job_subscription.png
│   ├── countplot_subscription.png
│   ├── countplot_marital.png
│   ├── boxplot_age_subscription.png
│   ├── boxplot.png
│   ├── heatmap.png
│   ├── piechart_contact.png
│   └── dashboard_report.png
│
├── README.md
├── requirements.txt
└── LICENSE
```

## How to Run the Project
1. Clone or download this repository.
2. (Recommended) Create a virtual environment:
   ```bash
   python -m venv venv
   source venv/bin/activate   # On Windows: venv\Scripts\activate
   ```
3. Install the required libraries:
   ```bash
   pip install -r requirements.txt
   ```
4. Launch Jupyter Notebook:
   ```bash
   jupyter notebook notebooks/Data_Cleaning_Visualization.ipynb
   ```
5. Run all cells (`Cell -> Run All`) to reproduce the cleaning steps and
   regenerate all visualizations into the `images/` folder.
