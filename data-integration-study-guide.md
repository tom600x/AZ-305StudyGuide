# Azure Data Integration & Analysis — Deep-Dive Study Guide for AZ-305

## Why Data Integration Matters on AZ-305
The exam tests your ability to:
- Choose between Azure Data Factory, Synapse Analytics, and Databricks
- Understand when to use Stream Analytics for real-time data
- Design data pipelines (batch vs streaming)
- Select the right tool for Extract, Transform, Load (ETL)/Extract, Load, Transform (ELT), big data analytics, and reporting

---

## 1. Azure Data Factory (ADF)

### What It Is
Azure Data Factory is a **cloud-based ETL and data integration service** for building data pipelines. It moves and transforms data from source systems to target destinations.

### Key Concepts

| Concept | Description |
|---|---|
| **Pipeline** | Container for activities that perform a unit of work |
| **Activity** | A single step in a pipeline (Copy, Transform, Control flow) |
| **Dataset** | Named reference to data (tables, files, folders) |
| **Linked Service** | Connection string to a data source or destination |
| **Trigger** | Defines when a pipeline runs (schedule, tumbling window, event-based) |
| **Integration Runtime (IR)** | The compute infrastructure that executes the activities |

### Integration Runtime Types

| IR Type | Description | Use Case |
|---|---|---|
| **Azure IR** | Fully managed, serverless cloud compute | Data between Azure services or public endpoints |
| **Self-Hosted IR** | Installed on-premises or in a private VM | Connect to on-premises data sources or private network resources |
| **Azure-SSIS IR** | Dedicated compute cluster to run SSIS packages | Lift-and-shift existing SQL Server Integration Services (SSIS) packages |

> **Exam tip:** If the scenario mentions on-premises data sources, the answer is **Self-Hosted IR**. If the scenario mentions existing SSIS packages, the answer is **Azure-SSIS IR**.

### Trigger Types

| Trigger | Description | Use Case |
|---|---|---|
| **Schedule** | Run at fixed intervals or times | Daily ETL batch jobs |
| **Tumbling Window** | Recurring windows of fixed size, retryable | Time-series data, hourly/daily processing |
| **Event-Based** | Triggered by blob creation/deletion in Storage | React to file arrival |
| **Manual** | On-demand | One-time data loads |

### Copy Activity
- Moves data between 90+ supported data stores (files, databases, SaaS)
- Supports schema mapping, data type conversion, column filtering
- Parallel copy with degree of copy parallelism control

### Mapping Data Flows
- Visually designed ETL transformations (no code)
- Runs on Spark clusters (managed by ADF)
- Supports joins, aggregations, window functions, schema drift

### Data Factory vs SSIS

| | ADF | SSIS (Azure-SSIS IR) |
|---|---|---|
| Execution model | Cloud-native, serverless | Runs SSIS packages as-is |
| Migration path | Redesign pipelines | Lift-and-shift existing |
| Monitoring | ADF Monitor, Azure Monitor | SSMS, SSISDB |

---

## 2. Azure Synapse Analytics

### What It Is
Azure Synapse is an **enterprise analytics service** that combines big data and data warehousing. It unifies **data integration, data warehousing, and big data analytics** into a single workspace.

### Key Components

| Component | Description | Use Case |
|---|---|---|
| **Dedicated SQL Pools** | Massively Parallel Processing (MPP) SQL data warehouse | Large-scale data warehousing, historical analytics |
| **Serverless SQL Pool** | Query data in Data Lake with SQL, pay per TB scanned | Ad-hoc queries, data exploration, no loading needed |
| **Apache Spark Pools** | Managed Spark clusters for big data processing | Machine learning, complex transformations, Delta Lake |
| **Synapse Pipelines** | Fully ADF-compatible pipelines built into Synapse | Data integration within Synapse workspace |
| **Synapse Link** | Zero-ETL analytical access to operational data in Cosmos DB, SQL | Hybrid Transactional and Analytical Processing (HTAP) |

### Dedicated SQL Pool (formerly SQL DW)

- Stores data in **columnar format** across 60 distributions for parallel processing
- **Distribution types:**
  - **Hash:** Distributes rows by a specified column — best for large fact tables
  - **Round Robin:** Distributes rows evenly — best for staging tables, no clear join key
  - **Replicated:** Copies entire table to each node — best for small dimension tables
- **Data Warehouse Unit (DWU):** Scaling unit — scale up/down or pause to save cost
- **Control table vs Distribution:** Design for minimizing data movement during joins

### Serverless SQL Pool
- No provisioning required, always available
- Queries files directly in ADLS Gen2 (Parquet, CSV, JSON, Delta)
- Cost: per TB of data scanned
- Great for creating external tables (views over data lake files)

### Synapse vs Cosmos DB Synapse Link (HTAP)
- Synapse Link connects Cosmos DB to a Synapse Spark or Serverless SQL Pool
- Analytical workloads do not impact transactional performance in Cosmos DB
- Data is automatically synced to an **analytical store** (columnar format)
- Zero-ETL — no pipeline needed

