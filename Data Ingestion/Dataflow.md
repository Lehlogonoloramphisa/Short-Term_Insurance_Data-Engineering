# SureLink Data Ingestion Pipeline

## Overview

SureLink uses Microsoft Fabric Dataflow Gen2 to ingest insurance data from Azure SQL Database into the Bronze Lakehouse.

The Dataflow Gen2 process connects to the Azure SQL source database, reads the required tables and loads them into Microsoft Fabric.

## Source System

The source database is hosted in Azure SQL Database.

Database: `surelink-source`

Source tables:

- Customer
- Vehicle
- Policy
- Claim

## Destination

The destination is the Microsoft Fabric Bronze Lakehouse.

Lakehouse: `SureLink_Bronze_lh`

## Dataflow Configuration

Dataflow name: `Surelink_Dataflow`

The Dataflow Gen2 connects to Azure SQL Database and retrieves the four insurance tables.

The following tables are included:

| Source table | Bronze destination |
|---|---|
| Customer | Customer |
| Vehicle | Vehicle |
| Policy | Policy |
| Claim | Claim |

System tables and the LoadLog table are excluded from ingestion.

## Ingestion Process

1. Connect Dataflow Gen2 to the Azure SQL Database.
2. Select the required insurance tables.
3. Exclude tables not required for the analytical dataset.
4. Configure the destination as the `SureLink_Bronze_lh` Lakehouse.
5. Load the source tables into the Bronze layer.
6. Validate the imported record counts against Azure SQL Database.

## Validation Results

The ingestion process loaded 78,296 records into the Bronze Lakehouse.

| Table | Azure SQL records | Bronze records |
|---|---:|---:|
| Customer | 17,171 | 17,171 |
| Vehicle | 20,597 | 20,597 |
| Policy | 20,597 | 20,597 |
| Claim | 19,931 | 19,931 |
| Total | 78,296 | 78,296 |

The matching record counts confirmed that all four source tables were imported successfully.

## Role in the Architecture

The Dataflow Gen2 handles the ingestion stage of the project.

The Bronze data is then used by the Silver transformation notebooks. These notebooks clean and standardise the data before the Gold dimensional model is created.

## Related Documentation

- [Azure SQL Setup](../Documentation/03_Azure_SQL_Setup.md)
- [Bronze Data Ingestion](../Documentation/04_Data_Ingestion_Bronze.md)
- [Silver Transformation](../Documentation/05_Silver_Transformation.md)