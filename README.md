# 🧹 Data Cleaning & Exploratory Data Analysis — Layoffs Dataset

A SQL-based data cleaning and EDA pipeline built with **PostgreSQL** to transform raw layoff data into a reliable, analysis-ready dataset and uncover key insights.

## 📌 Overview

This project demonstrates a structured approach to cleaning messy real-world data and performing exploratory data analysis using pure SQL. The source dataset (`Layoffs.csv`) contains global layoff records, and the pipeline addresses common data quality issues: duplicate rows, inconsistent formatting, and null values — followed by a full EDA to explore trends and patterns.

All transformations are performed on a **staging table** to preserve the original raw data.

## 📁 Project Structure

```
Data_cleaning/
├── Dataset/
│   └── Layoffs.csv                      # Raw source data
├── creation_table/
│   └── creation_of_table.sql            # Table creation & CSV import
├── practice_sql.sql/
│   ├── 1_duplicates.sql                 # Step 1 — Remove duplicate rows
│   ├── 2_sanderdize.sql                 # Step 2 — Standardize text fields
│   └── 3_null_values.sql                # Step 3 — Handle NULL values
├── Exploratory Data Analysis/
│   └── EDA.sql                          # Exploratory Data Analysis queries
└── README.md
```

## 🔧 Cleaning Pipeline

The scripts are designed to be run **sequentially**:

### Step 0 — Setup ([`creation_of_table.sql`](creation_table/creation_of_table.sql))

- Creates the `layoffs` table matching the CSV schema
- Imports data from `Layoffs.csv` using `COPY`
- Creates a `layoffs_staging` table (exact clone) so all cleaning is done on the copy, keeping raw data intact

### Step 1 — Remove Duplicates ([`1_duplicates.sql`](practice_sql.sql/1_duplicates.sql))

- Uses a CTE with `ROW_NUMBER()` partitioned across all columns to identify exact duplicate rows
- Leverages PostgreSQL's internal `ctid` to target and delete only the duplicate copies, keeping the first occurrence

### Step 2 — Standardize Data ([`2_sanderdize.sql`](practice_sql.sql/2_sanderdize.sql))

- **Company names** — Trims leading/trailing whitespace (e.g., `" WeWork "` → `"WeWork"`)
- **Industry values** — Consolidates inconsistent crypto-related entries (`"Crypto Currency"`, `"Crypto/Blockchain"`, etc.) into a single `"Crypto"` label
- **Country names** — Removes trailing dots (e.g., `"Brazil."` → `"Brazil"`)

### Step 3 — Handle NULLs ([`3_null_values.sql`](practice_sql.sql/3_null_values.sql))

- Converts blank-space strings to proper `NULL` values
- Back-fills missing `industry` values by joining on `company` — if the same company has a known industry in another row, that value is used
- Deletes rows where **both** `total_laid_off` and `percentage_laid_off` are `NULL` (no useful layoff data)

## 📊 Exploratory Data Analysis ([`EDA.sql`](Exploratory%20Data%20Analysis/EDA.sql))

After cleaning, the following EDA queries were performed on the `layoffs_staging` table:

| # | Analysis | Key Technique |
|---|----------|---------------|
| 1 | Basic overview — total rows, date range, distinct counts | `COUNT`, `MIN`, `MAX`, `DISTINCT` |
| 2 | Numeric summary — min/max of all numeric columns | Aggregate functions |
| 3 | Top 10 companies by total layoffs | `GROUP BY`, `ORDER BY`, `LIMIT` |
| 4 | Companies that completely shut down (100% laid off) | `WHERE percentage_laid_off = 1` |
| 5 | Top 10 industries by total layoffs | CTEs |
| 6 | Top 10 countries by total layoffs | `GROUP BY`, `ORDER BY`, `LIMIT` |
| 7 | Monthly layoff trend | `DATE_TRUNC` |
| 8 | Rolling (cumulative) total of layoffs | CTE + `SUM() OVER()` window function |
| 9 | Total layoffs per year | `EXTRACT(YEAR FROM date)` |
| 10 | Layoffs by company funding stage | `GROUP BY stage` |
| 11 | Top 5 companies per year | Chained CTEs + `DENSE_RANK()` + `PARTITION BY` |
| 12 | Companies with multiple layoff rounds | `HAVING COUNT(*) > 1` |

