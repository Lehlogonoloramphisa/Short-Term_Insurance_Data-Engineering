# SureLink: Data Ingestion into Microsoft Fabric Bronze

## 1. Introduction

After loading and validating the insurance data in Azure SQL Database, the next step was to import it into Microsoft Fabric.

The Bronze layer is the first storage layer in the Fabric data engineering process. It stores the data received from the source system before further cleaning and transformation.

For SureLink, Dataflow Gen2 was used to connect Azure SQL Database to a Fabric Lakehouse.

## 2. Microsoft Fabric Workspace

A Microsoft Fabric workspace was used to organise the project resources.

The workspace contains the Lakehouses, notebooks, Dataflow Gen2, semantic model and Power BI reports used throughout the project.

The following resources were created for the Bronze ingestion stage:

| Resource | Name |
|---|---|
| Lakehouse | `SureLink_Bronze_lh` |
| Dataflow Gen2 | `Surelink_Dataflow` |
| SQL analytics endpoint | `SureLink_Bronze_lh` |

The SQL analytics endpoint provides a way to query supported Lakehouse tables using SQL.

## 3. Connecting Azure SQL Database to Fabric

Dataflow Gen2 was used to establish a connection between the Azure SQL Database and Microsoft Fabric.

The source database was:

- Server: `surelink-sql-servers.database.windows.net`
- Database: `surelink-source`

The connection provided access to the insurance tables stored in Azure SQL Database.

Database credentials were used to authenticate the connection. They are not included in this documentation.

## 4. Configuring Dataflow Gen2

The Dataflow Gen2 named `Surelink_Dataflow` was created in the Fabric workspace.

The Azure SQL Database was selected as the source.

The source tables required for the project were selected:

1. `src.Customer`
2. `src.Vehicle`
3. `src.Policy`
4. `src.Claim`

System tables and the `LoadLog` table were excluded because they were not part of the insurance data required for the reporting model.

The selected tables were configured for loading into the `SureLink_Bronze_lh` Lakehouse.

## 5. Loading Data into the Bronze Lakehouse

After configuring the source and destination, the Dataflow Gen2 was executed.

The four insurance tables were imported into the Bronze Lakehouse.

The Bronze layer retained the ingested source records for use in the Silver transformation stage.

The data was not treated as fully cleaned or reporting-ready at this point. Further validation and transformation took place in the Silver layer.

## 6. Bronze Data Validation

After the Dataflow Gen2 completed, the SQL analytics endpoint was used to query the imported tables.

The record counts were compared with the corresponding Azure SQL Database tables.

### 6.1 Record Count Comparison

| Table | Azure SQL records | Bronze records |
|---|---:|---:|
| Customer | 17,171 | 17,171 |
| Vehicle | 20,597 | 20,597 |
| Policy | 20,597 | 20,597 |
| Claim | 19,931 | 19,931 |
| Total | 78,296 | 78,296 |

The counts matched across all four tables.

This confirmed that the Bronze Lakehouse contained the expected number of records from the Azure SQL source.

### 6.2 Why Record Count Validation Matters

Record count validation helps identify missing or unexpected records during data ingestion.

For example, if Azure SQL contains 19,931 claims but the Bronze Lakehouse contains 19,500, the difference indicates an ingestion issue requiring investigation.

Matching counts provide an initial check of the ingestion process. They do not, on their own, prove that every individual field value matches the source.

## 7. Challenges and Considerations

The ingestion process depended on a working connection between Azure SQL Database and Microsoft Fabric.

The connection also required valid authentication and access to the source database.

The source database firewall settings were relevant to connectivity during development.

The project excluded system tables and `LoadLog` from the ingestion process to keep the Bronze layer focused on the four insurance entities.

## 8. Outcome

The Bronze ingestion stage was completed successfully.

The `SureLink_Bronze_lh` Lakehouse contained the four imported insurance tables:

- Customer: 17,171 records.
- Vehicle: 20,597 records.
- Policy: 20,597 records.
- Claim: 19,931 records.

The total of 78,296 records matched the Azure SQL source database.

The Bronze layer provided the input for the next stage of the project: cleaning, standardisation and data quality checks in the Silver layer.

## 9. Evidence to Include

The following screenshots are useful evidence for this stage:

1. The Fabric workspace showing `SureLink_Bronze_lh` and `Surelink_Dataflow`.
2. The Dataflow Gen2 showing the Azure SQL source and selected tables.
3. The Dataflow Gen2 destination configuration showing the Bronze Lakehouse.
4. The Bronze Lakehouse Explorer showing the imported tables.
5. SQL query results showing the record counts for the four Bronze tables.

Before adding screenshots to GitHub, make sure they do not expose passwords, connection strings, access tokens or other credentials.