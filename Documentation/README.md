# SureLink: Short-Term Insurance Data Engineering Project

## Project Overview

SureLink is a data engineering project based on a South African short-term insurance business. It demonstrates how insurance data moves from a relational database into Microsoft Fabric, where it is cleaned, validated, transformed and prepared for reporting.

The project uses customer, vehicle, policy and claim data. It follows a layered data architecture to improve data quality and organise information for analysis.

The final data model supports reporting on insurance claims, claim amounts, policy information and data quality.

## Architecture Overview

![SureLink Architecture](architecture/SureLink_Architecture.png)

## Business Objective

Insurance businesses work with information from different operational areas. Poor data quality, missing values and inconsistent formats affect the reliability of reports and business decisions.

The objective of SureLink is to build a data pipeline that:

- Stores operational insurance data in Azure SQL Database.
- Ingests source data into Microsoft Fabric.
- Cleans and transforms data.
- Identifies data quality issues.
- Organises data into a dimensional model.
- Provides reporting through Power BI.

## Architecture

```text
Source Data
    |
    v
Azure SQL Database
    |
    v
Microsoft Fabric Dataflow Gen2
    |
    v
Bronze Lakehouse
    |
    v
Silver Lakehouse
    |
    v
Gold Lakehouse
    |
    v
Semantic Model
    |
    v
Power BI Reports
```

The architecture follows three main data layers:

| Layer | Purpose |
|---|---|
| Bronze | Stores data ingested from Azure SQL Database. |
| Silver | Cleans, standardises and validates the source data. |
| Gold | Organises transformed data into reporting tables. |

## Technologies Used

- Microsoft Azure SQL Database
- Microsoft Fabric
- Dataflow Gen2
- Fabric Lakehouse
- PySpark
- Python
- SQL and T-SQL
- Delta Lake
- Power BI
- Git and GitHub

## Data Pipeline

### 1. Azure SQL Database

The source data is stored in Azure SQL Database.

The database contains four main tables:

- Customer
- Vehicle
- Policy
- Claim

Python and pyodbc were used to load data into Azure SQL Database. The loading process used staging tables and SQL MERGE operations to support inserts and updates.

The final source database contains 78,296 records.

| Source table | Records |
|---|---:|
| Customer | 17,171 |
| Vehicle | 20,597 |
| Policy | 20,597 |
| Claim | 19,931 |
| Total | 78,296 |

### 2. Bronze Layer

Dataflow Gen2 connects to Azure SQL Database and imports the four source tables into the Bronze Lakehouse.

The Bronze layer retains the ingested data as the starting point for further processing.

### 3. Silver Layer

The Silver layer uses PySpark notebooks to clean and standardise the data.

The transformations include:

- Trimming unnecessary spaces from text fields.
- Converting blank text values into NULL.
- Standardising selected category fields.
- Converting email addresses to lowercase.
- Checking email formats.
- Identifying missing values.
- Checking for duplicate records.

The transformation process preserves missing values where business rules are not yet confirmed. For example, missing approved claim amounts are associated with claim statuses such as REJECTED and SUBMITTED.

### 4. Data Quality

Data quality checks identify issues in the Silver data.

The checks cover:

- Invalid customer email addresses.
- Missing customer cities.
- Missing vehicle makes.
- Missing vehicle models.
- Missing claim authority information.
- Negative extended liability limits.

The results are stored in a separate `DataQualityResults` table. This keeps quality monitoring information separate from the main business tables.

The project records the number of failed records, the rule name, severity, status and execution timestamp for each check.

### 5. Gold Layer

The Gold layer organises the data into a dimensional model for reporting.

It contains five tables:

| Table | Purpose |
|---|---|
| FactClaim | Stores claim-level measures and related insurance information. |
| DimCustomer | Stores customer details. |
| DimVehicle | Stores vehicle details. |
| DimPolicy | Stores policy details. |
| DimDate | Provides date information for time-based analysis. |

The FactClaim table contains 19,931 records. The model uses customer, vehicle, policy and date dimensions to support claim analysis.

### 6. Power BI Reporting

The Gold data model supports the SureLink insurance claims report.

The report contains three pages:

1. Claims Overview
2. Claims Analysis
3. Data Quality Monitoring

The report focuses on claim volumes, claim amounts, claim statuses, incident types, incident severity, provinces and data quality results.

## Key Data Validation Results

The final source data contains 78,296 records across four tables.

The Gold layer contains:

| Gold table | Records |
|---|---:|
| FactClaim | 19,931 |
| DimCustomer | 17,171 |
| DimVehicle | 20,597 |
| DimPolicy | 20,597 |
| DimDate | 389 |

Validation confirmed:

- No duplicate primary keys in the Gold dimension tables.
- No missing primary keys in the Gold dimension tables.
- No unmatched claim records for customer, vehicle, policy or date relationships.
- The FactClaim table retained the same record count as the source Claim table.

Data quality checks also identified records requiring further review. These findings are retained for monitoring rather than being changed without confirmed business rules.

## Repository Structure

```text
Surelink/
│
├── architecture/
├── Documentation/
│   ├── 01_Project_Overview.md
│   ├── 02_Architecture.md
│   ├── 03_Azure_SQL_Setup.md
│   ├── 04_Data_Ingestion_Bronze.md
│   ├── 05_Silver_Transformation.md
│   ├── 06_Data_Quality.md
│   ├── 07_Gold_Data_Model.md
│   ├── 08_PowerBI_Report.md
│   └── 09_Technical_Decisions_and_Challenges.md
│
├── Images/
├── notebooks/
├── Pipelines/
├── Report/
└── Surelink/
```

The folders contain the project materials for architecture, documentation, notebooks, pipelines and reporting. Refer to the individual documentation files for implementation details.

## Project Documentation

The `Documentation` folder contains detailed explanations of the project stages:

- [Project Overview](Documentation/01_Project_Overview.md)
- [Architecture](Documentation/02_Architecture.md)
- [Azure SQL Setup](Documentation/03_Azure_SQL_Setup.md)
- [Bronze Data Ingestion](Documentation/04_Data_Ingestion_Bronze.md)
- [Silver Transformations](Documentation/05_Silver_Transformation.md)
- [Data Quality](Documentation/06_Data_Quality.md)
- [Gold Data Model](Documentation/07_Gold_Data_Model.md)
- [Power BI Report](Documentation/08_PowerBI_Report.md)
- [Technical Decisions and Challenges](Documentation/09_Technical_Decisions_and_Challenges.md)

## Future Improvements

Possible future improvements include:

- Automating pipeline execution and scheduling.
- Adding monitoring and alerting for pipeline failures.
- Expanding data quality rules with confirmed business requirements.
- Improving report filtering and drill-through analysis.
- Exploring AI-based analysis for insurance claims and fraud indicators.

## Project Purpose

SureLink demonstrates practical work with Azure SQL Database, Microsoft Fabric, PySpark, data quality validation, dimensional modelling and Power BI.

The project also documents the technical decisions, validation steps and challenges encountered during development.