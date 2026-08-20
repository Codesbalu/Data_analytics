# MySQL — Layoff Data Project

This folder contains SQL scripts and data used to clean and explore a layoff dataset using MySQL.

Contents
- `layoffs.csv` — Raw dataset (CSV) of company layoffs and related metadata.
- `layoff_data_cleaning_project.sql` — SQL script that demonstrates data cleaning steps: removing duplicates, standardizing text, handling NULL/blank values, normalizing the date column, and other transformations. Uses window functions and JOIN-based updates.
- `layoff_data_exploratory_analysis.sql` — SQL script with sample queries for exploratory analysis: aggregations by company, industry, country, month/year, rolling totals, and ranking top companies per year.
- `readme.md` — This file.

Prerequisites
- MySQL 8.0+ (the cleaning scripts use window functions such as ROW_NUMBER() and analytic functions).
- Local access to load CSV (or use a GUI like MySQL Workbench).

How to import the CSV into MySQL (example)
1. Create a database and a staging table (the cleaning script creates a `world_layoff` database and uses `layoffs_stagging` / `layoffs_stagging2`).

Example commands (run in your shell or via a MySQL client):

- Create database and use it:
  - CREATE DATABASE IF NOT EXISTS world_layoff;
  - USE world_layoff;

- Create a simple table to match the CSV headers (or let the SQL script create tables):
  - See `layoff_data_cleaning_project.sql` for a full example of table creation.

- Import the CSV (example using LOAD DATA LOCAL INFILE):
  - LOAD DATA LOCAL INFILE 'path/to/mysql/layoffs.csv'
    INTO TABLE layoffs
    FIELDS TERMINATED BY ','
    OPTIONALLY ENCLOSED BY '"'
    LINES TERMINATED BY '\n'
    IGNORE 1 LINES
    (company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions);

Notes on the cleaning script
- Duplicate removal: uses ROW_NUMBER() OVER (PARTITION BY ...) to detect duplicates and delete rows where row_num > 1.
- Trimming and normalization: trims whitespace from company names and normalizes industry/country values (e.g., `Crypto%` -> `Crypto`, `United States%` -> `United States`).
- Date parsing: converts string dates in `m/d/Y` format to proper DATE type using STR_TO_DATE() and then alters the column type.
- Handling NULLs/blanks: replaces empty strings with NULL, backfills industry values for companies when possible, and deletes rows where both `total_laid_off` and `percentage_laid_off` are missing.

Notes on the exploratory analysis script
- Contains sample aggregate queries by company, industry, country, year, and month.
- Demonstrates rolling cumulative sums and per-year dense ranking to show top contributors to layoffs.

Data dictionary (columns in the CSV/table)
- company — Company name.
- location — City / office location.
- industry — Industry category (may contain inconsistent values before cleaning).
- total_laid_off — Integer number of employees laid off (NULL where unknown).
- percentage_laid_off — Fraction or proportion of workforce laid off (may be text or numeric; cleaning may convert/standardize it).
- date — Reported date (usually in mm/dd/yyyy format in the CSV).
- stage — Company stage (e.g., Series A, Post-IPO, Acquired, Unknown).
- country — Country name (may include trailing punctuation or variants; script normalizes some values).
- funds_raised_millions — Funds raised in millions (NULL where unknown).

Caveats & tips
- The CSV contains inconsistent values (e.g., 'United States.' vs 'United States', trailing spaces, and occasional typos). The cleaning script attempts to address common issues but you may need to extend normalization rules.
- Some percentage values are stored as integers (e.g., 1) to indicate 100% — be careful when interpreting these.
- Backup your raw CSV and original table before running destructive operations (DELETE, ALTER).

Extending this work
- Convert percentage fields to consistent numeric types (DECIMAL) and normalize percent vs fraction values.
- Enrich dataset with company metadata (headcount) to compute relative impact.
- Export cleaned results to CSV or connect to BI tools (Power BI / Tableau) for visualization.

Contact / Author
- Repository: Codesbalu/Data_analytics
- If you have questions or want improvements to the scripts, open an issue or submit a PR.