## 📊 Dataset Schema

| Column                 | Type            | Description                          |
|------------------------|-----------------|--------------------------------------|
| `company`              | `VARCHAR(50)`   | Company name                         |
| `location`             | `VARCHAR(50)`   | City / region                        |
| `industry`             | `VARCHAR(50)`   | Industry sector                      |
| `total_laid_off`       | `INTEGER`       | Number of employees laid off         |
| `percentage_laid_off`  | `NUMERIC(5,2)`  | Percentage of workforce laid off     |
| `date`                 | `DATE`          | Date of layoff event                 |
| `stage`                | `VARCHAR(50)`   | Company funding stage                |
| `country`              | `VARCHAR(50)`   | Country                              |
| `funds_raised_millions`| `NUMERIC(10,2)` | Total funds raised (in millions USD) |

## 🧠 What I Learned

### Data Cleaning
- **Staging tables** — Always work on a copy; never modify raw data directly
- **Duplicate detection** — Using `ROW_NUMBER()` with `PARTITION BY` across all columns to identify exact duplicates
- **`ctid`** — PostgreSQL's internal row identifier, useful for deleting specific duplicate rows
- **`TRIM`** — Cleaning whitespace from text fields
- **Self-joins** — Back-filling missing values by joining a table to itself on a shared key (e.g., filling `industry` based on `company`)
- **NULL handling** — Converting blank strings to `NULL`, and removing rows with no useful data

### Exploratory Data Analysis
- **Aggregate functions** — `SUM`, `COUNT`, `MIN`, `MAX`, `AVG` for summarizing data
- **`GROUP BY` + `ORDER BY` + `LIMIT`** — The core pattern for ranking and top-N queries
- **`WHERE` vs `HAVING`** — `WHERE` filters rows before grouping; `HAVING` filters groups after aggregation
- **Aliases** — Cannot be used in `WHERE` (runs before `SELECT`), but can be used in `ORDER BY`
- **`DATE_TRUNC`** — Truncating dates to month/year for time-series grouping
- **`EXTRACT`** — Pulling year/month from a date for grouping
- **Window functions** — `SUM() OVER(ORDER BY ...)` for rolling/cumulative totals without collapsing rows
- **`DENSE_RANK()`** — Ranking rows within partitions (`PARTITION BY year ORDER BY total_off DESC`)
- **Chained CTEs** — Using multiple CTEs separated by commas to build complex queries step by step
- **Type casting** — `::DATE` to cast timestamps to dates for cleaner output

## 🚀 Getting Started

### Prerequisites

- **PostgreSQL** installed and running
- A tool to run `.sql` files (e.g., `psql`, pgAdmin, DBeaver)

### Usage

1. Clone the repository:
   ```bash
   git clone https://github.com/your-username/Data_cleaning.git
   cd Data_cleaning
   ```

2. Update the CSV path in [`creation_of_table.sql`](creation_table/creation_of_table.sql) (line 29) to point to your local `Layoffs.csv` location.

3. Run the cleaning scripts in order:
   ```sql
   -- Step 0: Create tables & import data
   \i creation_table/creation_of_table.sql

   -- Step 1: Remove duplicates
   \i practice_sql.sql/1_duplicates.sql

   -- Step 2: Standardize text fields
   \i practice_sql.sql/2_sanderdize.sql

   -- Step 3: Handle NULL values
   \i practice_sql.sql/3_null_values.sql
   ```

4. Run the EDA queries:
   ```sql
   \i 'Exploratory Data Analysis/EDA.sql'
   ```

5. Query the cleaned data:
   ```sql
   SELECT * FROM layoffs_staging LIMIT 10;
   ```

## 🛠️ Tech Stack

- **SQL Dialect:** PostgreSQL
- **Key Concepts:** CTEs, Window Functions (`ROW_NUMBER`, `DENSE_RANK`, `SUM() OVER`), Self-Joins, `TRIM`, `COPY`, `DATE_TRUNC`, `EXTRACT`, `HAVING`

## 📝 License

This project is open source and available for learning and personal use.
