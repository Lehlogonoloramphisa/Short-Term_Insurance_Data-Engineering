# SureLink: Power BI Report and Dashboard

## 1. Introduction

Power BI was used to present the insurance data stored in the Gold Lakehouse.

The report is named `SureLink_Insurance_Claims_Dashboard` and uses the `SureLink_Semantic_Model` semantic model.

The report provides an overview of insurance claims, supports further analysis of claim patterns and displays data quality results.

It contains three pages:

1. Claims Overview
2. Claims Analysis
3. Data Quality Monitoring

## 2. Semantic Model Connection

The Power BI report uses the Gold tables from the SureLink semantic model.

The main reporting tables are:

- `FactClaim`
- `DimCustomer`
- `DimVehicle`
- `DimPolicy`
- `DimDate`
- `DataQualityResults`

The semantic model contains four active relationships connecting `FactClaim` to the customer, vehicle, policy and date dimensions.

The `DataQualityResults` table supports the data quality reporting page and is not connected to the claim dimensions.

## 3. Page One: Claims Overview

### 3.1 Purpose

The Claims Overview page provides a high-level view of insurance claim activity.

It displays key claim metrics and visual summaries to help users understand claim volumes, financial amounts, claim statuses and incident patterns.

### 3.2 KPI Cards

The page includes five KPI cards.

| Card | Purpose |
|---|---|
| Total Claims | Displays the total number of claims |
| Total Claim Amount | Displays the total amount claimed |
| Total Approved Amount | Displays the total approved claim amount |
| Average Claim Amount | Displays the average claim amount |
| Claims Under Review | Displays claims with an UNDER REVIEW status |

The expected total claim count is 19,931. The number of claims under review is 1,045.

### 3.3 Claims by Status

Visual: Clustered column chart.

This chart displays the number of claims in each status category.

It helps users compare claim volumes across statuses such as:

- PAID
- APPROVED
- REJECTED
- SUBMITTED
- UNDER REVIEW

### 3.4 Claims by Province

Visual: Clustered bar chart.

This chart displays claim volumes by incident province.

The chart is sorted in descending order to make it easier to identify provinces with higher claim volumes.

### 3.5 Claims by Incident Type

Visual: Clustered column chart.

This chart compares claim volumes across incident types.

It helps users understand which types of incidents appear most frequently in the claim data.

### 3.6 Monthly Claims Trend

Visual: Line chart.

This chart displays claim volumes over time using `DimDate[year_month]`.

It provides a monthly view of claim activity and helps users identify changes in claim volumes across the available reporting period.

## 4. Page Two: Claims Analysis

### 4.1 Purpose

The Claims Analysis page provides more detailed views of claim patterns.

It focuses on incident severity, financial amounts, monthly claim activity and the relationship between province and incident severity.

### 4.2 Claims by Incident Severity

Visual: Clustered column chart.

This chart compares the number of claims across incident severity categories.

It provides a view of how claims are distributed according to their recorded severity.

### 4.3 Total Claim Amount by Incident Type

Visual: Clustered bar chart.

This chart compares total claim amounts across different incident types.

It helps users identify which incident categories account for higher claim amounts.

### 4.4 Monthly Claims by Status

Visual: Stacked column chart.

This chart displays monthly claim volumes, separated by claim status.

The chart uses:

- `DimDate[year_month]` for the horizontal axis.
- `FactClaim[claim_status]` for the legend.
- Total Claims for the values.

It provides a monthly comparison of claim activity across different statuses.

### 4.5 Claims by Province and Severity

Visual: Matrix.

The matrix displays claim volumes by incident province and incident severity.

Provinces appear as rows, while severity categories appear as columns.

This allows users to compare the distribution of claim severity across provinces.

## 5. Page Three: Data Quality Monitoring

### 5.1 Purpose

The Data Quality Monitoring page presents the results of the validation checks performed in the Silver layer.

It uses the `DataQualityResults` table copied into the Gold Lakehouse.

The page focuses on the latest recorded quality-check execution.

### 5.2 Total Quality Checks

Visual: Card.

This card displays the number of validation checks in the latest execution.

The latest recorded run contains six checks.

### 5.3 Quality Failure Rate

Visual: Card.

This card displays the percentage of checks which failed in the latest execution.

The calculation is:

`Failed Quality Checks / Total Quality Checks × 100`

For the latest recorded run:

`4 / 6 × 100 = 66.67%`

### 5.4 Failed Quality Checks by Severity

Visual: Clustered column chart.

This chart displays the number of failed checks grouped by severity.

