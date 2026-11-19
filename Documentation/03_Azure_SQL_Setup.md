# SureLink: Azure SQL Database Setup and Data Loading

## 1. Introduction

Azure SQL Database was used as the source database for the SureLink project. It stores the insurance data before ingestion into Microsoft Fabric.

The database contains four main tables:

- Customer
- Vehicle
- Policy
- Claim

Python was used to load the source data into Azure SQL Database. The `pyodbc` library provided the connection between Python and the database.

## 2. Azure SQL Database Configuration

The database was created in Microsoft Azure.

| Setting | Configuration |
|---|---|
| Database server | `surelink-sql-servers.database.windows.net` |
| Database name | `surelink-source` |
| Database service | Azure SQL Database |
| Database driver | ODBC Driver 18 for SQL Server |
| Port | 1433 |
| Encryption | Enabled |

The database server hosts the `surelink-source` database.

### 2.1 Network Access

Azure SQL Database uses firewall rules to control which IP addresses are allowed to connect.

The public IP address used by the development machine was added to the SQL server firewall rules.

When the public IP address changed, the firewall rule needed to be updated before Python could connect successfully.

### 2.2 Database Connection

The Python loading script used a connection string stored in an environment variable named `SURELINK_SQL_CONN`.

The connection included:

- SQL Server name.
- Database name.
- Login credentials.
- Encryption settings.
- Connection timeout.

The password was entered separately and stored in the environment variable for the current PowerShell session. It was not included in the project documentation or source code.

The ODBC Driver 18 for SQL Server was used to establish the connection.

## 3. Python Database Connection

The `pyodbc` library was used to connect Python to Azure SQL Database.

The connection was tested before running the data loading process.

Example connection test:

```python
import os
import pyodbc

connection = pyodbc.connect(
    os.environ["SURELINK_SQL_CONN"],
    timeout=90
)

print("SQL CONNECTION SUCCESSFUL")

connection.close()
```

The test confirmed whether Python could establish a connection to the Azure SQL Database.

## 4. Data Loading Process

The project used a Python script named `load_to_sql.py` to load the insurance data.

The script processed the four source entities:

1. Customer
2. Vehicle
3. Policy
4. Claim

The loading process used staging tables and SQL MERGE statements.

### 4.1 Staging Tables

A staging table is a temporary holding area used before data enters its final database table.

The script first loaded each batch into the relevant staging table. This separated the incoming records from the existing target data.

### 4.2 SQL MERGE

After loading the staging data, the script used SQL MERGE statements to compare incoming records with existing records.

The process handled two main operations:

- Insert: adds a record when its matching key does not exist in the target table.
- Update: updates an existing record when its matching key already exists.

This approach supported repeated batch loading without simply adding every incoming record as a new row.

### 4.3 Batch Loading

The source data was loaded in three batches:

- Batch 00
- Batch 01
- Batch 02

Batch 00 loaded the initial records.

Batches 01 and 02 introduced additional records and updates to existing records.

The loading process used `fast_executemany` to improve the speed of inserting batches through the ODBC connection.

During development, a string truncation issue occurred when using `fast_executemany`. The script was updated to use `cursor.setinputsizes()` for text columns, with the relevant SQL data type and column lengths. This resolved the issue.

## 5. Loading Results

The following results were recorded after each batch.

### 5.1 Batch 00: Initial Load

| Table | Records loaded |
|---|---:|
| Customer | 16,718 |
| Vehicle | 19,994 |
| Policy | 19,994 |
| Claim | 19,118 |

Batch 00 established the initial data in Azure SQL Database.

### 5.2 Batch 01: Additional Records and Updates

| Table | Records inserted | Records updated |
|---|---:|---:|
| Customer | 222 | 501 |
| Vehicle | 303 | 0 |
| Policy | 303 | 399 |
| Claim | 404 | 1,408 |

The batch inserted new records and updated existing records where matching keys were found.

### 5.3 Batch 02: Additional Records and Updates

| Table | Records inserted | Records updated |
|---|---:|---:|
| Customer | 231 | 508 |
| Vehicle | 300 | 0 |
| Policy | 300 | 405 |
| Claim | 409 | 1,348 |

Batch 02 continued the loading process using the same staging and MERGE approach.

The recorded execution time for Batch 02 was approximately 11.29 seconds.

A rerun of Batch 01 was skipped by the loading process in approximately 2.02 seconds.

## 6. Final Database Record Counts

After all three batches were processed, SQL queries were used to check the final number of records in each table.

| Table | Final records |
|---|---:|
| Customer | 17,171 |
| Vehicle | 20,597 |
| Policy | 20,597 |
| Claim | 19,931 |
| Total | 78,296 |

These counts were used later to verify the data ingestion results in Microsoft Fabric.

## 7. Challenges and Solutions

### 7.1 SQL Server Connection Issues

The connection initially depended on the Azure SQL firewall allowing the development machine's public IP address.

When the IP address changed, the connection failed.

Solution:

The current public IP address was added to the Azure SQL server firewall rules.

### 7.2 String Truncation During Batch Loading

The initial use of `fast_executemany` caused string truncation errors when inserting text values.

Solution:

The script was updated to use `cursor.setinputsizes()` for text columns. The SQL text data type and appropriate column lengths were supplied before executing the batch.

### 7.3 Repeated Batch Processing

The loading process needed to handle existing records and new records without creating unnecessary duplicates.

Solution:

Staging tables and SQL MERGE statements were used to insert new records and update matching records.

## 8. Validation

The database loading process was checked using SQL queries.

The validation focused on:

- Confirming the database connection.
- Checking record counts in the four target tables.
- Confirming the results after each batch.
- Checking the final table counts.
- Comparing the final counts with the expected source data totals.

The final database contained 78,296 records across the four source tables.

These counts provided the baseline for validating the next stage, where Dataflow Gen2 imported the data into the Fabric Bronze Lakehouse.

## 9. Security Considerations

Database credentials were kept out of the project documentation and source code.

The connection string was stored in an environment variable during development.

Before publishing the repository, confirm the following:

- No database passwords appear in Python files.
- No connection strings containing passwords appear in notebooks.
- No credentials are included in screenshots.
- Local environment files containing secrets are excluded from Git.
- Azure firewall access is restricted to the required IP addresses.