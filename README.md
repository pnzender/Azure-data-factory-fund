# Azure Data Factory Examples – Event-Driven, Metadata-Driven CSV Pipeline

## Overview

This repository contains a hands-on **Azure Data Factory (ADF)** practice project designed to demonstrate real-world data engineering patterns. The project showcases how to build an **event-driven ingestion pipeline**, orchestrate multiple pipelines, dynamically process files using metadata, and apply transformations using **Mapping Data Flows**.

The goal of this project is to simulate a production-style CSV ingestion workflow using Azure-native services while following clean data lake design principles (landing, staging, and reporting zones).


## High-Level Architecture

The solution is triggered by a **Blob Events Trigger** when a specific file is uploaded to Azure Data Lake Storage Gen2. Once triggered, a master pipeline orchestrates downstream pipelines responsible for ingestion, cleanup, conditional file processing, and transformation.

**Key concepts demonstrated:**
- Event-driven pipelines (Blob Storage trigger)
- Pipeline orchestration (pipeline calling pipeline)
- Metadata-driven file handling
- Parameterized datasets
- Conditional logic with ForEach + If Condition
- Mapping Data Flow transformations


## Trigger

The pipeline execution starts when the following event occurs:

- **Trigger type:** BlobEventsTrigger  
- **Condition:** A file named `Fact_Sales_1.csv` is uploaded to  
  `source/csvfiles/Fact_Sales_1.csv`

This design ensures the pipeline runs automatically based on data arrival rather than a schedule.


## Storage Layout

The storage account follows a lake-style layout with clear separation of concerns:

- **source**  
  Landing zone for incoming files  
  `source/csvfiles/`

- **destination**  
  Staging area for validated and prepared files  
  `destination/csvfiles/`

- **reporting**  
  Processed and curated outputs  
  - `reporting/csv/`  
  - `reporting/dataflowOutput/`


## Pipelines

### prodPipeline (Main Orchestrator)

This is the entry-point pipeline triggered by the storage event. It orchestrates the execution of two pipelines in sequence:

1. `pipelinemanager`
2. `OnlySelectedFiles`


### pipelinemanager

This pipeline is responsible for ingestion, cleanup, and external data enrichment. It performs the following steps:

1. **Copy CSV**  
   Copies `Fact_Sales_1.csv` from `source/csvfiles` to `destination/csvfiles`.

2. **Delete File**  
   Deletes the original file from the source container to keep the landing zone clean and ready for future uploads.

3. **Execute pipelineGit**  
   Executes another pipeline that ingests a CSV file from a public GitHub repository using an HTTP linked service and stores it in `destination/csvfiles`.  
   This demonstrates how ADF can integrate with external HTTP-based data sources.


### OnlySelectedFiles

This pipeline demonstrates metadata-driven and conditional file processing.

1. **Get Metadata**  
   Retrieves the list of files (`childItems`) from `destination/csvfiles`.

2. **ForEach + If Condition**  
   Iterates over each file and checks the condition:  
   `@startsWith(item().name, 'Fact')`

3. **Conditional Copy**  
   If the condition is true, the file is copied dynamically from  
   `destination/csvfiles` → `reporting/csv`  
   using `@item().name` for the file name.

4. **Execute Mapping Data Flow**  
   After the copy step, a Mapping Data Flow is executed to transform the data.


## Mapping Data Flow – TransformCSV

The Mapping Data Flow applies business-style transformations to the selected CSV files.

**Steps performed:**

1. **Source**  
   Reads data from `reporting/csv`.

2. **Select Columns**  
   Keeps only the following columns:
   - `transaction_id`
   - `transactional_date`
   - `product_id`
   - `customer_id`
   - `payment`
   - `price`

3. **Filter Rows**  
   Excludes records where `customer_id = 12`.

4. **Conditional Split**  
   Splits the data into separate streams based on payment type:
   - Visa
   - Mastercard
   - Amex

5. **Aggregation**  
   Aggregates data by `customer_id` to produce grouped results.

6. **Alter Rows**  
   Uses an `alterRows` transformation with an insert condition (`1 == 1`) to define row-level behavior for the sink.

7. **Sink**  
   Writes only the **Visa** payment group to  
   `reporting/dataflowOutput`.


## Datasets and Parameterization

This project makes extensive use of parameterized datasets to support dynamic file processing:

- `csv_source` – Source CSV in `source/csvfiles`
- `source_git` – HTTP dataset pointing to a GitHub raw CSV file
- `metadatads` – Used for listing files in `destination/csvfiles`
- `param_source_csv` – Parameterized dataset using `@dataset().p_file_name`
- `reporting_sink` – Writes files dynamically to `reporting/csv`
- `dataflow_source` – Input for the data flow
- `dataflow_sink` – Final output of the data flow


## Security Notes

- No secrets, access keys, or credentials are stored in this repository.
- Linked services use encrypted credentials or Managed Identity.
- Azure resource IDs, subscription IDs, and storage account names can be safely redacted when sharing.
- This repository is intended strictly for learning and demonstration purposes.


## What This Project Demonstrates

- End-to-end ADF pipeline design
- Event-based ingestion patterns
- Clean separation of data lake zones
- Dynamic and scalable file processing
- Practical use of Mapping Data Flows
- GitHub-based version control for ADF assets


## Author Notes

This project was built as part of hands-on Azure Data Factory practice to reinforce data engineering concepts and showcase practical pipeline design patterns suitable for real-world scenarios.
