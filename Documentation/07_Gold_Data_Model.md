# SureLink: Gold Layer and Dimensional Data Model

## 1. Introduction

The Gold layer is the final data storage layer in the SureLink data engineering process. It contains structured tables prepared for reporting and analysis.

The Gold Lakehouse is named `SureLink_Gold_lh`.

The data model follows a dimensional modelling approach. It separates measurable business events from descriptive information.

For SureLink, the main business event is an insurance claim. The model uses one fact table and four dimension tables to support claim analysis.

The Gold Lakehouse also contains the `DataQualityResults` table for data quality reporting.

## 2. Dimensional Modelling

Dimensional modelling organises data into fact tables and dimension tables.

A fact table stores measurable business events and the keys used to connect related information.

A dimension table stores descriptive information used to analyse those events.

For this project:

- `FactClaim` stores claim-level information.
- `DimCustomer` stores customer details.
- `DimVehicle` stores vehicle details.
- `DimPolicy` stores policy information.
- `DimDate` stores calendar information.

This structure allows claims to be analysed by customer, vehicle, policy and date.

## 3. FactClaim

### 3.1 Purpose

`FactClaim` is the central fact table in the Gold model.

Its grain is one row per insurance claim. Grain describes what one row in a fact table represents.

The table was built by joining the Silver Claim data with related Policy information.

### 3.2 Main Columns

| Column | Description |
|---|---|
| `claim_id` | Unique claim identifier |
| `policy_id` | Related policy identifier |
| `customer_id` | Related customer identifier |
| `vehicle_id` | Related vehicle identifier |
| `incident_date` | Date of the incident |
| `total_claim_amount` | Total amount claimed |
| `approved_amount` | Approved claim amount |
| `claim_status` | Current claim status |
| `incident_type` | Type of incident |
| `incident_severity` | Severity of the incident |
| `incident_province` | Province where the incident occurred |
| `cover_level` | Policy cover level |
| `annual_premium` | Annual policy premium |

### 3.3 Validation

The Silver Claim table contained 19,931 records.

After building the Gold fact table, `FactClaim` also contained 19,931 records.

The matching counts confirmed that the transformation retained the expected number of claims without row multiplication or record loss.

## 4. DimCustomer

### 4.1 Purpose

`DimCustomer` stores descriptive information about insurance customers.

It allows claims to be analysed using customer attributes.

### 4.2 Main Columns

| Column | Description |
|---|---|
| `customer_id` | Customer identifier |
| `customer_ref` | Customer reference |
| `first_name` | Customer first name |
| `last_name` | Customer last name |
| `gender` | Customer gender |
| `birth_date` | Customer birth date |
| `occupation` | Customer occupation |
| `education_level` | Education level |
| `city` | Customer city |
| `province` | Customer province |
| `postal_code` | Customer postal code |

### 4.3 Validation

| Check | Result |
|---|---:|
| Total records | 17,171 |
| Duplicate customer IDs | 0 |
| Missing customer IDs | 0 |

The table retained all 17,171 customer records.

## 5. DimVehicle

### 5.1 Purpose

`DimVehicle` stores descriptive information about insured vehicles.

It allows claims to be analysed using vehicle details.

### 5.2 Main Columns

| Column | Description |
|---|---|
| `vehicle_id` | Vehicle identifier |
| `make` | Vehicle manufacturer |
| `model` | Vehicle model |
| `model_year` | Vehicle model year |

### 5.3 Validation

| Check | Result |
|---|---:|
| Total records | 20,597 |
| Duplicate vehicle IDs | 0 |
| Missing vehicle IDs | 0 |

The table retained all 20,597 vehicle records.

## 6. DimPolicy

### 6.1 Purpose

`DimPolicy` stores descriptive and financial information about insurance policies.

It allows claims to be analysed according to policy details and cover levels.

### 6.2 Main Columns

