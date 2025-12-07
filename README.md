# Global Air Quality Analysis Using PySpark

## Overview

This project builds a data pipeline in PySpark to analyse global air quality. It combines WHO city-level pollution measurements (PM2.5, PM10, NO2) with World Bank population data to compute a Pollution Index and a population-weighted Health Risk Index, allowing cross-country and temporal comparisons of air pollution and population exposure.

The project was completed as part of an MSc Cloud Computing assignment using Apache Spark.

---

## Repository Structure

```text
pyspark-air-quality-analysis/
├── Datasets/
│   ├── WHO Dataset.xlsx          # WHO Ambient Air Quality Database (2022 release)
│   └── POPULATION Dataset.csv    # World Bank country-level population data
├── outputs/
│   ├── bubble_chart.png          # Top 10 countries: Pollution Index vs Health Risk
│   ├── trend_chart.png           # Pollution trends 2011-2018 for top 5 countries
│   └── pm_bar_chart.png          # PM2.5 / PM10 / NO2 breakdown by country
├── Global Air Quality Analysis Using PySpark.ipynb
├── REPORT - Global Air Quality Analysis Using Spark.pdf
├── requirements.txt
└── README.md
```

---

## Visualisations

### Pollution Index vs Health Risk Index (Top 10 Countries)

![Bubble Chart](outputs/bubble_chart.png)

### Pollution Trends 2011-2018 (Top 5 Countries by Health Risk)

![Trend Chart](outputs/trend_chart.png)

### PM Levels by Country (All Years)

![PM Bar Chart](outputs/pm_bar_chart.png)

---

## Datasets

### WHO Ambient Air Quality Database

- Source: World Health Organization
- Format: Excel (.xlsx), sheet `AAP_2022_city_v9`
- Granularity: City-level measurements per year
- Key columns: country, city, measurement year, PM2.5, PM10, NO2 (µg/m3)
- Note: significant missingness in PM2.5 values, especially for earlier years

### World Bank Population Dataset

- Source: World Bank Open Data
- Format: CSV
- Granularity: Country-level population totals per year
- Key columns: country name, ISO code, year, population

---

## Pipeline

The full pipeline runs in a single Jupyter notebook across 8 steps.

**Step 1 - Data Ingestion**
The WHO Excel file is read with pandas using sheet `AAP_2022_city_v9` (the default tab is a README sheet and returns empty columns). Written to a temp CSV then loaded into Spark. Population data loaded directly from CSV.

**Step 2 - Population Cleaning**
Column names lowercased and renamed. String columns trimmed and lowercased to match the WHO naming convention before joining.

**Step 3 - Merge**
WHO country names are standardised against a 23-entry lookup dict to fix known mismatches between WHO and World Bank naming conventions. Left join on standardised country name + year keeps all 32,196 WHO rows.

**Step 4 - Validation**
Pollutant columns renamed to `pm25`, `pm10`, `no2`. Null rates, duplicate count, and value ranges checked. PM2.5 is around 30% null and NO2 around 53% null, which is normal for city-level monitoring data.

**Step 5 - Cleaning and Imputation**
Three duplicates removed. Missing population and pollutant values filled with per-country medians using PySpark window functions. Final row count: 30,247.

**Step 6 - Index Calculation**
Pollution Index: coverage-adjusted average of available pollutants per row. Health Risk Index: `(Pollution Index × Population) / 1,000,000,000`. Pollutants categorised Low / Medium / High against WHO thresholds.

**Step 7 - Final Check**
`describe()` and null counts on key columns confirm the dataset is clean.

**Step 8 - Visualisations**
Bubble chart (Pollution Index vs Health Risk, top 10 countries), line chart (Pollution Index trends 2011–2018, top 5 by health risk), stacked bar chart (cumulative PM breakdown, top 10 by PM2.5). All saved to `outputs/`.

---

## How to Run

```bash
pip install -r requirements.txt
jupyter notebook "Global Air Quality Analysis Using PySpark.ipynb"
```

Run all cells from top to bottom. The three output charts will be saved to `outputs/`.

Java 8 or above is required for PySpark to run locally.

---

## Technologies

- Apache Spark (PySpark) for distributed data processing
- pandas and openpyxl for Excel ingestion
- matplotlib and numpy for visualisation
- Jupyter Notebook as the development environment

---

## Key Results

- Pakistan and Bangladesh have the highest Health Risk Index scores, a product of high PM readings and large populations.
- China has the largest cumulative PM2.5 total across all monitored years.
- India has the largest cumulative PM10 total.
- Bangladesh shows a spike in the Pollution Index around 2013–2014 followed by a decline through 2018.
- India is the only top-5 country with a consistently rising Pollution Index across 2011–2018.

---

## Contributors

| Name | DCU Student ID | Email |
| --- | --- | --- |
| Achal Nanjundamurthy | A00050840 | [achalnm02@gmail.com](mailto:achalnm02@gmail.com) |
| Tejas Shiva Kumar | A00050674 | [tejasshivakumar104@gmail.com](mailto:tejasshivakumar104@gmail.com) |
