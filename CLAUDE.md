# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A single-dataset, end-to-end data analytics portfolio project (3,900 retail customer rows). It is **not** an application — there is no build system, package manifest, test suite, or linter. The "code" is one Jupyter notebook plus one SQL script; everything else is data or deliverables.

## Pipeline architecture

The repo is a linear five-stage pipeline, and the numeric filename prefixes encode the execution order:

```
1. Business Problem Document.pdf          (defines the 10 questions)
        ↓
2. raw_customer_shopping_behavior.csv     (18 cols, "Title Case (USD)" headers, 37 null Review Ratings)
        ↓  3. customer_shopping_behavior.ipynb   ← the only cleaning/feature-engineering code
4. cleaned_customer_shopping_behavior.csv (21 cols, snake_case) ─┐
        ↓  df.to_sql(...) in the same notebook                   │ Option A: manual Workbench import
   MySQL table `cleaned_customer_shopping_behavior`  ←───────────┘ Option B: SQLAlchemy push (recommended)
        ↓  5. customer_shopping_behavior_analysis.sql  (Q1–Q10)
6. customer_behavior_dashboard.pbix  →  7./8./9. report + presentations
```

### The contract between the notebook and the SQL

The notebook and `5. …analysis.sql` are coupled by names, not by any interface. Changing either side breaks the other silently:

- **Table name** must stay `cleaned_customer_shopping_behavior` (hardcoded in `df.to_sql(name=...)` and in all 11 SQL statements).
- **Column names** are produced by the notebook's header normalization (`.str.lower()`, `' '` → `'_'`, `'_(usd)'` → `''`, giving `purchase_amount`). The SQL queries reference these snake_case names directly.
- **Derived columns the SQL depends on**: `age_group` (from `pd.qcut(age, q=4)` → Young Adult / Adult / Middle-Aged / Senior) is used by Q10. `purchase_frequency_days` (mapped from `frequency_of_purchases` via a dictionary) is currently unused by the SQL but is part of the cleaned schema.
- `promo_code_used` is dropped in the notebook after verifying it is 100% identical to `discount_applied` — do not reintroduce it.
- Null `review_rating` values are imputed with the **per-category median**, not the global median.

### Known inconsistency

`age_group_2` and `age_group_3` (exploratory `pd.cut` variants) are still written to `4. cleaned_customer_shopping_behavior.csv`, even though the SQL file's header comment instructs dropping them via `ALTER TABLE`. Only `age_group` is used downstream. If asked to reconcile this, drop them in the notebook before export rather than post-hoc in SQL.

### File renaming

Because files are numbered, the numbers are referenced in prose across `README.md`, the SQL file's header comment block, and the notebook's `read_csv`/`to_csv` calls. Renaming or renumbering a file requires updating all three.

## Commands

Environment setup:

```bash
pip install pandas sqlalchemy pymysql jupyter
```

Stage 1 — run the cleaning notebook (run all cells; execution order matters, cells mutate `df` in place):

```bash
jupyter notebook "3. customer_shopping_behavior.ipynb"
```

Stage 2 — run the analysis queries against the MySQL database (default database name in the notebook is `customer_behavior`):

```sql
SOURCE "5. customer_shopping_behavior_analysis.sql";
```

Stages 3–4 (Power BI `.pbix`, `.pptx`, `.pdf`) are GUI-only and cannot be inspected or edited from the CLI.

## Conventions

- MySQL credentials in the notebook are placeholders (`password = "-----" #hidden`). Keep them redacted; never commit a real password.
- The notebook is written in a teaching style — inline comments explain *why* each pandas operation is used. Match that density when adding cells.
- SQL uses lowercase keywords in later queries and uppercase in earlier ones; aliases are quoted display strings (e.g. `AS 'Average Spend'`). Follow the surrounding query's style rather than imposing a uniform one.
