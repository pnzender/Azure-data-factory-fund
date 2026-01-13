# Azure Data Factory Practice – Event-Driven CSV Pipeline

This repository contains my Azure Data Factory (ADF) practice project. It demonstrates an event-driven pipeline using Azure Blob Storage triggers, pipeline orchestration, metadata-driven file processing, and a mapping data flow transformation.

When the file `Fact_Sales_1.csv` is uploaded to `source/csvfiles/Fact_Sales_1.csv`, a Blob Events Trigger starts the main pipeline `prodPipeline`. This pipeline orchestrates two pipelines in sequence: `pipelinemanager` and `OnlySelectedFiles`.

The `pipelinemanager` pipeline performs three actions. First, it copies the uploaded CSV file from `source/csvfiles` to `destination/csvfiles`. Second, it deletes the file from the source container to keep the landing zone clean. Third, it executes another pipeline called `pipelineGit`, which ingests a CSV file from a public GitHub repository using an HTTP linked service and stores it in `destination/csvfiles`.

After that, the `OnlySelectedFiles` pipeline is executed. This pipeline uses a Get Metadata activity to list all files in `destination/csvfiles`. A ForEach activity iterates over each file and applies a condition that checks whether the file name starts with `Fact`. If the condition is true, the file is copied dynamically from `destination/csvfiles` to `reporting/csv` using the file name obtained from `@item().name`.

Once the selected files are copied, a Mapping Data Flow called `TransformCSV` is executed. The data flow reads data from `reporting/csv`, selects a subset of columns (`transaction_id`, `transactional_date`, `product_id`, `customer_id`, `payment`, `price`), filters out records where `customer_id = 12`, and splits the data into payment groups (Visa, Mastercard, and Amex). The data is then aggregated by `customer_id`. An `alterRows` transformation is used with an insert condition (`1 == 1`) to define insert behavior. Finally, only the Visa payment group is written to `reporting/dataflowOutput`.

The storage layout follows a simple lake-style pattern. The `source` container is used as a landing zone, the `destination` container is used as a staging area, and the `reporting` container stores processed outputs, including both intermediate CSV files and final transformed data.

All Azure resource identifiers, URLs, and credentials have been redacted or generalized. Linked services are intended to be configured using Managed Identity or secure credential management. This project is for learning and demonstration purposes only.
