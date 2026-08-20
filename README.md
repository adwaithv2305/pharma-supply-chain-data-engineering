# Pharmaceutical Supply Chain Data Engineering Pipeline
## Project Overview

An end-to-end pharmaceutical supply chain data engineering pipeline built using Databricks, PySpark, Python, SQL, and Delta Lake.

The project simulates the processing of pharmaceutical supply chain data from raw source files through Bronze, Silver, and Gold layers. It includes data ingestion, data transformation, incremental processing, automated data quality validation, workflow orchestration, and scheduled execution.

## Project Objective

The objective of this project is to build a reliable data engineering pipeline that transforms raw pharmaceutical supply chain data into clean, validated, and analytics-ready datasets.

The project covers multiple business domains:

- Purchase Orders
- Sales
- Shipments
- Quality Events
- Inventory
- Batches
- Suppliers
- Drug Master

## Technology Stack

| Technology | Purpose |
|---|---|
| Databricks | Data processing, notebooks, and workflow orchestration |
| PySpark | Data ingestion and transformation |
| Python | Pipeline logic and Data Quality validation |
| SQL | Data querying and validation |
| Delta Lake | Reliable table storage and incremental MERGE processing |
| Databricks Volumes | Raw file storage |
| Databricks Jobs | Pipeline orchestration and scheduling |

## Architecture

The pipeline follows a Medallion Architecture consisting of Bronze, Silver, and Gold layers.

```text
                     RAW DATA
                         |
                         v
              +-------------------+
              |   BRONZE LAYER    |
              | Raw/Ingested Data |
              +---------+---------+
                        |
                        v
              +-------------------+
              |   SILVER LAYER    |
              | Clean + Transform |
              | Incremental MERGE |
              +---------+---------+
                        |
                        v
              +-------------------+
              |   DATA QUALITY    |
              | Validation Checks  |
              +---------+---------+
                        |
                       PASS
                        |
                        v
              +-------------------+
              |    GOLD LAYER     |
              | Business-Ready    |
              | Analytics Data    |
              +---------+---------+
                        |
                        v
              +-------------------+
              | Databricks Jobs   |
              | Orchestration     |
              +-------------------+
```

## Dataset / Business Domains

| Dataset | Business Key | Purpose |
|---|---|---|
| Purchase Orders | `po_id` | Tracks purchase orders |
| Sales | `sale_id` | Tracks sales transactions |
| Shipments | `shipment_id` | Tracks shipments |
| Quality Events | `quality_event_id` | Tracks quality-related events |
| Inventory | — | Inventory information |
| Batches | — | Pharmaceutical batch information |
| Suppliers | — | Supplier information |
| Drug Master | — | Drug reference information |

## Bronze Layer

The Bronze layer contains raw data ingested from CSV files stored in Databricks Volumes.

The purpose of the Bronze layer is to provide a staging layer containing the source data before business transformations are applied.

Current flow:

CSV → Databricks Volume → Bronze Delta Table

The Bronze layer currently uses snapshot-style ingestion from the source files.

## Silver Layer

The Silver layer contains cleaned and transformed data.

Transformations include:

- Data standardization
- Business transformations
- Derived columns
- Removing unnecessary intermediate columns
- Adding ingestion timestamps
- Incremental processing using Delta MERGE

### Purchase Orders

- Standardized `po_status`
- Added `ingestion_timestamp`
- Used `po_id` as the MERGE key

### Sales

- Calculated `revenue`
- Added `ingestion_timestamp`
- Used `sale_id` as the MERGE key

### Shipments

- Transformed shipment quantities
- Removed intermediate columns
- Added `ingestion_timestamp`
- Used `shipment_id` as the MERGE key

### Quality Events

- Added `ingestion_timestamp`
- Used `quality_event_id` as the MERGE key

## Incremental Processing

Incremental processing was implemented using Delta Lake `MERGE` operations.

The pipeline compares incoming records against existing Silver records using business keys.

