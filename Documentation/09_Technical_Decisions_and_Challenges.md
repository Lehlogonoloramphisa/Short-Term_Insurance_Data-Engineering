# SureLink: Technical Decisions and Development Challenges

## 1. Introduction

During the development of SureLink, several technical challenges required investigation and changes to the implementation.

This document records the main issues encountered across Azure SQL Database, Python, Microsoft Fabric, PySpark, Delta Lake and Power BI.

It also explains important decisions made to protect data accuracy and maintain consistency throughout the project.

## 2. Azure SQL Database Connection

### Challenge

Python initially experienced connection problems when accessing Azure SQL Database.

The database server uses firewall rules to control which public IP addresses are allowed to connect.

When the development machine's public IP address changed, the existing firewall rule no longer allowed the connection.

### Solution

The current public IP address was added to the Azure SQL server firewall rules.

The Python connection was tested again using `pyodbc`.

### Technical Learning

Cloud databases require both valid authentication and network access. A correct username and password are not enough when firewall rules block the connection.

## 3. Python Batch Loading and String Truncation

### Challenge

The Python loading script initially experienced string truncation errors when using `fast_executemany`.

This affected the batch insertion process because some text values did not fit the input sizes being used by the ODBC driver.

### Solution

The script was updated to use:

`cursor.setinputsizes()`

The input sizes specified the SQL text data type and appropriate column lengths.

The batch loading process was then tested again with `fast_executemany=True`.

### Technical Learning

Batch loading performance settings need to match the data types and column sizes in the target database.

## 4. Handling Repeated Data Loads

### Challenge

The source data was loaded in multiple batches. The process needed to distinguish new records from existing records.

Inserting every incoming record without checking existing data would create duplicate records.

### Solution

Staging tables and SQL MERGE statements were used.

The process inserted records when no matching key existed and updated existing records when a match was found.

This approach supported the initial load and subsequent batches.

### Technical Learning

Staging tables provide a controlled area for incoming data. MERGE statements support insert and update operations based on matching keys.

## 5. PySpark Column Resolution Error

### Challenge

During the Fabric data engineering work, a PySpark notebook encountered an `AnalysisException` related to an unresolved `OrderDate` column.

The error indicated that Spark could not find the requested column in the DataFrame schema.

### Solution and Learning

The DataFrame schema and available column names needed to be inspected before referencing the column.

This reinforced the importance of checking source schemas rather than assuming column names from an expected structure.

The SureLink transformations were based on the actual Customer, Vehicle, Policy and Claim schemas.

## 6. Preserving Unusual Policy Values

### Challenge

During Policy validation, 21 records contained an `extended_liability_limit` value of -1,000,000.

All 21 records belonged to THIRD PARTY policies.

The available project information did not confirm whether this value represented an invalid record or a specific business condition.

### Decision

The values were preserved.

They were identified as a data quality issue instead of being replaced with zero or another assumed value.

### Reason

Changing the values without a confirmed business rule would introduce an assumption into the data.

The issue was retained for further investigation.

## 7. Handling Missing Claim Values

### Challenge

The Claim table contained missing values in `approved_amount` and `authorities_contacted`.

The missing approved amounts were associated with REJECTED, SUBMITTED and UNDER REVIEW claims.

The missing authorities-contacted values were concentrated in PARKED CAR and VEHICLE THEFT incidents.

### Decision

The missing values were preserved as NULL.

No replacement values were assigned because the project did not have confirmed rules for determining the correct values.

### Reason

Missing information does not always mean incorrect information.

For example, an approved amount might not exist yet for a claim still under review. Replacing it with zero would change the meaning of the data.

The missing values were therefore recorded for data quality monitoring.

## 8. Registering Data Quality Results in Gold

### Challenge

The DataQualityResults table existed in the Silver Lakehouse, but the project also needed the results as a managed table in Gold for the semantic model.

An initial attempt to use a OneLake shortcut exposed the underlying Delta files and transaction metadata rather than providing the expected managed table.

### Solution

A separate notebook named `SureLink_Gold_DataQuality_Registration` was created.

The notebook reads the Silver DataQualityResults table and writes a managed Delta table in the Gold Lakehouse.

