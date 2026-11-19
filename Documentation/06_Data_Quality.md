# SureLink: Data Quality Checks and Monitoring

## 1. Introduction

Data quality is an important part of the SureLink data engineering process. Before data is used for reporting, it needs to be checked for issues such as missing values, invalid formats and unexpected values.

For this project, data quality checks were developed in Microsoft Fabric using PySpark.

The checks were performed on the Silver tables:

- Customer
- Vehicle
- Policy
- Claim

The results were stored in a separate Delta table named `DataQualityResults`. This table records the outcome of each validation check and provides information for the Data Quality Monitoring report.

## 2. Data Quality Approach

The project used rule-based validation.

A data quality rule is a condition used to check whether records meet a specific requirement.

For example, an email validation rule checks whether customer email addresses follow an expected format.

Each check records:

- The table being checked.
- The column being checked.
- The validation rule.
- The number of failed records.
- The severity assigned to the issue.
- The check status.
- The total number of records checked.
- The execution timestamp.
- A unique run ID.

The checks were designed to identify issues without automatically changing or deleting source records.

## 3. Reusable Data Quality Functions

Two reusable functions were developed in the Fabric notebook:

### 3.1 `run_quality_check`

This function executes a validation rule against a selected table and records the result.

It captures information such as the table name, column name, rule name, failed record count, severity and check status.

A check receives one of two statuses:

| Status | Meaning |
|---|---|
| PASS | No records failed the validation rule |
| FAIL | One or more records failed the validation rule |

### 3.2 `save_quality_results`

This function saves the validation results into the `DataQualityResults` Delta table.

It allows results from different checks and executions to be stored in a consistent structure.

The table keeps the history of previous runs instead of replacing every result with the latest execution.

## 4. Data Quality Results Table

The `DataQualityResults` table contains the following columns:

| Column | Description |
|---|---|
| `run_id` | Unique identifier for a quality-check execution |
| `table_name` | Name of the table checked |
| `column_name` | Name of the column checked |
| `rule_name` | Name of the validation rule |
| `failed_records` | Number of records failing the rule |
| `severity` | Assigned severity level |
| `status` | PASS or FAIL |
| `description` | Explanation of the validation rule |
| `run_timestamp` | Date and time of the execution |
| `total_records` | Number of records checked |

The table was created in the Silver Lakehouse and registered as `DataQualityResults` under the `dbo` schema.

## 5. Validation Rules

Six validation checks were included in the latest recorded execution.

| Table | Column | Validation rule | Failed records | Status |
|---|---|---|---:|---|
| Claim | AUTHORITIES_CONTACTED | AUTHORITIES_CONTACTED_REQUIRED | 1,792 | FAIL |
| Vehicle | VEHICLE_MAKE | VEHICLE_MAKE_REQUIRED | 0 | PASS |
| Customer | EMAIL | VALID_EMAIL_FORMAT | 170 | FAIL |
| Customer | CITY | CITY_REQUIRED | 148 | FAIL |
| Vehicle | VEHICLE_MODEL | VEHICLE_MODEL_REQUIRED | 0 | PASS |
| Policy | EXTENDED_LIABILITY_LIMIT | NON_NEGATIVE_LIABILITY_LIMIT | 21 | FAIL |

The latest recorded run contained:

- 6 quality checks.
- 4 failed checks.
- 2 passed checks.
- 2,131 total rule failures.

The figure of 2,131 represents the sum of failed records across the six checks. It does not represent 2,131 unique records, because one record could fail more than one rule.

## 6. Severity Levels

Severity levels were assigned to help distinguish issues according to their potential importance.

The project used three levels:

| Severity | Purpose |
|---|---|
| HIGH | Identifies issues assigned a high level of attention |
| MEDIUM | Identifies issues requiring investigation |
| LOW | Identifies issues assigned a lower level of attention |

The latest recorded run produced the following failed-check distribution:

| Severity | Failed checks | Failed records |
|---|---:|---:|
| HIGH | 1 | 21 |
| MEDIUM | 2 | 1,962 |
| LOW | 1 | 148 |
| Total | 4 | 2,131 |

These severity levels reflect the rules defined for this portfolio project. They are not presented as an official insurance industry standard.

## 7. Latest Quality Check Execution

The `DataQualityResults` table stores results from multiple executions.

The latest recorded execution used this run ID:

`1927ed62-1ea7-4b23-b504-a0f330d74a91`

Execution timestamp:

`2026-11-15 15:19:51.800043`

The run contained six validation checks.

The latest run is important for reporting because older results should not be counted as if they belong to the current execution.

The Power BI measures were therefore designed to calculate quality metrics using the latest recorded timestamp.

### 7.1 Latest Run Metrics

| Metric | Result |
|---|---:|
| Total quality checks | 6 |
| Failed quality checks | 4 |
| Passed quality checks | 2 |
| Quality failure rate | 66.67% |
| High-severity failed checks | 1 |
| Medium-severity failed checks | 2 |
| Low-severity failed checks | 1 |

The quality failure rate is calculated as:

`Failed Quality Checks / Total Quality Checks × 100`

For the latest run:

`4 / 6 × 100 = 66.67%`

## 8. Preserving Data During Quality Checks

The quality-check process identifies records requiring attention. It does not automatically correct every failed record.

Several issues required further business information before a correction could be made.

Examples include:

- Missing `authorities_contacted` values in Claim.
- Invalid email formats in Customer.
- Missing city values in Customer.
- Negative extended liability limits in Policy.

The checks record these issues so they remain visible for investigation.

This approach avoids introducing assumptions into the insurance data.

## 9. Moving Quality Results into Gold

The `DataQualityResults` table was created in Silver.

To make the results available to the Gold semantic model, a separate Fabric notebook named `SureLink_Gold_DataQuality_Registration` was used.

The notebook reads the Silver quality results and writes them as a managed Delta table in the Gold Lakehouse.

The Gold copy is a snapshot of the Silver results at the time the registration notebook runs.

It does not automatically update when new results are added to Silver. The registration notebook must be run again to refresh the Gold copy.

The Gold table is used by the semantic model and Power BI report.

## 10. Power BI Data Quality Monitoring

The Power BI report includes a page named `Data Quality Monitoring`.

The page uses the Gold `DataQualityResults` table to display the quality-check results.

The page includes:

1. Total Quality Checks.
2. Quality Failure Rate.
3. Failed Quality Checks by Severity.
4. Quality Check Details.

The detail table includes:

- Table name.
- Rule name.
- Failed records.
- Total records.
- Severity.
- Status.
- Run timestamp.

The report focuses on the latest execution so users can review the current recorded validation results without mixing them with older runs.

## 11. Challenges and Technical Decisions

### 11.1 Keeping Quality Results Separate

Instead of adding every validation result directly to the business tables, a separate `DataQualityResults` table was created.

This keeps the business data structure separate from the validation history.

### 11.2 Registering Results in Gold

The Silver quality table needed to be available as a managed table in Gold for the semantic model.

A separate registration notebook was used to read the Silver table and write a Gold copy.

### 11.3 Reporting Only the Latest Run

The quality table contains historical results from multiple executions.

Power BI measures were updated to use the latest `run_timestamp` when calculating the quality metrics.

This prevents historical executions from inflating the current check totals.

## 12. Outcome

The project includes a reusable data quality process that checks selected fields across the Silver tables.

The results are stored with execution details and severity levels, copied into Gold and presented through the Power BI Data Quality Monitoring page.

The process provides visibility into data issues and supports further investigation before the data is used for business reporting.