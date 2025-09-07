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

SQL Data Cleaning and Exploratory Data Analysis (EDA)
This project focuses on cleaning and analyzing a dataset of company layoffs from 2022. The dataset was obtained from Kaggle and initially contained various data quality issues, including duplicate entries, inconsistent text formatting, and missing values.

The primary tools used for this project are SQL for all data manipulation and analysis tasks.

Project Goals

## Data Cleaning:

Identify and remove duplicate rows.

Standardize text data to ensure consistency (e.g., correcting spelling, handling different formats).

Handle missing or null values appropriately.

Remove any unnecessary rows or columns.

## Exploratory Data Analysis (EDA):

Analyze trends in layoffs by company, industry, country, and location.

Identify the companies with the largest total layoffs.

Examine the highest percentages of layoffs to find companies that were completely shut down.

Calculate rolling totals of layoffs over time to understand the overall trend.

## Data Cleaning Process
The data cleaning process was structured into four key steps:

**Removing Duplicates**: A staging table was created to work with the data without altering the original table. Using a ROW_NUMBER() window function partitioned by key columns, duplicate rows were identified and then removed from the staging table.

**Standardizing Data**: Various standardization tasks were performed:

Leading and trailing spaces were removed.

Text inconsistencies, such as Crypto Currency and CryptoCurrency, were consolidated to a single format (Crypto).

The date column was converted from a string to a proper DATE data type for easier manipulation and time-based analysis.

**Handling Null Values**: Null values in the industry column were addressed by cross-referencing company names with other entries in the dataset to fill in missing information where possible. Null values in columns like total_laid_off and percentage_laid_off were kept as they were, as they represent meaningful missing data that would be useful for specific queries later on.

**Removing Unnecessary Data**: Rows where both total_laid_off and percentage_laid_off were NULL were removed, as they provided no useful information for analysis. A temporary row_num column, which was created during the duplicate removal step, was dropped to finalize the cleaned table.

## Key Findings from EDA
The company with the highest number of layoffs was Amazon.

Some startups had a 100% layoff rate (percentage_laid_off = 1), indicating they completely shut down.

The Consumer and Retail industries were among the most affected.

The data showed that layoffs were not always directly correlated with a company's funding, with some highly-funded companies still experiencing massive layoffs.

This project provides a robust framework for cleaning and analyzing similar datasets using SQL, transforming raw data into a clean, usable format for insightful analysis.

## SQL Queries
Here are some of the key SQL queries used in this project.

## **Finding Duplicates**
This query identifies duplicate rows based on multiple columns, assigning a row number to each partition. Duplicates will have a row_num greater than 1.

SELECT *,
		ROW_NUMBER() OVER (
			PARTITION BY company, location, industry, total_laid_off, percentage_laid_off, `date`, stage, country, funds_raised_millions
			) AS row_num
FROM world_layoffs.layoffs_staging2;

## **Standardizing Country Data**
This query standardizes the country column by removing trailing periods.

UPDATE layoffs_staging2
SET country = TRIM(TRAILING '.' FROM country);

## **Top 10 Companies with the Most Layoffs**
This query aggregates the total number of laid-off employees for each company and orders the results to show the top 10.

SELECT company, SUM(total_laid_off) AS total_layoffs
FROM world_layoffs.layoffs_staging2
GROUP BY company
ORDER BY 2 DESC
LIMIT 10;

## **Calculating Rolling Total Layoffs by Month**
This query calculates the cumulative sum of layoffs over time, providing a clear trend of the total number of layoffs month by month.

WITH DATE_CTE AS (
    SELECT SUBSTRING(date, 1, 7) as dates, SUM(total_laid_off) AS total_laid_off
    FROM layoffs_staging2
    GROUP BY dates
)
SELECT dates, SUM(total_laid_off) OVER (ORDER BY dates ASC) as rolling_total_layoffs
FROM DATE_CTE
ORDER BY dates ASC;


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
             

