# SureLink: Silver Layer Data Transformation

## 1. Introduction

The Silver layer is responsible for cleaning and standardising the data ingested into the Bronze Lakehouse.

For SureLink, the transformations were performed using PySpark in Microsoft Fabric.

The Silver Lakehouse is named `SureLink_Silver_lh`, and the main transformation notebook is `SureLink_Silver_Transformation_nb`.

The notebook reads the source tables from the Bronze Lakehouse, applies the required transformations and saves the processed tables in Silver.

The four tables processed were:

- Customer
- Vehicle
- Policy
- Claim

## 2. Transformation Approach

The transformations focused on improving data consistency while preserving the original business information.

The main activities included:

- Inspecting table schemas.
- Checking missing values.
- Checking duplicate records.
- Identifying empty text values.
- Trimming unnecessary spaces.
- Converting blank text values into NULL.
- Standardising selected text and category fields.
- Investigating unusual values before deciding how to handle them.

The cleaned tables were saved as Delta tables in the Silver Lakehouse.

## 3. Customer Transformation

### 3.1 Source Structure

The Customer table contains information about insurance customers.

The columns include:

- `customer_id`
- `customer_ref`
- `first_name`
- `last_name`
- `gender`
- `birth_date`
- `email`
- `occupation`
- `education_level`
- `address_line`
- `city`
- `province`
- `postal_code`
- `created_at`
- `last_modified`

### 3.2 Data Inspection

The initial checks found:

| Check | Result |
|---|---:|
| Total records | 17,171 |
| Duplicate customer IDs | 0 |
| Missing city values | 148 |
| Invalid email formats | 170 |

No other missing values or empty text values were found in the initial checks.

### 3.3 Transformations

The following transformations were applied:

- Trimmed text fields to remove unnecessary spaces.
- Converted blank text values into NULL.
- Converted email addresses to lowercase.
- Checked email values against an email format rule.
- Identified records with missing city values.

Two additional columns were added:

| Column | Purpose |
|---|---|
| `email_valid` | Indicates whether the email address passes the format check |
| `city_missing` | Indicates whether the city value is missing |

### 3.4 Results

The cleaned Customer table contained 17,171 records.

The quality distribution was:

| Email valid | City missing | Records |
|---|---|---:|
| True | False | 16,853 |
| True | True | 148 |
| False | False | 170 |

The original customer records were retained. Invalid email formats and missing city values were identified rather than removed.

## 4. Vehicle Transformation

### 4.1 Source Structure

The Vehicle table contains information about insured vehicles.

Its columns include:

- `vehicle_id`
- `make`
- `model`
- `model_year`
- `last_modified`

### 4.2 Data Inspection

The initial checks found:

| Check | Result |
|---|---:|
| Total records | 20,597 |
| Duplicate vehicle IDs | 0 |
| Missing make values | 0 |
| Missing model values | 0 |
| Empty text values | 0 |

### 4.3 Transformations

The following transformations were applied:

- Trimmed the `make` and `model` text values.
- Added a `make_missing` flag.
- Added a `model_missing` flag.

These flags indicate whether the corresponding vehicle information is missing.

### 4.4 Results

The cleaned Vehicle table contained 20,597 records.

All records had both `make_missing` and `model_missing` set to false.

No duplicate vehicle IDs were found.

## 5. Policy Transformation

### 5.1 Source Structure

The Policy table contains insurance policy information.

Its columns include:

- `policy_id`
- `policy_number`
- `customer_id`
- `vehicle_id`
- `bind_date`
- `cover_level`
- `excess_amount`
- `annual_premium`
- `extended_liability_limit`
- `policy_status`
- `last_modified`

### 5.2 Data Inspection

The initial checks found:

| Check | Result |
|---|---:|
| Total records | 20,597 |
| Duplicate policy IDs | 0 |
| Missing values | 0 |
| Empty text values | 0 |

### 5.3 Transformations

The following transformations were applied:

- Trimmed `policy_number`.
- Trimmed `cover_level`.
- Trimmed `policy_status`.
- Converted `cover_level` to uppercase.
- Converted `policy_status` to uppercase.

This standardised category values and improved consistency across the table.

### 5.4 Investigation of Negative Liability Limits

During validation, 21 records were found with a negative `extended_liability_limit` of -1,000,000.

All 21 records belonged to policies with the `THIRD PARTY` cover level.

Their policy statuses were:

| Policy status | Records |
|---|---:|
| ACTIVE | 18 |
| LAPSED | 3 |
| Total | 21 |

