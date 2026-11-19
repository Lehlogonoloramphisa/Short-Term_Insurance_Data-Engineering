# SureLink: Insurance Data Engineering Platform

## 1. Project Overview

SureLink is a South African short-term insurance data engineering project. It demonstrates how insurance data from a relational database can be ingested, cleaned, transformed and organised for reporting and analysis using Microsoft Azure and Microsoft Fabric.

The project uses four main data entities:

- Customer
- Vehicle
- Policy
- Claim

The data moves through different stages, starting in Azure SQL Database and continuing through the Bronze, Silver and Gold layers in Microsoft Fabric.

The final Gold layer contains structured tables designed to support insurance reporting and business analysis. A Power BI report presents information about claims, claim amounts, claim statuses, incident types, provinces and data quality.

## 2. Project Objectives

The main objectives are to:

1. Set up an Azure SQL Database to store the source insurance data.
2. Load customer, vehicle, policy and claim records into Azure SQL Database.
3. Connect Microsoft Fabric to the Azure SQL Database.
4. Use Dataflow Gen2 to ingest source tables into the Bronze Lakehouse.
5. Clean and standardise data in the Silver layer using PySpark.
6. Run data quality checks and record their results.
7. Build dimensional tables and a fact table in the Gold layer.
8. Create a semantic model for reporting.
9. Develop a Power BI dashboard to analyse insurance claims and monitor data quality.

## 3. Technologies Used

| Technology | Purpose |
|---|---|
| Microsoft Azure SQL Database | Stores the source insurance data |
| Python | Loads source data into Azure SQL Database |
| SQL and T-SQL | Database queries, validation and data checks |
| pyodbc | Connects Python to Azure SQL Database |
| Microsoft Fabric | Provides the data engineering environment |
| Dataflow Gen2 | Imports source tables into the Bronze Lakehouse |
| Fabric Lakehouse | Stores data across the Bronze, Silver and Gold layers |
| PySpark | Cleans, transforms and validates data |
| Delta Lake | Stores the processed Lakehouse tables |
| Power BI | Presents insurance and data quality reports |
| Git and GitHub | Supports project version control and portfolio documentation |

## 4. Project Architecture

The project follows a layered data architecture.

### Source Layer: Azure SQL Database

The source database stores four insurance data tables: Customer, Vehicle, Policy and Claim.

Python and pyodbc were used to load the source records into Azure SQL Database. SQL queries were used to verify record counts after loading.

### Bronze Layer: Raw Data Ingestion

The Bronze Lakehouse receives the source tables from Azure SQL Database through a Fabric Dataflow Gen2.

The data is stored in the Bronze layer before further transformation.

### Silver Layer: Data Cleaning and Validation

The Silver layer contains cleaned and standardised versions of the source tables.

PySpark notebooks were used to:

- Trim unnecessary spaces from text fields.
- Convert blank text values into NULL.
- Standardise selected category values.
- Check for missing values.
- Check for duplicate records.
- Validate email formats.
- Identify data quality issues.

Data quality results are stored separately in a `DataQualityResults` table.

### Gold Layer: Dimensional Data Model

The Gold layer contains reporting-ready tables:

- `FactClaim`
- `DimCustomer`
- `DimVehicle`
- `DimPolicy`
- `DimDate`
- `DataQualityResults`

The fact table stores claim-level information. The dimension tables provide descriptive information used to analyse claims.

### Reporting Layer: Power BI

A semantic model connects the Gold tables and defines their relationships.

The Power BI report contains three pages:

1. Claims Overview
2. Claims Analysis
3. Data Quality Monitoring

These pages provide a view of claim volumes, claim amounts, claim statuses, incident patterns and data quality results.

## 5. Source Data Summary

The source database contains the following records after loading all three data batches.

| Source table | Records |
|---|---:|
| Customer | 17,171 |
| Vehicle | 20,597 |
| Policy | 20,597 |
| Claim | 19,931 |
| Total | 78,296 |

The source tables were checked after loading to confirm the expected records were present.

## 6. Project Scope

This project focuses on building a data engineering workflow from source database ingestion through to reporting.

The completed work includes:

- Azure SQL Database setup and data loading.
- Fabric Dataflow Gen2 ingestion.
- Bronze, Silver and Gold Lakehouse layers.
- PySpark transformations.
- Data quality checks.
- Dimensional modelling.
- Power BI reporting.

An AI extension is a possible future development. It is not included as a completed project feature.

## 7. Business Value

The platform organises insurance data into a structure suitable for analysis.

The reporting layer helps users examine:

- Claim volumes and claim statuses.
- Claim amounts and approved amounts.
- Claims across provinces.
- Different incident types and severity levels.
- Monthly claim trends.
- Data quality issues requiring further investigation.

These insights support business reporting and provide information for decision-making. Data quality results also help identify records requiring attention before the data is used for analysis.

## 8. Project Status

The Azure SQL source database, Fabric ingestion, Bronze, Silver and Gold layers, semantic model and Power BI report have been developed.

The project remains a portfolio demonstration. It does not represent a live production insurance system.