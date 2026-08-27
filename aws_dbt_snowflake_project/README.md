# 🏠 Airbnb End-to-End Data Engineering Project

## 📋 Overview

This project demonstrates an end-to-end data engineering pipeline for Airbnb data using **AWS S3, Snowflake, dbt, Python, and Git**.

The pipeline follows a **Medallion Architecture (Bronze → Silver → Gold)** to transform raw Airbnb data into clean, standardized, and analytics-ready datasets.

The project demonstrates practical data engineering concepts including:

- Data ingestion and staging
- Incremental data processing
- SQL-based transformations
- Dimensional modeling
- One Big Table (OBT) modeling
- SCD Type 2 historical tracking
- Custom dbt Jinja macros
- Data quality testing
- Source definitions
- dbt documentation and lineage
- Git/GitHub version control

---

## 🏗️ Architecture

### Data Flow
```
Source Data (CSV) → AWS S3 → Snowflake (Staging) → Bronze Layer → Silver Layer → Gold Layer
                                                           ↓              ↓           ↓
                                                      Raw Tables    Cleaned Data   Analytics
```

The pipeline separates data into multiple layers so that raw, cleaned, and business-ready data can be managed independently.

---

## 🛠️ Technology Stack

| Technology | Purpose |
|------------|---------|
| **AWS S3** | Cloud data storage |
| **Snowflake** | Cloud data warehouse |
| **dbt** | Data transformation and modeling |
| **SQL** | Data transformation and analysis |
| **Python** | Project utilities |
| **Jinja** | Dynamic SQL and reusable dbt macros |
| **Git** | Version control |
| **GitHub** | Source code management |

---

## 📊 Data Model

### Medallion Architecture

## 🥉 Bronze Layer — Raw Data

The Bronze layer contains Airbnb data loaded from the Snowflake staging layer with minimal transformation.

### Models

- `bronze_bookings`
- `bronze_hosts`
- `bronze_listings`

### Incremental Processing

The `bronze_bookings` model uses dbt incremental materialization:

```sql
{{ config(materialized='incremental') }}

SELECT *
FROM {{ source('staging', 'bookings') }}

{% if is_incremental() %}

WHERE CREATED_AT >
(
    SELECT COALESCE(MAX(CREATED_AT), '1900-01-01')
    FROM {{ this }}
)

{% endif %}
```

During incremental runs, the model compares incoming records against the latest processed `CREATED_AT` value and processes only newer records.

This reduces unnecessary processing when working with larger datasets.

---

# 🥈 Silver Layer — Cleaned Data

The Silver layer cleans and standardizes the Bronze data before it is consumed by downstream analytical models.

### Models

- `silver_bookings`
- `silver_hosts`
- `silver_listings`

Transformations include:

- Data cleaning
- Standardization
- Derived columns
- Business logic
- Data quality improvements
- Price categorization

The Silver layer acts as the foundation for downstream analytical models.

---

# 🥇 Gold Layer — Analytics Ready

The Gold layer contains business-ready datasets designed for analytical workloads.

### Models

```text
gold/
├── fact.sql
├── obt.sql
└── ephemeral/
    ├── bookings.sql
    ├── hosts.sql
    └── listings.sql
```

### Fact Model

The `fact` model provides a fact-oriented structure for analytical workloads.

### OBT Model

The `obt` model follows a **One Big Table (OBT)** approach by combining relevant booking, listing, and host information into a denormalized analytical dataset.

### Ephemeral Models

Ephemeral models are used for intermediate transformations that do not need to be persisted as physical tables.

---

# 📸 Snapshots — SCD Type 2

The project uses **dbt snapshots** to preserve historical versions of changing records.

Snapshots include:

- `dim_bookings`
- `dim_hosts`
- `dim_listings`

The snapshots use a timestamp-based strategy.

Example:

```yaml
snapshots:
  - name: dim_bookings
    relation: ref('bookings')

    config:
      schema: gold
      database: AIRBNB
      unique_key: BOOKING_ID
      strategy: timestamp
      updated_at: CREATED_AT
      dbt_valid_to_current: "to_date('9999-12-31')"
```

This allows historical changes to be tracked instead of simply overwriting previous values.

The resulting historical records can be used for point-in-time analysis.

---

# 🧩 Custom dbt Macros

The project includes reusable Jinja macros to reduce repetitive SQL logic.

### Macro Structure

```text
macros/
├── generate_schema_name.sql
├── multiply.sql
├── tag.sql
└── trimmer.sql
```

## `generate_schema_name`

Customizes how dbt generates schemas for models.

This allows the project to organize models into separate database schemas such as:

```text
AIRBNB.BRONZE
AIRBNB.SILVER
AIRBNB.GOLD
```

---

## `multiply`

