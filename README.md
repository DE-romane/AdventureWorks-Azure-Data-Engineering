### Adventure Works -end to end Azure data enginnering poject

####  Project Overview

This project demonstrates a complete **Azure-based data engineering pipeline** using real-world practices.  
Data is ingested from the **GitHub API** and processed through a **Medallion architecture** (Bronze, Silver, Gold).  
**Azure Data Factory** orchestrates ingestion, while **Databricks** handles transformations.  
**Synapse Analytics** stores the final curated data, and **Power BI** is used for reporting.  
The goal is to build a **scalable, automated, and production-ready solution** using key Azure services.

![[project -Azure/p/img/1.png]]





#### Data Architecture

A solid architecture is the foundation of a successful data engineering project. This solution follows the **Medallion architecture**, comprising three main layers:

- **Bronze (Raw Layer)**:
    - Stores raw data directly from the source (GitHub API).
    - Acts as the landing zone for ingested data.
- **Silver (Transformed Layer)**:
    - Data is cleaned, filtered, and transformed using **Azure Databricks**.
    - Adds structure and usability for analytics.
- **Gold (Serving Layer)**:
    - Final data curated and optimized for reporting and stakeholder access.
    - Loaded into **Azure Synapse Analytics** for performance querying.

>  **Source**: Data is fetched via **HTTP connection** from GitHub API to mimic real-world data ingestion.

---

####  Azure Tools Used

##### Azure Data Factory
- A powerful orchestration tool with low-code/no-code capabilities.
- Handles ingestion from GitHub API into the **Bronze** layer.
- Implements **dynamic pipelines**, **parameters**, and **loops** to scale data flows.
- Also demonstrated within **Synapse Analytics** as **Synapse Pipelines**.

##### Azure Databricks
- Uses **Apache Spark** for big data processing.
- Transforms data from **Bronze to Silver** with PySpark notebooks and custom logic.
- Efficient for both batch and interactive data processing.

#####  Azure Synapse Analytics
- Builds and stores the **Gold layer** in a serverless  SQL pool.
- Enables fast querying and integration with downstream tools.

##### 📈 Power BI
- Connects directly to Synapse Analytics for dashboarding and reporting.

