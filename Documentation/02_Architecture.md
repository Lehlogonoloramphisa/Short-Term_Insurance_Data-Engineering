# SureLink: Data Architecture

## 1. Architecture Overview

SureLink follows a layered data architecture. Data starts in Azure SQL Database and moves through Microsoft Fabric before reaching the reporting layer.

The architecture separates data ingestion, data cleaning, data modelling and reporting. Each stage has a specific purpose.

## 2. Architecture Flow

```text
             SOURCE DATABASE
          Microsoft Azure SQL
                    |
                    |
              Dataflow Gen2
                    |
                    v
             BRONZE LAYER
          SureLink_Bronze_lh
          Raw ingested tables
                    |
                    |
                    v
              SILVER LAYER
          SureLink_Silver_lh
          Cleaned and standardised
          data using PySpark
                    |
                    |
             Data Quality Checks
                    |
                    v
               GOLD LAYER
           SureLink_Gold_lh
           Reporting-ready tables
                    |
                    |
                    v
             SEMANTIC MODEL
          SureLink_Semantic_Model
                    |
                    |
                    v
                POWER BI
          Insurance Claims Dashboard
```

The data quality results are also stored in a separate `DataQualityResults` table. This table supports the Data Quality Monitoring report page.

## 3. Source Layer: Azure SQL Database

The source database is hosted in Microsoft Azure SQL Database.

Database name: `surelink-source`

The database contains four main tables:

| Table | Description |
|---|---|
| Customer | Customer information |
| Vehicle | Vehicle information |
| Policy | Insurance policy information |
| Claim | Insurance claim information |

Python was used to load the source records into Azure SQL Database. The `pyodbc` library provided the connection between Python and SQL Server.

The loading process used staging tables and SQL MERGE statements to insert new records and update existing records.

After loading, SQL queries were used to verify the number of records in each table.

Total source records: 78,296.

## 4. Bronze Layer: Data Ingestion

The Bronze layer is the first storage layer in Microsoft Fabric.

Lakehouse: `SureLink_Bronze_lh`

Dataflow Gen2: `Surelink_Dataflow`

The Dataflow Gen2 connects to the Azure SQL Database and imports the four source tables:

- Customer
- Vehicle
- Policy
- Claim

The Bronze layer keeps the ingested source data available for the next transformation stage.

The record counts were checked against Azure SQL Database after ingestion.

| Table | Bronze records |
|---|---:|
| Customer | 17,171 |
| Vehicle | 20,597 |
| Policy | 20,597 |
| Claim | 19,931 |
| Total | 78,296 |

The matching counts confirmed that all four source tables were ingested with the expected number of records.

## 5. Silver Layer: Data Transformation

Lakehouse: `SureLink_Silver_lh`

Notebook: `SureLink_Silver_Transformation_nb`

The Silver layer contains cleaned and standardised data. PySpark was used to process the Bronze tables.

The transformations included:

- Removing unnecessary spaces from text values.
- Converting blank text values into NULL.
- Converting email addresses to lowercase.
- Standardising selected category values.
- Checking for missing values.
- Checking for duplicate primary keys.
- Checking email formats.
- Investigating unusual values.

Data quality observations were retained where business rules were not available to confirm whether a value was incorrect.

For example, 21 policies contained a negative extended liability limit of -1,000,000. These values were preserved rather than changed without confirmation of the correct business rule.

The Silver layer provides the cleaned data used to build the Gold layer.

## 6. Data Quality Monitoring

Data quality checks were developed to identify issues in the processed data.

The results are stored in the Silver `DataQualityResults` table.

Each result records information such as:

- Table name.
- Column name.
- Rule name.
- Number of failed records.
- Severity.
- Pass or fail status.
- Run timestamp.
- Total records checked.

The results are copied into the Gold Lakehouse through a separate registration notebook. This provides a table for the semantic model and Power BI report.

The Gold copy is a snapshot. It must be refreshed when the Silver quality results change.

## 7. Gold Layer: Dimensional Data Model

Lakehouse: `SureLink_Gold_lh`

The Gold layer contains tables structured for reporting and analysis.

It follows a dimensional modelling approach. This means the data is organised into fact and dimension tables.

### Fact table

`FactClaim`

The fact table contains one row per insurance claim. It combines claim information with related policy details.

It includes fields such as:

- Claim ID.
- Policy ID.
- Customer ID.
- Vehicle ID.
- Incident date.
- Total claim amount.
- Approved amount.
- Claim status.
- Incident type.
- Incident severity.
- Incident province.
- Cover level.
- Annual premium.

The table contains 19,931 records, matching the Silver Claim record count.

### Dimension tables

| Table | Purpose |
|---|---|
| DimCustomer | Provides customer details |
| DimVehicle