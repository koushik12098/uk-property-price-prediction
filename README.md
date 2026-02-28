#  Property Price Prediction — Big Data Pipeline

> **Module:** 7006SCN — Big Data Analytics  
> **Tools:** Apache Spark 3.5.1 (PySpark) · Python 3.12 · Jupyter Notebook  
> **Data Source:** [HM Land Registry — Price Paid Data (GOV.UK)](https://www.gov.uk/government/statistical-data-sets/price-paid-data-downloads)  
> **Licence:** Contains HM Land Registry data © Crown copyright and database right 2021. Licensed under the [Open Government Licence v3.0](http://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/)

---

##  Project Overview

This project builds a scalable big data pipeline to analyse and predict residential property sale prices across **England and Wales** using the full HM Land Registry Price Paid Dataset. The pipeline covers end-to-end data engineering, feature extraction, machine learning model training, and distributed processing performance analysis — all implemented with **Apache Spark (PySpark)** for large-scale processing.

The Price Paid Dataset is one of the UK's most authoritative open datasets, tracking all residential property sales lodged with HM Land Registry for registration since January 1995, covering over **30 million transactions**.

---

##  Project Structure

```
project/
├── notebooks/
│   ├── 1_data_ingestion.ipynb                        # Data loading & exploration
│   ├── 2_feature_engineering.ipynb                   # Feature extraction & transformation
│   ├── 3_model_engineering_notebook.ipynb            # ML model training & evaluation
│   └── 4_distributed_processing_and_scaling.ipynb   # Scalability & benchmarking
├── pp_parquet/                                       # Raw dataset (Parquet format)
├── pp_features/                                      # Processed ML-ready features (output)
├── checkpoints/                                      # Spark streaming checkpoints
└── README.md
```

---

##  About the Dataset

### Source
**HM Land Registry — Price Paid Data**  
🔗 https://www.gov.uk/government/statistical-data-sets/price-paid-data-downloads

The Price Paid Data tracks **all residential property sales in England and Wales** that are sold for value and registered with HM Land Registry. It is one of the most comprehensive and reliable sources of UK house price information, updated monthly on the 20th working day.

- **Coverage:** England and Wales
- **Time Period:** January 1995 to present
- **Total Records in this project:** 30,856,185 transactions
- **Data Category:** Category A (Standard — single residential properties sold for full market value) and Category B (Additional — repossessions, buy-to-lets, transfers to non-private individuals)
- **Format used:** Parquet (converted from original CSV)
- **Update Frequency:** Monthly

###  What the Dataset Excludes
- Sales not lodged with HM Land Registry
- Sales not made for value
- Leases of 7 years or less
- Properties in Scotland and Northern Ireland

---

##  Dataset Columns

| Column | Full Name | Description |
|---|---|---|
| `price` | Sale Price | Final sale price (£) as stated on the transfer deed (excl. VAT where applicable) |
| `date` | Date of Transfer | Date the sale was completed, as stated on the transfer deed |
| `postcode` | Postcode | UK postcode at time of original transaction |
| `prop_type` | Property Type | `D` = Detached, `S` = Semi-Detached, `T` = Terraced, `F` = Flats/Maisonettes, `O` = Other |
| `old_new` | Old or New | `Y` = Newly built property, `N` = Established residential building |
| `duration` | Duration / Tenure | `F` = Freehold, `L` = Leasehold, `U` = Unknown |
| `town` | Town/City | Town or city of the property (primary address identifier) |
| `district` | District | Local government district |
| `county` | County | County of the property |

###  Key Statistics (from this dataset)

| Metric | Value |
|---|---|
| Total Records | 30,856,185 |
| Mean Sale Price | £233,159 |
| Std Dev of Price | £955,949 |
| Min Sale Price | £1 |
| Max Sale Price | £900,000,000 |
| Distinct Property Types | 5 |
| Distinct Districts | 467 |
| Distinct Counties | 132 |

---

##  Pipeline Notebooks

### 1. Data Ingestion (`1_data_ingestion.ipynb`)
- Initialises a PySpark session (`local[*]`, 4g driver memory)
- Loads raw data from Parquet format (`pp_parquet/`)
- Explores schema, row/column counts, and data distributions
- Performs initial data quality assessment

### 2. Feature Engineering (`2_feature_engineering.ipynb`)
- Extracts `year` from the `date` timestamp column
- Drops the raw `date` column after extraction
- Applies `log1p` transformation to `price` → `log_price` to reduce right skew
- Encodes 5 categorical columns using `StringIndexer` + `OneHotEncoder`:
  - `prop_type`, `old_new`, `duration`, `district`, `county`
- Assembles all encoded features + `year` into a single `features` vector using `VectorAssembler`
- Final feature vector dimensionality: **610 features**
- Saves processed dataset to `pp_features/` in Parquet format

### 3. Model Engineering (`3_model_engineering_notebook.ipynb`)
- Loads ML-ready features from `pp_features/`
- Trains regression models using Spark MLlib to predict `log_price`
- Evaluates model performance (RMSE, R², MAE)
- Supports distributed training across all available cores

### 4. Distributed Processing & Scaling Analysis (`4_distributed_processing_and_scaling.ipynb`)
- Benchmarks pipeline performance against dataset size
- Analyses Spark job execution plans, partitioning strategy, and memory usage
- Measures throughput and latency at scale (30M+ rows)

---

##  Setup & Requirements

### System Prerequisites

| Requirement | Version |
|---|---|
| Java (OpenJDK) | 17 |
| Apache Spark | 3.5.1 |
| Python | 3.12.3 |
| Jupyter Notebook | Latest |

### Environment Variables
```bash
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
export SPARK_HOME=/home/<your_username>/spark
```

### Python Dependencies
```bash
pip install pyspark jupyter pandas numpy matplotlib
```

### Running the Pipeline
Run all notebooks **in order**:
```bash
jupyter notebook
```
1. `notebooks/1_data_ingestion.ipynb`
2. `notebooks/2_feature_engineering.ipynb`
3. `notebooks/3_model_engineering_notebook.ipynb`
4. `notebooks/4_distributed_processing_and_scaling.ipynb`

> **Note:** Update all absolute data paths (e.g. `/home/koushik/pp_parquet`) to match your local environment before running.

---

##  Data Licence & Attribution

This project uses data from HM Land Registry's Price Paid Data, released under the **Open Government Licence v3.0**.

**Required Attribution:**
> Contains HM Land Registry data © Crown copyright and database right 2021. This data is licensed under the Open Government Licence v3.0.

Under the OGL, you are permitted to use, adapt, and share the data for both commercial and non-commercial purposes, provided you include the above attribution. The OGL does not cover third-party rights.

 Full licence: http://www.nationalarchives.gov.uk/doc/open-government-licence/version/3/

---