Provides reusable multiplication logic:

```sql
{% macro multiply(x, y, precision) %}
    {{ x }} * {{ y }} * {{ precision }}
{% endmacro %}
```

---

## `tag`

Categorizes values into different levels:

```sql
CASE
    WHEN {{ col }} < 100 THEN 'low'
    WHEN {{ col }} < 200 THEN 'medium'
    ELSE 'high'
END
```

This can be reused across dbt models instead of repeatedly writing the same SQL logic.

---

## `trimmer`

Provides reusable trimming and uppercase transformation:

```sql
{{ col_name | trim | upper }}
```

These macros demonstrate how repetitive transformation logic can be abstracted using Jinja.

---

# 🧪 Data Quality Testing

The project includes custom SQL-based data quality testing.

### Test Location

```text
tests/
└── source_tests.sql
```

Example business rule:

```sql
{{ config(
    severity = 'warn'
) }}

SELECT
    1
FROM {{ source('staging', 'bookings') }}
WHERE BOOKING_AMOUNT < 200
```

The test identifies booking records that violate the defined business rule.

The test uses:

```yaml
severity: warn
```

This allows potential data-quality issues to be reported without necessarily stopping the entire dbt execution.

---

# 🔗 Source Definitions

The project uses dbt source definitions to represent upstream Snowflake staging tables.

Example source usage:

```sql
{{ source('staging', 'bookings') }}
```

This provides clearer dependencies between source data and downstream dbt models and allows dbt to track lineage.

---

## 📁 Project Structure

```
AWS_DBT_Snowflake/
├── README.md                           # This file
├── pyproject.toml                      # Python dependencies
├── main.py                             # Main execution script
│
├── SourceData/                         # Raw CSV data files
│   ├── bookings.csv
│   ├── hosts.csv
│   └── listings.csv
│
├── DDL/                                # Database schema definitions
│   ├── ddl.sql                         # Table creation scripts
│   └── resources.sql
│
└── aws_dbt_snowflake_project/         # Main dbt project
    ├── dbt_project.yml                 # dbt project configuration
    ├── ExampleProfiles.yml             # Snowflake connection profile
    │
    ├── models/                         # dbt models
    │   ├── sources/
    │   │   └── sources.yml             # Source definitions
    │   ├── bronze/                     # Raw data layer
    │   │   ├── bronze_bookings.sql
    │   │   ├── bronze_hosts.sql
    │   │   └── bronze_listings.sql
    │   ├── silver/                     # Cleaned data layer
    │   │   ├── silver_bookings.sql
    │   │   ├── silver_hosts.sql
    │   │   └── silver_listings.sql
    │   └── gold/                       # Analytics layer
    │       ├── fact.sql
    │       ├── obt.sql
    │       └── ephemeral/              # Temporary models
    │           ├── bookings.sql
    │           ├── hosts.sql
    │           └── listings.sql
    │
    ├── macros/                         # Reusable SQL functions
    │   ├── generate_schema_name.sql    # Custom schema naming
    │   ├── multiply.sql                # Math operations
    │   ├── tag.sql                     # Categorization logic
    │   └── trimmer.sql                 # String utilities
    │
    ├── analyses/                       # Ad-hoc analysis queries
    │   ├── explore.sql
    │   ├── if_else.sql
    │   └── loop.sql
    │
    ├── snapshots/                      # SCD Type 2 configurations
    │   ├── dim_bookings.yml
    │   ├── dim_hosts.yml
    │   └── dim_listings.yml
    │
    ├── tests/                          # Data quality tests
    │   └── source_tests.sql
    │
    └── seeds/                          # Static reference data
```

---

# 🚀 Getting Started

## Prerequisites

Before running the project, make sure you have:

- Python 3.12+
- A Snowflake account
- dbt Core
- dbt Snowflake adapter
- Git
- AWS account if using S3 for source storage

---

## 1. Clone the Repository

```bash
git clone <repository-url>

cd AWS_DBT_Snowflake
```

---

## 2. Create a Virtual Environment

### Windows PowerShell

```powershell
python -m venv .venv

.venv\Scripts\Activate.ps1
```

### Linux / macOS

```bash
python -m venv .venv

source .venv/bin/activate
```

---

## 3. Install Dependencies

Using pip:

```bash
pip install -e .
```

Or install the required dbt packages:

```bash
pip install dbt-core
pip install dbt-snowflake
```

---

# ❄️ Snowflake Configuration

Configure the Snowflake connection in:

```text
~/.dbt/profiles.yml
```

Example:

```yaml
aws_dbt_snowflake_project:

  outputs:

    dev:
      type: snowflake
      account: <your-account-identifier>
      user: <your-username>
      password: <your-password>

      role: <your-role>
      database: AIRBNB
      schema: dbt_schema
      warehouse: COMPUTE_WH

      threads: 4

  target: dev
```