Other THIRD PARTY policies had liability limits ranging from 0 to 10,000,000.

The negative values were retained because the correct business rule was not available to confirm whether they represented invalid data or a specific business condition.

The issue was recorded for data quality monitoring rather than automatically replacing the values.

### 5.5 Results

The cleaned Policy table contained 20,597 records.

No duplicate policy IDs or missing values were found during the initial checks.

## 6. Claim Transformation

### 6.1 Source Structure

The Claim table contains information about reported insurance claims.

Its columns include:

- `claim_id`
- `claim_number`
- `policy_id`
- `incident_date`
- `reported_date`
- `incident_type`
- `collision_type`
- `incident_severity`
- `authorities_contacted`
- `incident_province`
- `incident_city`
- `incident_location`
- `incident_hour`
- `vehicles_involved`
- `bodily_injuries`
- `witnesses`
- `property_damage`
- `police_report_available`
- `injury_claim`
- `property_claim`
- `vehicle_claim`
- `total_claim_amount`
- `approved_amount`
- `claim_status`
- `fraud_reported`
- `claim_notes`
- `last_modified`

### 6.2 Data Inspection

The initial checks found:

| Check | Result |
|---|---:|
| Total records | 19,931 |
| Duplicate claim IDs | 0 |
| Missing authorities_contacted | 1,792 |
| Missing approved_amount | 5,831 |

No empty text values were found in the initial checks.

### 6.3 Transformations

The following transformations were applied:

- Trimmed text fields.
- Converted blank text values into NULL.
- Standardised selected category fields to uppercase.

The category fields included:

- `incident_type`
- `collision_type`
- `incident_severity`
- `authorities_contacted`
- `claim_status`
- `fraud_reported`

### 6.4 Investigation of Missing Approved Amounts

The missing `approved_amount` values were compared with claim statuses.

| Claim status | Total records | Missing approved amounts |
|---|---:|---:|
| APPROVED | 949 | 0 |
| PAID | 13,151 | 0 |
| REJECTED | 3,821 | 3,821 |
| SUBMITTED | 965 | 965 |
| UNDER REVIEW | 1,045 | 1,045 |

The missing values were associated with claims that were rejected, submitted or under review.

The values were preserved as NULL. No replacement amounts were created because the project did not have a confirmed business rule for calculating or assigning approved amounts to these statuses.

### 6.5 Investigation of Missing Authorities Contacted

A total of 1,792 records had no value in `authorities_contacted`.

The missing values were investigated by incident type.

| Incident type | Total records | Missing values | Missing percentage |
|---|---:|---:|---:|
| PARKED CAR | 1,670 | 997 | 59.70% |
| VEHICLE THEFT | 1,865 | 795 | 42.63% |
| MULTI-VEHICLE COLLISION | 8,307 | 0 | 0% |
| SINGLE VEHICLE COLLISION | 8,089 | 0 | 0% |

The populated values included:

| Authorities contacted | Records |
|---|---:|
| POLICE | 5,841 |
| FIRE | 4,439 |
| AMBULANCE | 3,932 |
| OTHER | 3,927 |
| NULL | 1,792 |

The missing values were preserved because the available information did not establish which authority, if any, should be assigned to each record.

The issue was retained for data quality monitoring.

### 6.6 Results

The cleaned Claim table contained 19,931 records.

No duplicate claim IDs were found.

The missing values in `approved_amount` and `authorities_contacted` were preserved for further investigation.

## 7. Silver Layer Output

The four transformed tables were saved in the `SureLink_Silver_lh` Lakehouse.

| Silver table | Records |
|---|---:|
| Customer | 17,171 |
| Vehicle | 20,597 |
| Policy | 20,597 |
| Claim | 19,931 |
| Total | 78,296 |

The record counts matched the Bronze tables.

This confirmed that the transformations retained the expected number of records.

## 8. Data Handling Decisions

During the transformation stage, some values required further investigation.

The project followed these decisions:

| Data issue | Handling decision |
|---|---|
| Invalid customer email formats | Retained and flagged |
| Missing customer city | Retained and flagged |
| Missing vehicle make or model | Checked and flagged |
| Negative policy liability limits | Preserved for business rule confirmation |
| Missing approved amounts | Preserved as NULL |
| Missing authorities contacted | Preserved as NULL |

These decisions avoided changing source information without sufficient evidence.

## 9. Outcome

The Silver layer contains cleaned and standardised versions of the four insurance tables.

The transformations improved text consistency and identified data quality issues for further investigation.

The resulting tables provide the input for the Gold dimensional model and the data quality monitoring process.