---

## 3. Azure Databricks

### What It Is
Azure Databricks is a **managed Apache Spark platform** for large-scale data engineering, machine learning, and collaborative analytics. Built on Spark, optimized for Azure.

### Key Concepts

| Concept | Description |
|---|---|
| **Workspace** | Collaborative environment for notebooks, jobs, and data |
| **Cluster** | Spark compute (all-purpose or job clusters) |
| **Notebook** | Interactive code (Python, R, Scala, SQL) |
| **Job** | Scheduled automated execution of notebooks/JARs |
| **Delta Lake** | Atomicity, Consistency, Isolation, Durability (ACID)-compliant storage layer on top of Parquet files |

### Delta Lake
- Adds ACID transactions to data lake storage (Azure Data Lake Storage Gen2)
- Schema enforcement and evolution
- Time travel — query historical snapshots of data
- Upserts (MERGE operations) on data lake files

### Databricks vs Synapse Spark

| Feature | Azure Databricks | Synapse Spark |
|---|---|---|
| ML/AI capabilities | MLflow integration, AutoML | Basic ML |
| Delta Lake | Native, optimized | Supported |
| Collaboration | Best-in-class notebooks | Good |
| Integration with Synapse | Via linked services | Native |
| Cost control | More granular | Simpler |
| Best For | ML workloads, complex ETL | Analytics within Synapse ecosystem |

---

## 4. Azure Stream Analytics

### What It Is
Azure Stream Analytics is a **real-time event processing and analytics service**. Processes streaming data from Event Hubs, IoT Hub, or Blob Storage using a SQL-like query language.

### Key Concepts

| Concept | Description |
|---|---|
| **Input** | Event Hubs, IoT Hub, Blob/ADLS |
| **Output** | Power BI, SQL Database, Cosmos DB, Event Hubs, Blob, Functions |
| **Query** | SQL-like SAQL (Stream Analytics Query Language) |
| **Windowing** | Tumbling, Hopping, Sliding, Session, Snapshot windows |
| **Streaming Units (SU)** | Scale unit for processing power |

### Windowing Functions

| Window Type | Description | Use Case |
|---|---|---|
| **Tumbling** | Fixed, non-overlapping intervals | Count events per minute |
| **Hopping** | Fixed size, overlapping intervals | 10-minute averages updated every 1 minute |
| **Sliding** | Window emitted when event occurs | Event-triggered aggregations |
| **Session** | Grouped by activity (gap between events) | User session analysis |

---

## 5. Choosing the Right Tool

| Scenario | Recommended Tool |
|---|---|
| Move data from on-premises SQL Server to Azure Data Lake daily | Azure Data Factory (Self-Hosted IR) |
| Lift-and-shift existing SSIS packages to Azure | ADF with Azure-SSIS IR |
| Large data warehouse with historical reporting (TB-scale) | Synapse Dedicated SQL Pool |
| Explore files in Data Lake with SQL, no loading | Synapse Serverless SQL Pool |
| Real-time IoT anomaly detection, alerts | Azure Stream Analytics |
| Machine learning pipeline on large datasets | Azure Databricks |
| ACID transactions on data lake files | Delta Lake (Databricks or Synapse Spark) |
| Analyze Cosmos DB data without impacting app performance | Synapse Link |
| Orchestrate complex multi-step data workflows | Azure Data Factory pipelines |
| Event-based pipeline trigger (file lands in Blob) | ADF Event Trigger |

---

## 6. Data Integration Architecture Patterns

### Lambda Architecture
- **Batch layer:** Historical data processing (ADF + Synapse)
- **Speed layer:** Real-time processing (Stream Analytics, Event Hubs)
- **Serving layer:** Query layer (Synapse, Power BI)

### Modern Data Lakehouse Pattern
1. Ingest raw data → ADLS Gen2 (raw zone)
2. Transform with ADF or Databricks → processed zone
3. Serve with Synapse SQL Pool or Synapse Serverless → gold zone
4. Report with Power BI

---

## 7. Exam Scenario Cheat Sheet

| Scenario | Answer |
|---|---|
| Orchestrate data movement from 10+ sources to data lake | Azure Data Factory |
| Run existing SSIS packages in Azure | ADF + Azure-SSIS Integration Runtime |
| On-premises database as source, needs secure connectivity | ADF + Self-Hosted Integration Runtime |
| Real-time fraud detection on payment streams | Azure Stream Analytics |
| TB-scale historical reporting, complex aggregations | Synapse Dedicated SQL Pool |
| Data scientists need collaborative notebooks on Spark | Azure Databricks |
| Query CSV/Parquet files in ADLS without ETL | Synapse Serverless SQL Pool |
| ACID transactions and time travel on data lake | Delta Lake |
| Cosmos DB operational data analyzed without impacting app | Synapse Link + Synapse Analytics |
| React to blob file arrival, start pipeline | ADF Event-Based Trigger |