```text
Incoming Record
      |
      v
Compare Business Key
      |
   +--+--+
   |     |
Exists  New
   |     |
UPDATE  INSERT
```
## Data Quality

A centralized Data Quality notebook was created to validate important Silver datasets before Gold processing.

The Data Quality task acts as a quality gate in the pipeline.

### Purchase Orders

Checks:

- NULL `po_id`
- Duplicate `po_id`
- Negative `quantity_ordered`
- Invalid `po_status`

### Sales

Checks:

- NULL `sale_id`
- Duplicate `sale_id`
- Negative `quantity_sold`

### Shipments

Checks:

- NULL `shipment_id`
- Duplicate `shipment_id`
- Negative `quantity_shipped`

### Quality Events

Checks:

- NULL `quality_event_id`
- Duplicate `quality_event_id`
- Invalid `event_status`

The Data Quality notebook produces an overall PASS/FAIL result.

If any validation fails, the notebook raises an exception and dependent Gold tasks do not proceed.

## Gold Layer

The Gold layer contains business-ready datasets intended for downstream analytics and reporting.

Gold datasets include:

- Gold Inventory
- Gold Supplier
- Gold Sales
- Gold Quality

Gold processing depends on successful upstream processing and Data Quality validation.

## Orchestration

The complete pipeline is orchestrated using Databricks Jobs.

The workflow follows:

```text
Bronze / Silver Tasks
        |
        v
Data Quality Checks
        |
      PASS
        |
        v
Gold Tasks
```
## Scheduling

The Databricks Job is configured for daily scheduled execution.

**Schedule:** Daily  
**Time:** 6:00 AM IST

Databricks Job monitoring provides:

- Run history
- Task-level status
- Execution duration
- Logs
- DAG visualization
- Failure details

## Pipeline Validation

The pipeline was validated using:

- Record count validation
- NULL key checks
- Duplicate key checks
- Business-rule validation
- Incremental record validation
- Data Quality PASS/FAIL validation
- End-to-end Job execution

The complete workflow was successfully executed:

```text
Bronze
   ↓
Silver
   ↓
Data Quality
   ↓
Gold
```

## Business Impact

The pipeline improves the reliability and maintainability of pharmaceutical supply chain data processing by:

- Automating data transformations
- Reducing manual data processing
- Supporting incremental data ingestion
- Preventing invalid data from reaching Gold datasets
- Providing centralized workflow orchestration
- Enabling scheduled execution
- Providing task-level monitoring and failure visibility

## Future Improvements

Potential production-level improvements include:

- AWS S3-based source ingestion
- Databricks Auto Loader
- Change Data Capture (CDC)
- More comprehensive Data Quality framework
- Automated email or Slack alerts
- Audit and logging tables
- Schema evolution handling
- Parameterized and reusable notebooks
- Advanced pipeline monitoring

## Key Engineering Concepts

- Medallion Architecture
- ETL / ELT
- PySpark transformations
- Delta Lake
- Incremental data processing
- MERGE-based upserts
- Data Quality validation
- Workflow orchestration
- Task dependencies
- Scheduled batch processing

## Repository Structure

```text
pharma-supply-chain-data-engineering/
│
├── README.md
│
├── notebooks/
│   ├── 01_PO_Bronze_Silver
│   ├── 02_Shipments_Bronze_Silver
│   ├── 03_Sales_Bronze_Silver
│   ├── 04_Quality_Events_Bronze_Silver
│   ├── 05_Inventory_Bronze_Silver
│   ├── 06_Batches_Bronze_Silver
│   ├── 07_Suppliers_Bronze_Silver
│   ├── 08_Drug_Master_Bronze_Silver
│   ├── 09_Gold_Inventory
│   ├── 10_Gold_Supplier
│   ├── 11_Gold_Sales
│   ├── 12_Gold_Quality
│   └── 15_Data_Quality_Checks
│
└── images/
    └── architecture.png
```
## Author

**Adwaith Varma**

Data Engineer | Databricks | PySpark | SQL | AWS