The latest recorded run contains:

| Severity | Failed checks |
|---|---:|
| HIGH | 1 |
| MEDIUM | 2 |
| LOW | 1 |

### 5.5 Quality Check Details

Visual: Table.

The detail table provides information about individual validation checks.

It includes:

- Table name.
- Rule name.
- Failed records.
- Total records.
- Severity.
- Status.
- Run timestamp.

This allows users to identify which tables and fields require further investigation.

## 6. Power BI Measures

The report uses measures to calculate key metrics.

A measure is a calculation performed by Power BI based on the data and filters applied to a report.

### 6.1 Total Claims

Counts the claims in the fact table.

```dax
Total Claims =
COUNTROWS(FactClaim)
```

### 6.2 Claims Under Review

Counts claims with an UNDER REVIEW status.

```dax
Claims Under Review =
CALCULATE(
    [Total Claims],
    FactClaim[claim_status] = "UNDER REVIEW"
)
```

### 6.3 Latest Run

Identifies the latest quality-check timestamp.

```dax
Latest Run =
VAR LatestRun =
    CALCULATE(
        MAX(DataQualityResults[run_timestamp]),
        ALL(DataQualityResults)
    )
RETURN
    LatestRun
```

### 6.4 Total Quality Checks

Counts the validation results from the latest execution.

```dax
Total Quality Checks =
VAR LatestRun =
    CALCULATE(
        MAX(DataQualityResults[run_timestamp]),
        ALL(DataQualityResults)
    )
RETURN
    CALCULATE(
        COUNTROWS(DataQualityResults),
        FILTER(
            ALL(DataQualityResults),
            DataQualityResults[run_timestamp] = LatestRun
        )
    )
```

### 6.5 Failed Quality Checks

Counts failed checks from the latest execution.

```dax
Failed Quality Checks =
VAR LatestRun =
    CALCULATE(
        MAX(DataQualityResults[run_timestamp]),
        ALL(DataQualityResults)
    )
RETURN
    CALCULATE(
        COUNTROWS(DataQualityResults),
        FILTER(
            ALL(DataQualityResults),
            DataQualityResults[run_timestamp] = LatestRun
                && DataQualityResults[status] = "FAIL"
        )
    )
```

### 6.6 Passed Quality Checks

Counts passed checks from the latest execution.

```dax
Passed Quality Checks =
VAR LatestRun =
    CALCULATE(
        MAX(DataQualityResults[run_timestamp]),
        ALL(DataQualityResults)
    )
RETURN
    CALCULATE(
        COUNTROWS(DataQualityResults),
        FILTER(
            ALL(DataQualityResults),
            DataQualityResults[run_timestamp] = LatestRun
                && DataQualityResults[status] = "PASS"
        )
    )
```

### 6.7 Quality Failure Rate

Calculates the percentage of failed checks.

```dax
Quality Failure Rate =
DIVIDE(
    [Failed Quality Checks],
    [Total Quality Checks],
    0
)
```

The measure is formatted as a percentage in Power BI.

### 6.8 High Severity Failures

```dax
High Severity Failures =
VAR LatestRun =
    CALCULATE(
        MAX(DataQualityResults[run_timestamp]),
        ALL(DataQualityResults)
    )
RETURN
    CALCULATE(
        COUNTROWS(DataQualityResults),
        FILTER(
            ALL(DataQualityResults),
            DataQualityResults[run_timestamp] = LatestRun
                && DataQualityResults[status] = "FAIL"
                && DataQualityResults[severity] = "HIGH"
        )
    )
```

The Medium and Low severity measures follow the same calculation, using their respective severity values.

## 7. Report Design and Navigation

The report separates the main insurance overview from detailed analysis and data quality monitoring.

This structure allows users to move from general claim information to more detailed comparisons and validation results.

The report uses KPI cards, charts, a matrix and a detail table to present the information.

## 8. Business Reporting Value

The report supports several types of insurance analysis.

Users can examine:

- Overall claim volumes.
- Total and approved claim amounts.
- Claims by status.
- Claims by province.
- Claims by incident type.
- Claim severity patterns.
- Monthly claim trends.
- Data quality failures.

These views provide information for reporting and help identify areas requiring further investigation.

## 9. Outcome

The SureLink Power BI report contains three pages built using the Gold semantic model.

The Claims Overview page presents key metrics and claim summaries.

The Claims Analysis page provides more detailed comparisons of claim patterns.

The Data Quality Monitoring page displays the latest recorded validation results.

Together, these pages provide a reporting layer for the SureLink insurance data engineering project.