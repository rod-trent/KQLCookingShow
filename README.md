# KQL Cooking Show Datasets - README

This README provides instructions for using the datasets created for the [*KQL Cooking Show: Recipes for Tasty Data Insights*](https://rodtrent.substack.com/p/kql-cooking-show-recipes-for-tasty) blog post. These datasets are designed to work with the Kusto Query Language (KQL) queries presented in the blog post, enabling you to replicate the *Data Smoothie* and *Hearty Analytics Stew* recipes in Azure Data Explorer.

## Overview

The datasets are provided as CSV files. They include:

- **EventsTable.csv**: User activity logs for the *Data Smoothie* recipe.
- **UsersTable.csv**: User demographics for the *Data Smoothie* recipe.
- **LogsTable.csv**: System logs for the *Hearty Analytics Stew* recipe.
- **MachinesTable.csv**: Machine metadata for the *Hearty Analytics Stew* recipe.

These datasets are small for simplicity but contain enough data to demonstrate the KQL queries effectively.

## Prerequisites

- **Azure Data Explorer**: Access to an Azure Data Explorer cluster and database where you can ingest data and run KQL queries.
- **KQL Queries**: The KQL queries from the [*KQL Cooking Show* blog post](https://rodtrent.substack.com/p/kql-cooking-show-recipes-for-tasty), available in the blog post content.
- **Tools**: Basic knowledge of Azure Data Explorer's ingestion methods (e.g., via the UI or KQL commands).

## Setup Instructions

### 1. Download the Datasets
- Download the datasets:
  - `EventsTable.csv`
  - `UsersTable.csv`
  - `LogsTable.csv`
  - `MachinesTable.csv`

### 2. Create Tables in Azure Data Explorer
Create tables in your Azure Data Explorer database to hold the data. Use the following KQL commands to define the table schemas:

```kql
.create table EventsTable (
    Timestamp: datetime,
    UserId: string,
    EventType: string,
    SessionDuration: int
)

.create table UsersTable (
    UserId: string,
    AgeGroup: string,
    Region: string
)

.create table LogsTable (
    Timestamp: datetime,
    MachineId: string,
    LogLevel: string,
    ErrorCode: string,
    ResponseTime: int
)

.create table MachinesTable (
    MachineId: string,
    Location: string,
    MachineType: string
)
```

Run these commands in the Azure Data Explorer query editor to create the tables.

### 3. Ingest the Datasets
Ingest the CSV files into the corresponding tables. You can use one of the following methods:

#### Option A: Ingest via Azure Data Explorer UI
1. Open the Azure Data Explorer web UI and navigate to your database.
2. Select **Data** > **Ingest data**.
3. Choose **Upload files** and select a CSV file (e.g., `EventsTable.csv`).
4. Map the file to the corresponding table (e.g., `EventsTable`).
5. Ensure the CSV format is detected correctly (comma-separated, with headers).
6. Repeat for each CSV file.

#### Option B: Ingest via KQL Commands
Use the `.ingest` command to ingest the CSV files programmatically. Assuming the CSV files are accessible via a publicly available URL (or you can upload them to Azure Blob Storage), use commands like:

```kql
.ingest into table EventsTable @"path/to/EventsTable.csv" with (format="csv", ignoreFirstRecord=true)
.ingest into table UsersTable @"path/to/UsersTable.csv" with (format="csv", ignoreFirstRecord=true)
.ingest into table LogsTable @"path/to/LogsTable.csv" with (format="csv", ignoreFirstRecord=true)
.ingest into table MachinesTable @"path/to/MachinesTable.csv" with (format="csv", ignoreFirstRecord=true)
```

Replace `path/to/` with the actual file paths or URLs. The `ignoreFirstRecord=true` option skips the CSV header row.

If the files are local, upload them to Azure Blob Storage or use the UI method.

### 4. Verify Data Ingestion
Confirm that the data has been ingested correctly by running simple queries:

```kql
EventsTable | count
UsersTable | count
LogsTable | count
MachinesTable | count
```

Each table should return a count of 10 for `EventsTable`, 5 for `UsersTable`, 10 for `LogsTable`, and 4 for `MachinesTable`.

### 5. Run the KQL Queries
Use the KQL queries from the [*KQL Cooking Show* blog post](https://rodtrent.substack.com/p/kql-cooking-show-recipes-for-tasty) to generate insights:

- **Data Smoothie Query**:
  ```kql
  EventsTable
  | where Timestamp > ago(7d)
  | join kind=inner UsersTable on UserId
  | summarize AvgSessionDuration = avg(SessionDuration) by AgeGroup, Region
  | top 5 by AvgSessionDuration desc
  ```

- **Hearty Analytics Stew Query**:
  ```kql
  let TimeRange = ago(30d);
  LogsTable
  | where Timestamp > TimeRange
  | join kind=inner MachinesTable on MachineId
  | extend IsHighResponse = iif(ResponseTime > 500, 1, 0)
  | where LogLevel == "Error" and IsHighResponse == 1
  | summarize ErrorCount = count(), AvgResponseTime = avg(ResponseTime) by MachineType, Location, bin(Timestamp, 1h)
  | render timechart
  ```

Copy these queries into the Azure Data Explorer query editor and run them to see the results.

## Notes
- **Data Scale**: The datasets are intentionally small for ease of use. To scale up, generate additional rows with similar patterns and ingest them into the tables.
- **Timestamp Handling**: Ensure your Azure Data Explorer cluster is set to UTC to match the `Timestamp` values in the datasets (e.g., `2025-06-25T10:00:00Z`).
- **Query Optimization**: Follow the *Cooking Tips* from the [blog post](https://rodtrent.substack.com/p/kql-cooking-show-recipes-for-tasty) to optimize query performance, such as applying `where` clauses early and using appropriate join types.
- **Visualization**: The *Hearty Analytics Stew* query uses `render timechart`. View the results in the Azure Data Explorer UI to see the time-series chart.

## Troubleshooting
- **Ingestion Errors**: If ingestion fails, check the CSV file format (e.g., ensure no extra commas or missing headers) and verify the table schema matches the CSV columns.
- **Empty Query Results**: Ensure the data was ingested correctly and the `Timestamp` values align with the query’s time range (e.g., `ago(7d)` or `ago(30d)`).
- **Performance Issues**: If queries run slowly, check for missing indexes on columns like `Timestamp` or `UserId`. Use the `.create column policy` command to add indexes if needed.

## Additional Resources
- [Azure Data Explorer Documentation](https://docs.microsoft.com/en-us/azure/data-explorer/)
- [KQL Reference](https://docs.microsoft.com/en-us/azure/data-explorer/kusto/query/)
- The [*KQL Cooking Show* blog post](https://rodtrent.substack.com/p/kql-cooking-show-recipes-for-tasty) for the full context of the queries and recipes.

Happy cooking with KQL!