| Column | Description |
|---|---|
| `policy_id` | Policy identifier |
| `policy_number` | Policy number |
| `customer_id` | Related customer identifier |
| `vehicle_id` | Related vehicle identifier |
| `bind_date` | Policy start or binding date |
| `cover_level` | Insurance cover level |
| `excess_amount` | Policy excess amount |
| `annual_premium` | Annual premium |
| `extended_liability_limit` | Extended liability limit |
| `policy_status` | Policy status |

### 6.3 Validation

| Check | Result |
|---|---:|
| Total records | 20,597 |
| Duplicate policy IDs | 0 |
| Missing policy IDs | 0 |

The 21 negative extended liability limits identified during Silver validation were preserved in Gold.

They were not changed because the correct business rule had not been confirmed.

## 7. DimDate

### 7.1 Purpose

`DimDate` provides calendar information for analysing claims over time.

It supports reporting by date and enables time-based analysis in Power BI.

### 7.2 Date Range

The date dimension covers:

- Start date: 1 January 2025.
- End date: 24 January 2026.

### 7.3 Validation

| Check | Result |
|---|---:|
| Total date records | 389 |
| Duplicate dates | 0 |
| Missing date keys | 0 |

Each date appears once in the dimension table.

## 8. DataQualityResults

The Gold Lakehouse also contains a `DataQualityResults` table.

It stores the results copied from the Silver data quality process.

The table supports the Data Quality Monitoring page in Power BI.

The Gold table is refreshed by running the `SureLink_Gold_DataQuality_Registration` notebook. It is a separate copy of the Silver results and does not update automatically.

## 9. Relationships Between Tables

The Gold semantic model uses four active relationships between `FactClaim` and the dimension tables.

| Fact table column | Dimension table | Dimension column |
|---|---|---|
| `customer_id` | DimCustomer | `customer_id` |
| `vehicle_id` | DimVehicle | `vehicle_id` |
| `policy_id` | DimPolicy | `policy_id` |
| `incident_date` | DimDate | `date_key` |

Each relationship uses:

- Many-to-one cardinality.
- Single-direction filtering.
- An active relationship.

Many-to-one means multiple claims can relate to one customer, vehicle, policy or date.

Single-direction filtering means filters flow from the dimension table towards the fact table.

## 10. Referential Integrity Validation

Referential integrity checks help confirm that fact table records link to matching dimension records.

The following checks were performed:

| Validation | Unmatched records |
|---|---:|
| Claims without matching customers | 0 |
| Claims without matching vehicles | 0 |
| Claims without matching policies | 0 |
| Claims without matching dates | 0 |

The results confirmed that every claim in the fact table had a matching record in the related dimensions.

## 11. Gold Layer Record Counts

The final Gold tables contained the following records:

| Gold table | Records |
|---|---:|
| FactClaim | 19,931 |
| DimCustomer | 17,171 |
| DimVehicle | 20,597 |
| DimPolicy | 20,597 |
| DimDate | 389 |

`DataQualityResults` is also included in the Gold Lakehouse. Its record count depends on the quality-check executions copied from Silver.

## 12. Semantic Model

The Gold tables were connected through a semantic model named `SureLink_Semantic_Model`.

A semantic model provides the relationships and structure used by Power BI to query and analyse the data.

The four active relationships connect `FactClaim` to the customer, vehicle, policy and date dimensions.

The `DataQualityResults` table is included for data quality reporting and is not connected to the claim dimensions.

## 13. Business Reporting Value

The Gold model supports analysis of insurance claims from different perspectives.

Examples include:

- Claim volumes by customer province.
- Claim amounts by incident type.
- Claim severity across different provinces.
- Claims by policy cover level.
- Monthly claim trends.
- Claim statuses and approved amounts.

The model provides a consistent structure for building reports without repeatedly joining the original source tables.

## 14. Outcome

The Gold layer contains one fact table, four dimension tables and the data quality results table.

The fact and dimension tables were validated for record counts, duplicate keys and missing keys.

The referential integrity checks returned zero unmatched claim records across all four dimensions.

The resulting semantic model provides the foundation for the SureLink Power BI report.