### Technical Decision

The Gold table is a snapshot of the Silver results at the time the registration notebook runs.

It does not automatically synchronise with Silver.

The registration notebook must be run again when new Silver quality results need to appear in Gold.

## 9. Power BI Latest Run Calculations

### Challenge

The DataQualityResults table stores results from multiple quality-check executions.

The initial Power BI metrics counted historical results, which caused the dashboard to display totals from more than one execution.

### Solution

The DAX measures were updated to identify the maximum `run_timestamp` and calculate the metrics using records from that timestamp.

This allowed the dashboard cards to focus on the latest recorded execution.

### Technical Learning

When reporting from a table containing execution history, the reporting logic needs to distinguish between historical results and the latest run.

## 10. Power BI Detail Table Filtering

### Challenge

The Data Quality Monitoring page needed to display the individual checks from the latest execution.

A measure-based visual filter did not initially produce the expected detail rows.

A calculated column named `IsLatestRun` was then introduced to identify rows belonging to the latest timestamp.

The table visual still required further verification to ensure it displayed the six individual checks rather than only an aggregated total.

### Decision

The detail table needed to display the individual validation records, with numeric fields set to the appropriate summarisation behaviour.

The visual type and field configuration also needed to be checked.

### Technical Learning

Power BI tables and matrices handle fields and aggregations differently. A correct measure does not automatically guarantee the expected row-level detail in a visual.

## 11. Data Validation Across Layers

### Challenge

The project needed to confirm that data was not lost or duplicated as it moved from Azure SQL through Bronze, Silver and Gold.

Matching total counts alone were not enough to confirm the relationships between the tables.

### Solution

Validation checks were performed at different stages.

These included:

- Comparing Azure SQL and Bronze record counts.
- Comparing Bronze and Silver record counts.
- Checking duplicate primary keys.
- Checking missing key values.
- Comparing the Gold fact table count with the Silver Claim count.
- Checking referential integrity between the fact and dimension tables.

The Gold referential integrity checks found zero unmatched claim records for customer, vehicle, policy and date.

### Technical Learning

Data validation should take place throughout a data pipeline. Checking the final output alone makes it harder to identify where a problem started.

## 12. Key Technical Decisions

| Decision | Reason |
|---|---|
| Use Azure SQL as the source database | Provides a relational database for the insurance source data |
| Use Dataflow Gen2 for ingestion | Connects Azure SQL to the Fabric Bronze Lakehouse |
| Separate Bronze, Silver and Gold layers | Separates ingestion, transformation and reporting |
| Use PySpark for Silver transformations | Supports processing and validation of the source tables |
| Store quality results separately | Keeps validation history separate from business data |
| Preserve uncertain values | Avoids changing data without confirmed business rules |
| Use a fact and dimension model in Gold | Supports structured reporting and analysis |
| Use a separate Gold registration notebook | Creates a managed Gold copy of Silver quality results |
| Filter Power BI metrics by latest run | Prevents historical executions from inflating current metrics |

## 13. Lessons Learned

The development of SureLink provided practical experience in several areas:

1. Connecting Python applications to a cloud-hosted SQL database.
2. Loading records in batches using staging tables and SQL MERGE.
3. Investigating ODBC data insertion errors.
4. Using Dataflow Gen2 to ingest relational data into Microsoft Fabric.
5. Inspecting schemas and transforming data with PySpark.
6. Designing reusable data quality checks.
7. Preserving uncertain values rather than making unsupported corrections.
8. Building fact and dimension tables.
9. Validating relationships between Gold tables.
10. Creating Power BI measures and reporting from execution history.

## 14. Future Improvements

The project provides a foundation for further development.

Possible improvements include:

- Automating the full ingestion and transformation workflow.
- Scheduling data quality checks.
- Refreshing the Gold quality results automatically after each quality-check run.
- Adding more validation rules based on confirmed insurance business requirements.
- Improving the Power BI detail table and latest-run filtering.
- Adding pipeline execution logging and failure notifications.
- Introducing an AI-based feature after the core data pipeline is fully validated.

These are potential improvements and should not be presented as completed features.