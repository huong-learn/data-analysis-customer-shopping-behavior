# Customer Shopping Behavior Analysis

An end-to-end data analytics project that explores the shopping behavior of **3,900 retail customers**. The project follows a complete data lifecycle — framing the **business problem**, cleaning and feature engineering in **Python (pandas)**, business analysis in **MySQL**, interactive reporting in **Power BI**, and a final **presentation** summarizing insights and recommendations.

The goal is to turn raw transactional data into actionable business insights: who spends the most, how discounts and subscriptions affect revenue, which products perform best, and how customers can be segmented for targeted marketing.

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| **Business problem document** | Scope, stakeholder questions, and success criteria that drive the analysis |
| **Python (pandas, SQLAlchemy)** | Data cleaning, feature engineering, automated load into MySQL |
| **MySQL** | Business-question analysis with SQL (aggregations, segmentation, window functions) |
| **Power BI** | Interactive dashboard and visualizations |
| **Presentation (PDF + PowerPoint)** | Final summary of findings and recommendations |

---

## Project Structure

Files are numbered to reflect the order of the workflow.

| File | Description |
|------|-------------|
| `README.md` | This file — project overview and run instructions |
| `1. Business Problem  Document.pdf` | Business context: the questions the analysis needs to answer and why they matter |
| `2. raw_customer_shopping_behavior.csv` | Source dataset — 3,900 rows, 18 columns (age, gender, item, category, purchase amount, location, size, color, season, review rating, subscription, shipping, discounts, payment method, purchase frequency, etc.) |
| `3. customer_shopping_behavior.ipynb` | Python/pandas notebook: cleaning, feature engineering, and automated push to MySQL |
| `4. cleaned_customer_shopping_behavior.csv` | Cleaned dataset with standardized column names and derived columns |
| `5. customer_shopping_behavior_analysis.sql` | 10 business-question SQL queries |
| `6. customer_behavior_dashboard.pbix` | Power BI dashboard file |
| `7. Customer Shopping Behavior Analysis.pdf` | Written report of the analysis |
| `8. Final presentation.pptx` and `9. Final Presentation (version 2).pptx`| PowerPoint presentation of findings and recommendations |

---

## How to Run the Project

The project is designed to run in five sequential stages. Follow them in order.

### Prerequisites

- Python 3.9+ with `pandas` and `sqlalchemy` (plus a MySQL driver such as `pymysql` or `mysql-connector-python`)
- MySQL Server + MySQL Workbench
- Power BI Desktop

Install the Python dependencies:

```bash
pip install pandas sqlalchemy pymysql jupyter
```

### Step 0 — Read the Business Problem

Start with `1. Business Problem  Document.pdf`. It defines the business context, the stakeholder questions the analysis must answer, and what a useful result looks like — the 10 SQL questions in Step 2 and the dashboard in Step 3 both trace back to it.

### Step 1 — Python: Clean the Data & Engineer Features

Open the notebook and run all cells:

```bash
jupyter notebook "3. customer_shopping_behavior.ipynb"
```

This stage:
- Loads `2. raw_customer_shopping_behavior.csv` with `pandas`.
- Standardizes column names (e.g. `Purchase Amount (USD)` → `purchase_amount`).
- Creates derived columns:
  - **`age_group`** — quartile-based age bands (`qcut`).
  - **`purchase_frequency_days`** — maps text frequency (e.g. "Fortnightly", "Weekly") to numeric days via a mapping dictionary.

Once the DataFrame is cleaned, there are **two ways to move it into MySQL** — pick whichever fits your workflow:

#### Option A — Export to CSV, then import manually via MySQL Workbench

Save the cleaned DataFrame to a CSV file:

```python
df.to_csv("4. cleaned_customer_shopping_behavior.csv", index=False)
```

Then open MySQL Workbench and import the file using the **Table Data Import Wizard** (right-click a schema → *Table Data Import Wizard* → select `4. cleaned_customer_shopping_behavior.csv`). Simple, no extra setup — good for a one-off load.

#### Option B — Automated push with SQLAlchemy (recommended)

Load the DataFrame straight into MySQL from Python, no manual import step:

```python
from sqlalchemy import create_engine

# 1. Set up MySQL credentials
user = "root"
password = "my_password" #hidden
host = "localhost"
port = "3306"          # default MySQL port
database = "your_database"   # the database you created in Workbench

# 2. Create the MySQL connection engine
engine = create_engine(f"mysql+pymysql://{user}:{password}@{host}:{port}/{database}")

# 3. Push the clean DataFrame straight into MySQL
df.to_sql("cleaned_customer_shopping_behavior", con=engine, if_exists="replace", index=False)
```

> Update the credentials (username, password, host, port, database) before running. This creates/replaces the `cleaned_customer_shopping_behavior` table automatically — repeatable and script-friendly.

### Step 2 — MySQL: Business Analysis

Import the cleaned data into a MySQL database (either automatically from Step 1, or manually via MySQL Workbench), then run the analysis queries:

```sql
SOURCE "5. customer_shopping_behavior_analysis.sql";
```

The script answers 10 business questions, including:
1. Total revenue by gender
2. Discount users who still spend above the average purchase amount
3. Top 5 products by average review rating
4. Average purchase amount: Standard vs. Express shipping
5. Subscribers vs. non-subscribers — spend and total revenue
6. Top 5 products by discount-usage rate
7. Customer segmentation into **New / Returning / Loyal**
8. Top 3 products per category (window functions)
9. Do repeat buyers also subscribe?
10. Revenue contribution by age group

### Step 3 — Power BI: Dashboard

Open `6. customer_behavior_dashboard.pbix` in Power BI Desktop. Point the data source to either the cleaned CSV or the MySQL table, then refresh to explore the interactive visualizations (revenue breakdowns, customer segments, product performance, and more).

### Step 4 — Presentation

Review the final presentation — a summary of the key findings from the SQL analysis and Power BI dashboard, along with business recommendations:
- `7. Customer Shopping Behavior Analysis.pdf` — written report.
- `8. Final presentation.pptx` — PowerPoint slide deck.
- `9. Final Presentation (version 2).pptx` — Another design for the presentation.

---

## Key Insights

The analysis surfaces insights such as revenue differences across gender and age groups, the impact of subscriptions and discounts on spending, top-performing products, and customer loyalty segments — providing a foundation for targeted marketing and retention strategies.
