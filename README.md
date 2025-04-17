# World_layoffs_Analysis_using_SQL
# Layoffs Analysis 2022 – SQL-Based Data Cleaning & EDA

This project presents a comprehensive data cleaning and exploratory data analysis (EDA) of global layoffs using SQL. The dataset, sourced from [Kaggle](https://www.kaggle.com/datasets/swaptr/layoffs-2022), provides insights into company downsizing trends during economic uncertainty, particularly in the tech and startup ecosystem.

---

## Dataset Overview

- **Source**: Kaggle – [Layoffs 2022 Dataset](https://www.kaggle.com/datasets/swaptr/layoffs-2022)
- **Fields**: 
  - **Company**: Name of the company
  - **Industry**: Industry in which the company operates
  - **Total Laid Off**: Total number of employees laid off
  - **Percentage Laid Off**: Percentage of the workforce laid off
  - **Country**: Country where the company is located
  - **Location**: Specific location of the company’s headquarters or office
  - **Date**: Date when layoffs were announced
  - **Funding Stage**: The stage of funding the company was at when layoffs occurred
  - **Funds Raised**: Total funds raised by the company prior to layoffs

---

## Data Cleaning (SQL)

Performed structured data cleaning using MySQL to prepare the dataset for analysis. The process involved:

### 1. Duplicate Detection & Removal  
   - Used `ROW_NUMBER()` with `PARTITION BY` to identify and remove exact duplicates.
     ```sql
     WITH RowNumbered AS (
         SELECT *,
                ROW_NUMBER() OVER (PARTITION BY Company, Date ORDER BY Date DESC) AS row_num
         FROM layoffs_raw
     )
     DELETE FROM layoffs_raw
     WHERE EXISTS (
         SELECT 1
         FROM RowNumbered
         WHERE layoffs_raw.ID = RowNumbered.ID AND RowNumbered.row_num > 1
     );
     ```
   - Verified edge cases (e.g., companies with similar but valid multiple entries).

### 2. Standardization of Text Fields  
   - Removed inconsistencies in company names, industries, and countries (e.g., spacing, casing) using `TRIM()`, `UPPER()`, and `REPLACE()`.
     ```sql
     UPDATE layoffs_raw
     SET Company = UPPER(TRIM(Company)),
         Industry = UPPER(TRIM(Industry)),
         Country = UPPER(TRIM(Country));
     ```

### 3. Null Value Analysis & Handling  
   - Checked for null values and removed rows with missing values where analysis was not possible.
     ```sql
     DELETE FROM layoffs_raw
     WHERE Total_Laid_Off IS NULL OR Date IS NULL OR Industry IS NULL;
     ```

### 4. Data Staging Strategy  
   - Created a dedicated staging table (`layoffs_staging`) to ensure raw data remains untouched for future reference.
     ```sql
     CREATE TABLE layoffs_staging AS
     SELECT * FROM layoffs_raw;
     ```

---

## Exploratory Data Analysis (SQL)

Following the cleaning phase, a series of analytical SQL queries were executed to extract key trends and business insights:

### Key Analyses Conducted

- **Top Companies by Total Layoffs**
  ```sql
  SELECT Company, SUM(Total_Laid_Off) AS Total_Layoffs
  FROM layoffs_raw
  GROUP BY Company
  ORDER BY Total_Layoffs DESC
  LIMIT 10;

- **Highest Single event Layoffs**
  ```sql
  SELECT Company, Total_Laid_Off, Date
  FROM layoffs_raw
  ORDER BY Total_Laid_Off DESC
  LIMIT 1;
  
-**Countries with 100% workforce layoffs**
```sql
SELECT Company, Date, Total_Laid_Off
FROM layoffs_raw
WHERE Percentage_Laid_Off = 100;

-**Layoff by country,Location and Industry**
```sql
SELECT Country, Industry, SUM(Total_Laid_Off) AS Total_Layoffs
FROM layoffs_raw
GROUP BY Country, Industry
ORDER BY Total_Layoffs DESC;

-**Funding vs Layoff Corelation**
sql```
SELECT Funding_Stage, SUM(Total_Laid_Off) AS Total_Layoffs
FROM layoffs_raw
GROUP BY Funding_Stage
ORDER BY Total_Layoffs DESC;

-**Layoff Trends by Year**
```sql
SELECT YEAR(Date) AS Year, SUM(Total_Laid_Off) AS Total_Layoffs
FROM layoffs_raw
GROUP BY Year
ORDER BY Year;

## Key Insights & Observations

- The United States experienced the highest number of layoffs, followed by India, indicating that tech-centric economies were the most affected.

- The most significant layoffs occurred in the first quarter of 2023, followed by another major wave in 2020, highlighting patterns linked to post-pandemic economic shifts.

- A large number of layoffs were observed in the consumer and retail sectors, making them the hardest-hit industries during the downturn.

- Many of the affected companies were startups, suggesting that early-stage businesses struggled the most with market instability and unsustainable growth.

- Several companies laid off 100% of their workforce, which likely indicates complete shutdowns or acquisitions that led to workforce reductions.

- While funding is often viewed as a safety net, lack of funding does not appear to be the main factor. Several well-funded companies also conducted mass layoffs, pointing to flawed business models, poor management, or over-expansion.

- Companies in advanced funding stages or those nearing IPOs also faced significant layoffs, emphasizing that even late-stage startups were not immune to the broader economic pressures.

---

## Repository Structure

```
├── README.md               # Project overview and documentation  
├── 01_data_cleaning.sql    # SQL script for cleaning the raw dataset  
├── 02_eda_analysis.sql     # SQL script for performing EDA  
├── data/                   # Folder containing raw and cleaned datasets  
   └── layoffs_raw.csv  
             