> ⚠️ Never commit your `profiles.yml` or Snowflake credentials to GitHub.

Use environment variables or another secure credentials-management approach for real projects.

---

# ▶️ Running the Project

Navigate to the dbt project:

```bash
cd aws_dbt_snowflake_project
```

## Test Snowflake Connection

```bash
dbt debug
```

---

## Install dbt Dependencies

```bash
dbt deps
```

---

## Run All Models

```bash
dbt run
```

---

## Run Bronze Models

```bash
dbt run --select bronze.*
```

---

## Run Silver Models

```bash
dbt run --select silver.*
```

---

## Run Gold Models

```bash
dbt run --select gold.*
```

---

## Run Data Quality Tests

```bash
dbt test
```

---

## Run Snapshots

```bash
dbt snapshot
```

---

## Build Models and Tests

```bash
dbt build
```

---

## Generate dbt Documentation

```bash
dbt docs generate
```

Then:

```bash
dbt docs serve
```

dbt documentation can be used to explore model relationships, dependencies, and data lineage.

---

# ⚡ Key Engineering Concepts Demonstrated

## 1. Incremental Processing

Incremental models reduce unnecessary processing by loading only records that are newer than the latest processed timestamp.

```text
Initial Run
     │
     ▼
Process Existing Records
     │
     ▼
Store Latest CREATED_AT
     │
     ▼
Future Run
     │
     ▼
Process New Records Only
```

---

## 2. Medallion Architecture

The project separates data into:

```text
Bronze
  ↓
Raw / minimally transformed data

Silver
  ↓
Cleaned and standardized data

Gold
  ↓
Analytics-ready data
```

This improves organization, maintainability, and downstream data consumption.

---

## 3. SCD Type 2

dbt snapshots preserve historical versions of changing records.

Instead of:

```text
Old Value → overwritten
```

the pipeline maintains:

```text
Old Version → Historical Record
New Version → Current Record
```

This enables historical analysis of changes in bookings, hosts, and listings.

---

## 4. Reusable Jinja Macros

Custom macros allow frequently used SQL logic to be written once and reused across multiple models.

This improves:

- Maintainability
- Reusability
- Consistency
- Developer productivity

---

## 5. Dimensional Modeling

The Gold layer contains analytical structures including:

- Fact models
- One Big Table (OBT)
- Ephemeral intermediate models

These structures support downstream analytical workloads.

---

## 6. Data Quality

Custom dbt tests are used to identify records that violate defined business rules.

This helps detect data issues before they affect downstream analysis.

---

# 🔐 Security & Best Practices

The project follows several data engineering best practices:

### Credentials

- Do not commit Snowflake credentials.
- Keep `profiles.yml` outside the repository.
- Use environment variables or secure credential management.

### Version Control

- Git is used to track project changes.
- GitHub is used to maintain the project source code.

### Transformation

- dbt is used to manage SQL transformations.
- Reusable Jinja macros reduce duplicated SQL.
- Incremental processing reduces unnecessary computation.

### Data Organization

- Bronze, Silver, and Gold layers separate different stages of data processing.
- Snowflake schemas provide logical separation between layers.

---

# 📈 Future Enhancements

Potential improvements for the project include:

- Add automated CI/CD using GitHub Actions
- Add workflow orchestration using Apache Airflow
- Add automated data quality monitoring
- Build a Power BI or Tableau dashboard
- Add more advanced business metrics
- Add pipeline monitoring and alerting
- Improve automated AWS S3 → Snowflake ingestion
- Add more comprehensive dbt tests
- Add documentation for individual models and columns

---

# 📚 Technologies & Concepts

```text
Cloud
├── AWS S3
└── Snowflake

Data Transformation
├── dbt
├── SQL
└── Jinja

Data Engineering
├── Medallion Architecture
├── Incremental Processing
├── SCD Type 2
├── Dimensional Modeling
├── OBT Modeling
└── Data Quality Testing

Development
├── Python
├── Git
└── GitHub
```

---

# 👤 Author

**Chiranjeev Singh**

### Project

**Airbnb End-to-End Data Engineering Pipeline**

### Technologies

**AWS S3 • Snowflake • dbt • SQL • Python • Git**

---

## ⭐ Project Summary

This project demonstrates how raw Airbnb data can be transformed into analytics-ready datasets using a modern cloud data engineering stack.

The implementation focuses on practical engineering concepts such as:

**AWS → Snowflake → dbt → Bronze → Silver → Gold → Analytics**

with additional capabilities including:

**Incremental Processing • SCD Type 2 • Custom Macros • Data Quality Testing • Dimensional Modeling • Data Lineage**
