
# SQL Data Cleaning Project: Layoffs 2022

This repository contains a structured SQL-based data cleaning project performed on the **Layoffs 2022** dataset from Kaggle. The project focuses on cleaning, standardizing, and preparing the data for further exploratory analysis, following systematic data engineering and data preparation practices.

---

## Dataset Overview

- **Source:** [Layoffs 2022 — Kaggle](https://www.kaggle.com/datasets/swaptr/layoffs-2022)
- **Description:** Records of company layoffs during 2022, including information on company, location, industry, total layoffs, percentage layoffs, company stage, country, funds raised, and date.

---

## Tools & Technologies

- **SQL (MySQL)**
- **Kaggle Datasets**

---

## Project Structure

The data cleaning process followed a systematic, multi-step workflow:

---

### 1. Create a Staging Table

A staging table is created to work on the data safely without affecting the original source table.

```sql
CREATE TABLE world_layoffs.layoffs_staging 
LIKE world_layoffs.layoffs;

INSERT layoffs_staging 
SELECT * FROM world_layoffs.layoffs;
```

---

### 2. Remove Duplicate Records

**Objective:** Identify and remove exact duplicate rows based on multiple key columns.

- Used `ROW_NUMBER()` window function with `PARTITION BY` to assign a row number to duplicate rows.
- Records with a row number greater than 1 were identified as duplicates.

```sql
SELECT company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions,
ROW_NUMBER() OVER (PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions) AS row_num
FROM world_layoffs.layoffs_staging;
```

- Created a new staging table `layoffs_staging2` with `row_num` values to ease deletion.

```sql
CREATE TABLE world_layoffs.layoffs_staging2 ( ... , row_num INT);

INSERT INTO world_layoffs.layoffs_staging2
SELECT ..., ROW_NUMBER() OVER (PARTITION BY ... ) AS row_num
FROM world_layoffs.layoffs_staging;
```

- Deleted records where `row_num >= 2`.

```sql
DELETE FROM world_layoffs.layoffs_staging2
WHERE row_num >= 2;
```

---

### 3. Standardize and Correct Data

#### 3.1 Handle Blank and Null Industry Values

- Replaced blank `industry` values with `NULL`.
- Updated null entries by matching other rows of the same company with non-null values.

```sql
UPDATE world_layoffs.layoffs_staging2
SET industry = NULL
WHERE industry = '';

UPDATE layoffs_staging2 t1
JOIN layoffs_staging2 t2
ON t1.company = t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL
AND t2.industry IS NOT NULL;
```

#### 3.2 Standardize Industry Categories

Unified inconsistent entries for 'Crypto' industry values.

```sql
UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry IN ('Crypto Currency', 'CryptoCurrency');
```

#### 3.3 Standardize Country Values

- Removed trailing periods from country names using `TRIM()`.

```sql
UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country);
```

---

### 4. Convert Date Formats

- Converted text-based dates into SQL `DATE` type using `STR_TO_DATE`.
- Altered the column type to `DATE` for clean date operations.

```sql
UPDATE layoffs_staging2
SET `date` = STR_TO_DATE(`date`, '%m/%d/%Y');

ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;
```

---

### 5. Review and Manage Null Values

- Retained null values in `total_laid_off`, `percentage_laid_off`, and `funds_raised_millions` for meaningful interpretation in EDA.
- Removed records where both `total_laid_off` and `percentage_laid_off` were null as these were non-informative.

```sql
DELETE FROM world_layoffs.layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;
```

---

### 6. Remove Temporary Columns

- Dropped the `row_num` column used during duplicate removal.

```sql
ALTER TABLE layoffs_staging2
DROP COLUMN row_num;
```

---

## Final Cleaned Dataset

The cleaned and standardized dataset is available in the `layoffs_staging2` table, ready for further exploratory data analysis.

---

## Key Outcomes

- Created a structured, reliable, and analysis-ready dataset through systematic data cleaning.
- Gained experience in real-world data preparation tasks like duplicate handling, standardizing inconsistent entries, null value management, and date conversions.
- Reinforced best practices for working with staging tables and preserving data integrity.