#### Data Understanding 
🔗 **Source**: [AdventureWorks Dataset on Kaggle](https://www.kaggle.com/datasets/ukveteran/adventure-works)

- The project uses the **AdventureWorks dataset**, known for its rich relational structure.
- Chosen for its **real-world relevance**, allowing practice with joins, lookups, transformations, and multi-table relationships.
- Includes **sales data (2015–2017)**, **returns**, and multiple dimension tables like customers, products, and territories.
- The dataset is pre-loaded into a **GitHub repository** to simulate **API-based ingestion** via Azure Data Factory.
---

#####  Dataset Overview
- **Calendar Table**: Date-related data for building time-based dimensions.
- **Customers Table**: Customer details including name, address, and birthdate.
- **Product-Related Tables**:
    - **Product Categories & Subcategories**: Define product hierarchy.
    - **Products**: Core product information used in sales and returns.
- **Sales Data**: Fact table covering 2015–2017 sales transactions.
- **Returns Data**: Tracks returned items, enabling more complex reporting.
- **Territories Table**: Adds geographic segmentation for analysis.

#### Phase 1 – Ingesting Data from GitHub API to Bronze Layer

#####  Objective

The focus of **Phase 1** is to ingest data from a **GitHub API** and land it into the **Bronze layer** of an **Azure Data Lake**.

---

#####  Resource Group Creation

- A dedicated **Resource Group** named `aw_project` is created to organize all Azure resources under one logical unit.


---

##### Storage Account Setup

- A **Storage Account** is provisioned with **Hierarchical Namespace enabled**, allowing it to function as **Azure Data Lake Storage Gen2**.

- **Three containers** are created inside the Data Lake to support the **Medallion Architecture**:
    - `bronze` → Raw data from the GitHub API
    - `silver` → Transformed data
    - `gold` → Data ready for reporting and analysis
![[project -Azure/p/img/2.png]]

---

#####  Azure Data Factory (ADF)

- **Azure Data Factory** is created under the `aw_project` resource group to orchestrate data ingestion workflows.

#####  Linked Services
**Purpose**: Establishes connections between ADF and external systems (source and destination). 
- **GitHub API (Source)**:
    - Connector Type: `HTTP`
    - Name: `HttpLinkService`
- **Azure Data Lake Destination**:
    - Connector Type: `Azure Data Lake Storage Gen2`
    - Name: `StorageDataLink`
![[project -Azure/p/img/3.png]]

---
#####  Pipeline Configuration
![[4.png]]
**Lookup Activity (`LookupGit`)**

- Retrieves a control JSON file (`git.json`) from storage  account, which contains parameters for each file to ingest.
- Example of the JSON structure:
```json
{
  "p_rel_url": "DE-romane/AdventureWorks-Azure-Data-Engineering/main/Data/AdventureWorks_Product_Categories.csv",
  "p_sink_folder": "AdventureWorks_Product_Categories",
  "p_sink_file": "AdventureWorks_Product_Categories.csv"
}
```

---

  ForEach Activity (`ForEachGit`)
- Iterates over all parameters returned from `LookupGit`.
- Each iteration:
    - Executes a **Copy Data activity** (`DynamicCopy`)
    - Copies the corresponding file from GitHub to the appropriate folder in the **Bronze container**
        

📂 **Result**: Each file from GitHub is dynamically loaded and stored as-is into its respective subfolder under the **Bronze layer**, mimicking real-world data ingestion patterns.
![[5.png]]


#### Phase 2 – Transforming Bronze to Silver Using Databricks

##### 🎯 Objective
In **Phase 2**, data is read from the **Bronze layer**, cleaned and transformed using **Azure Databricks**, and then written into the **Silver layer** of the Data Lake. This reflects typical **ETL processing** in a big data pipeline.

##### Databricks Workspace Setup
- A **Databricks workspace** is created within the existing **`aw_project` Resource Group**.
- This workspace is used to run **PySpark notebooks** that transform raw data from Bronze to structured Silver format.

**Accessing Azure Data Lake**
To enable **Databricks to access Azure Data Lake (ADLS Gen2)** securely:
- A **Service Principal** (`aw_project_app`) is created using **Microsoft Entra ID** 
- Required permissions are granted to the service principal for scoped access to the storage account.
- Credentials are securely stored in **Databricks secrets** for use in the notebooks.
![[project -Azure/p/img/6.png]]

![[project -Azure/p/img/7.png]]

- A **Databricks cluster** is created to run transformation workloads.
   
---

##### 📓 Transformation Notebook

A **Databricks Notebook** is developed for this phase. It performs the following tasks:    
2. **Reads data** from the **Bronze container**.
3. Applies **data cleaning and transformation
4. **Writes the processed data** into corresponding folders in the **Silver container**.


---

✅ **Outcome**:  
Structured and cleansed data now resides in the **Silver layer**, ready for analytical modeling or warehousing in the next phase.
![[Pasted image 20250809183815.png]]


#### Phase 3: Data Warehousing & Visualization (Azure Synapse Analytics & Power BI) 
setting up **Azure Synapse Analytics** to query and serve data from the **Silver layer** of an Azure Data Lake, create a **serverless SQL database**, and enable visualization with **Power BI**. A **Synapse workspace** is created with a managed identity for secure data access, eliminating the need for a service principal. The **lakehouse** concept is used to query Data Lake files (Parquet format) with **OPENROWSET**, creating **views** and **external tables** in the **Gold layer** for reporting. 

The serverless SQL endpoint connects to Power BI for data visualization, emphasizing practical skills like managed identities, lakehouse architecture, and Synapse’s unified platform for data warehousing, integration, and analytics.


- Create Synapse workspace with default storage.
- Grant access to Data Lake using managed identity (Storage Blob Data Contributor role for workspace and user).
- Create serverless SQL database (e.g., aw_database), schema (e.g., gold), and views using OPENROWSET to query Silver data.
- Create external tables with CETAS to store data in Gold layer (using credentials, external data sources, file formats).
- Copy serverless SQL endpoint from Synapse overview.
- In Power BI Desktop, connect using the endpoint and SQL credentials; load external tables (e.g., gold.ext_sales).
- Build simple visualizations (e.g., line charts for trends, cards for KPIs) to demonstrate data serving.

### Overview 🌟
This segment, **Phase 3** of the Azure Data Engineer project, focuses on setting up **Azure Synapse Analytics** to query and serve data from the **Silver layer** of an Azure Data Lake, create a **serverless SQL database**, and enable visualization with **Power BI**. A **Synapse workspace** is created with a managed identity for secure data access, eliminating the need for a service principal. The **lakehouse** concept is used to query Data Lake files (Parquet format) with **OPENROWSET**, creating **views** and **external tables** in the **Gold layer** for reporting. The serverless SQL endpoint connects to Power BI for data visualization, emphasizing practical skills like managed identities, lakehouse architecture, and Synapse’s unified platform for data warehousing, integration, and analytics.

 **Steps** 
1. **Create Synapse Analytics Workspace** :
2. **Configure Data Access (Managed Identity)** :
3. **Create Serverless SQL Database**() :
Synapse offers **serverless SQL pools** (for lakehouse architecture, querying Data Lake data without storing it in a database
![[8.png]]
4. **Create Schema and Views** 🗂️:
   - Create a schema to organize views in the Gold layer.
```sql
CREATE SCHEMA gold;
```
   - Create views: Views store queries (not data) for reusable access to Data Lake data.
   - here the link 

6. **Create External Tables** 📊:
-----------------
   - external tables store data in the Data Lake (Gold layer) for persistent access.

  - **Prerequisites**:
1. **Master Key**: Create a database master key for security:
``` sql
-- Create a master key to secure credentials in the database

-- This is required before creating database scoped credentials for external data access

CREATE MASTER KEY ENCRYPTION BY PASSWORD ='romany&&&1'

GO
```
2. **Database-Scoped Credential**: Define a credential using managed identity:
```sql
-- Create a database scoped credential to authenticate using Managed Identity

-- This credential will be used by Synapse to access external data sources without embedding secrets

CREATE DATABASE SCOPED CREDENTIAL cred_rom
WITH IDENTITY = 'Managed Identity'
GO
```

3. **External Data Source**: Create data sources for Silver and Gold containers:
```sql
-- Create an external data source pointing to the Silver layer in Azure Data Lake Storage
-- This defines the location and credential to read data from the silver container
CREATE EXTERNAL DATA SOURCE source_silver
WITH (
    LOCATION ='https://awdatalakemain.blob.core.windows.net/silver',
    CREDENTIAL=cred_rom
)
GO
-- Create an external data source pointing to the Gold layer in Azure Data Lake Storage

-- This will be used to store or query data in the gold container

CREATE EXTERNAL DATA SOURCE source_gold
WITH (
    LOCATION ='https://awdatalakemain.blob.core.windows.net/gold',
    CREDENTIAL=cred_rom
)
GO
```

4. **External File Format**: Define the file format (e.g., Parquet with Snappy compression):
```sql
-- Define an external file format for reading/writing Parquet files

-- Snappy compression is specified since it's commonly used in Parquet for optimized performance

CREATE EXTERNAL FILE FORMAT format_parquet
WITH (
       FORMAT_TYPE = PARQUET
      , DATA_COMPRESSION = 'org.apache.hadoop.io.compress.SnappyCodec'
    )
GO
```
  - **External Table Creation**:
- Use **CETAS (CREATE EXTERNAL TABLE AS SELECT)** to create an external table from a view and store data in the Gold layer:
```sql
CREATE EXTERNAL TABLE gold.extsales

WITH

(

    LOCATION='extsales',
    DATA_SOURCE=source_gold,
    FILE_FORMAT=format_parquet
)
AS
SELECT * FROM gold.sales
```
---------------------------------------------------
7. **Connect Synapse to Power BI** 📈:
-   **Purpose**: Enable data analysts to visualize Gold layer data using Power BI.
   - Copy **Serverless SQL Endpoint** from Synapse Studio’s workspace overview.
   - In Power BI Desktop, go to **Get Data** > **Azure** > **Azure Synapse Analytics SQL > Paste endpoint, enter database (`aw_database`), and SQL credentials.
   ![[9.png]]
   - Load external table (e.g., `gold.ext_sales`) and create visualizations (e.g., line chart for `order_date` vs. `order_number`, card for `customer_key` count).
![[10.png]]

