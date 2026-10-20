# ✈️ End-to-End Azure Data Factory Project: Medallion Architecture Pipeline
 
An end-to-end **data engineering project on Microsoft Azure** that ingests data from **three different sources** (GitHub, Azure SQL Database, and an on-premises/local drive), organises it in a **Bronze → Silver → Gold (Medallion) architecture** inside Azure Data Lake Storage, and serves **business-ready aggregated tables** for analytics. The whole workflow is orchestrated with **Azure Data Factory (ADF)** and monitored through an automated **Logic App email alert**.
 
![Azure](https://img.shields.io/badge/Azure-Data%20Factory-0078D4?logo=microsoftazure&logoColor=white)
![SQL](https://img.shields.io/badge/Azure-SQL%20Database-0078D4?logo=microsoftsqlserver&logoColor=white)
![Storage](https://img.shields.io/badge/ADLS-Gen2-5E5E5E)
![Logic Apps](https://img.shields.io/badge/Azure-Logic%20Apps-0078D4)
![GitHub](https://img.shields.io/badge/Source-GitHub-181717?logo=github)
 
---
 
## 📌 Table of Contents
 
1. [Project Overview](#-project-overview)
2. [Architecture](#-architecture)
3. [Tech Stack](#-tech-stack)
4. [Azure Resources](#-azure-resources)
5. [Data Model](#-data-model)
6. [Pipelines in Detail](#-pipelines-in-detail)
   - [Ingestion Layer (Bronze)](#1-ingestion-layer-bronze)
   - [Transformation Layer (Silver)](#2-transformation-layer-silver)
   - [Serving Layer (Gold)](#3-serving-layer-gold--business-views)
   - [Orchestration & Alerting](#4-orchestration--alerting)
7. [Key Concepts Demonstrated](#-key-concepts-demonstrated)
8. [How to Reproduce](#-how-to-reproduce)
9. [Project Structure](#-project-structure)
10. [Learnings & Future Improvements](#-learnings--future-improvements)
11. [Author](#-author)
---
 
## 🎯 Project Overview
 
Real-world data rarely lives in one place. This project simulates that scenario for an **airline bookings** dataset, where data is spread across:
 
| # | Source | Type | Ingestion Method |
|---|--------|------|------------------|
| 1 | **GitHub Repository** | REST / HTTP files | `Web` activity + `Copy data` (HTTP connector) |
| 2 | **Azure SQL Database** | Relational database | `Lookup` + `Copy data` with **incremental (watermark) load** |
| 3 | **Local PC / On-Premises Drive** | File system | **Self-hosted Integration Runtime** + `ForEach` + `Copy data` |
 
All three sources are consolidated into a **Data Lake**, cleaned and transformed using **ADF Mapping Data Flows**, and finally aggregated into **Gold-layer business views** (e.g., revenue per airline, Top 5 airlines).
 
### ✨ Highlights
- ✅ Multi-source ingestion (cloud, database, on-prem)
- ✅ **Medallion architecture** (Bronze / Silver / Gold)
- ✅ **Incremental loading** using a watermark table
- ✅ **Parameterised, dynamic** pipelines with `ForEach` loops
- ✅ Code-free transformations with **Mapping Data Flows**
- ✅ Master **orchestration pipeline** with a scheduled trigger
- ✅ Automated **email alert** on pipeline status via **Logic Apps + Outlook**
- ✅ Source control with **GitHub integration** (`main` branch)
---
 
## 🏗️ Architecture
 
> 📷 Replace the URL below with your architecture diagram (if you have one).
 
![Architecture Diagram](https://res.cloudinary.com/dh5wlux0q/image/upload/v1792469373/Screenshot_2026-10-20_093917_akwrmx.png)
 
```text
                    ┌──────────────────────┐
                    │   Orchestration      │──── Trigger (scheduled)
                    │   Pipeline (ADF)     │
                    └─────────┬────────────┘
                              │
      ┌───────────────────────┼───────────────────────┐
      ▼                       ▼                       ▼
┌────────────┐        ┌──────────────┐        ┌──────────────┐
│ On-Prem /  │        │   GitHub     │        │ Azure SQL DB │
│ Local PC   │        │   (HTTP/API) │        │ (Watermark)  │
└─────┬──────┘        └──────┬───────┘        └──────┬───────┘
      │  Self-hosted IR      │  Web + Copy           │  Lookup + Copy
      └──────────────────────┼───────────────────────┘
                             ▼
                 ┌──────────────────────┐
                 │  🥉 BRONZE (Raw)     │  Azure Data Lake (ADLS Gen2)
                 └──────────┬───────────┘
                            ▼   Mapping Data Flow: DataTransformation
                 ┌──────────────────────┐
                 │  🥈 SILVER (Cleaned) │  Standardised dimensions & facts
                 └──────────┬───────────┘
                            ▼   Mapping Data Flow: DataServing
                 ┌──────────────────────┐
                 │  🥇 GOLD (Business)  │  Aggregated, analytics-ready views
                 └──────────┬───────────┘
                            ▼
                 Logic App ➜ Outlook Email (Success / Failure Alert)
```
 
---
 
## 🛠️ Tech Stack
 
| Category | Technology |
|----------|-----------|
| **Cloud Platform** | Microsoft Azure (Central India region) |
| **Orchestration / ETL** | Azure Data Factory (V2) |
| **Storage** | Azure Data Lake Storage Gen2 (Storage Account) |
| **Database** | Azure SQL Database + SQL Server |
| **Transformation** | ADF Mapping Data Flows (Spark-backed) |
| **Alerting** | Azure Logic Apps + Outlook API Connection |
| **Connectivity** | Self-hosted Integration Runtime (on-prem), HTTP connector (GitHub) |
| **Version Control** | Git / GitHub (ADF Git integration) |
 
---
 
## ☁️ Azure Resources
 
All resources are deployed in a single resource group, **`adf-project`**, in **Central India**.
 
![Azure Resource Group](https://res.cloudinary.com/dh5wlux0q/image/upload/v1792468714/Screenshot_2026-10-18_225435_lztmfk.png)
 
| Resource | Type | Purpose |
|----------|------|---------|
| `adf-project-adf` | Data Factory (V2) | Pipelines, data flows, triggers |
| `adf-project-srvr` | SQL Server | Hosts the SQL database |
| `adf-project-db` | SQL Database | Source database + watermark/metadata tables |
| `adfprojectstoragee` | Storage Account (ADLS Gen2) | Data lake – Bronze, Silver, Gold containers |
| `pipeline_runs_status` | Logic App | Receives pipeline status and sends email |
| `outlook` | API Connection | Outlook connector used by the Logic App |
 
---
 
## 🗂️ Data Model
 
The dataset follows a **star schema** for flight bookings.
 
| Table | Type | Description |
|-------|------|-------------|
| `FactBookings` | **Fact** | Booking transactions with cost / amount |
| `DimAirline` | Dimension | Airline ID, name, country |
| `DimFlight` | Dimension | Flight ID, flight number and related attributes |
| `DimPassenger` | Dimension | Passenger ID, full name, gender, age, country |
| `DimAirport` | Dimension | Airport ID, name, city, country |
 
---
 
## 🔄 Pipelines in Detail
 
The factory contains **6 pipelines**, organised into folders:
 
```text
Pipelines
├── __ORCHESTRATION__
│   └── orchrstration          ← Master pipeline
├── DataFlow_pipeline
│   ├── SilverLayer            ← Runs DataTransformation data flow
│   └── BusinessView           ← Runs DataServing data flow
├── github-data-load
│   └── git-data-load
├── onprem-data-load
│   └── onprem-data-load
└── sql-data-load
    └── sql-data-load
```
 
### 1. Ingestion Layer (Bronze)
 
Raw data is landed **as-is** in the Bronze container of the data lake.
 
#### 🔹 a) GitHub → Data Lake (`git-data-load`)
- A **`Web` activity (`WebAPICall`)** calls the GitHub source.
- On success, a **`Copy data` activity (`CopyGitHub`)** copies the files using the dataset `ds_github_src` into the lake.
![GitHub Pipeline](https://res.cloudinary.com/dh5wlux0q/image/upload/v1792468713/Screenshot_2026-10-18_230348_-_Copy_p5c9hz.png)
 
#### 🔹 b) Azure SQL Database → Data Lake (`sql-data-load`) — *Incremental Load*
- Two **`Lookup`** activities (`LastLoad`, `LatestLoad`) fetch the previously loaded watermark and the newest available value.
- **`CopySqlData`** copies **only new/changed records** between those two values.
- A second **`Copy data` (`Watermark`)** and a **`Set variable`** activity update the watermark, so the next run continues from where this one stopped.
> 💡 This avoids re-loading the full table every time, which saves cost and time.
 
![SQL Incremental Pipeline](https://res.cloudinary.com/dh5wlux0q/image/upload/v1792468713/Screenshot_2026-10-18_230444_-_Copy_efjxum.png)
 
#### 🔹 c) On-Premises / Local PC → Data Lake (`onprem-data-load`)
- Uses a **Self-hosted Integration Runtime** to securely connect the local machine to Azure.
- A **`ForEach` activity (`ForEachFile`)** loops over the files and runs a `Copy data` activity (`MigrateOnPremData`) for each one, which makes the pipeline **dynamic and scalable**.
![On-Prem Pipeline](https://res.cloudinary.com/dh5wlux0q/image/upload/v1792468799/Screenshot_2026-10-18_230416_-_Copy_hwbpw1.png)
 
---
 
### 2. Transformation Layer (Silver)
 
**Data Flow:** `DataTransformation` — cleans and standardises each entity from Bronze and writes to Silver.
 
| Entity | Transformations Applied |
|--------|------------------------|
| **DimAirline** | Derived column `country` → Alter Row → Sink |
| **DimFlight** | Select / rename columns → Alter Row → Sink |
| **DimPassenger** | Select gender flag → derive columns for male / female → derive `first_name` → Alter Row → Sink |
| **DimAirport** | Derived columns for airport name, city, country → Alter Row → Sink |
| **FactBookings** | Cast columns to correct data types (e.g., cost) → Alter Row → Sink |
 
**Techniques used:** Derived Column, Select (rename), Cast, Alter Row (upsert logic), multiple Sinks.
 
![Silver Layer Data Flow](https://res.cloudinary.com/dh5wlux0q/image/upload/v1792468713/Screenshot_2026-10-18_230729_-_Copy_l0vddn.png)
 
---
 
### 3. Serving Layer (Gold) — Business Views
 
**Data Flow:** `DataServing` — reads cleaned Silver data and creates **aggregated, business-ready tables**.
 
- **Join** `DimBookings` with `DimAirline` (Left Outer Join) and `DimFlight`
- **Select** only the required columns
- **Aggregate** metrics by `airline_name` (e.g., total bookings and total cost)
- **Window function (Rank)** to rank airlines by revenue
- **Filter** to keep the **Top 5 airlines**
- **Alter Row** + **Sink** to write to the Gold layer
**Example business questions answered:**
- Which airlines generate the most revenue?
- Who are the Top 5 airlines by total booking cost?
- How are bookings distributed per airline?
![Gold Layer Data Flow](https://res.cloudinary.com/dh5wlux0q/image/upload/v1792468713/Screenshot_2026-10-18_230659_-_Copy_aukcbm.png)
 
---
 
### 4. Orchestration & Alerting
 
The master pipeline **`orchrstration`** chains the ingestion pipelines in sequence using **`Execute Pipeline`** activities:
 
```text
ExecuteOnPrem  ──▶  ExecuteAPI-pipe  ──▶  ExecuteSql-pipe  ──▶  Web (Alert)
```
 
- A **scheduled trigger** is attached so the whole flow runs automatically.
- The final **`Web` activity (`Alert`)** sends a `POST` request with the pipeline name and status to a **Logic App HTTP endpoint**.
- The **Logic App (`pipeline_runs_status`)** then sends a formatted **email via Outlook**, so success or failure is known without opening the portal.
![Orchestration Pipeline](https://res.cloudinary.com/dh5wlux0q/image/upload/v1792468714/Screenshot_2026-10-18_230551_-_Copy_bxdmq3.png)
 
> ![Alert Email](https://res.cloudinary.com/dh5wlux0q/image/upload/v1792469040/Screenshot_2026-10-18_231639_xlxqdp.png)
 
---
 
## 🧠 Key Concepts Demonstrated
 
| Concept | Where It's Used |
|---------|-----------------|
| Medallion Architecture | Bronze → Silver → Gold in ADLS Gen2 |
| Incremental / Delta Loading | Watermark logic in `sql-data-load` |
| Dynamic Pipelines | `ForEach` loop in `onprem-data-load` |
| Hybrid Connectivity | Self-hosted Integration Runtime |
| REST/HTTP Ingestion | `Web` + `Copy data` for GitHub |
| Data Cleansing & Modelling | Mapping Data Flows (Derived Column, Cast, Select) |
| Aggregation & Ranking | Aggregate, Window (Rank), Filter in `DataServing` |
| Pipeline Orchestration | Master pipeline using `Execute Pipeline` |
| Monitoring & Alerting | Web activity → Logic App → Outlook email |
| CI/CD Readiness | ADF connected to GitHub (`main` branch) |
 
---
 
## 🚀 How to Reproduce
 
1. **Create the resource group** `adf-project` in Azure.
2. **Deploy resources:** Data Factory, Storage Account (enable hierarchical namespace), SQL Server + SQL Database, Logic App.
3. **Create containers** in the storage account: `bronze`, `silver`, `gold`.
4. **Load the source data:** upload the sample files to GitHub, insert the tables into Azure SQL, and keep the files on your local drive.
5. **Install a Self-hosted Integration Runtime** on your local machine and register it in ADF.
6. **Create linked services & datasets** (HTTP, Azure SQL, File System, ADLS Gen2).
7. **Build the pipelines:** `git-data-load`, `sql-data-load`, `onprem-data-load`.
8. **Create the data flows:** `DataTransformation` (Silver) and `DataServing` (Gold), then wrap them in `SilverLayer` and `BusinessView` pipelines.
9. **Configure the Logic App** with an HTTP trigger and Outlook "Send an email" action.
10. **Build the orchestration pipeline** and attach a trigger.
11. **Publish** and run a **Trigger Now** to test.
---
 
## 📁 Project Structure
 
```text
.
├── README.md
├── dataset/                 # Sample source files (CSV)
├── pipeline/                # ADF pipeline JSON
├── dataflow/                # Mapping data flow JSON
├── dataset_definitions/     # ADF dataset JSON
├── linkedService/           # Linked service JSON (secrets removed)
├── trigger/                 # Trigger JSON
└── images/                  # Screenshots used in this README
```
 
> *(Adjust folder names to match your repo, since ADF's Git integration generates these automatically.)*
 
---
 
## 📈 Learnings & Future Improvements
 
**What I learned**
- Designing a multi-source ingestion framework in ADF
- Implementing incremental loads with a watermark pattern
- Building scalable Mapping Data Flows
- Connecting on-premises data securely to the cloud
- Setting up monitoring and alerts for production-style pipelines
**Planned enhancements**
- [ ] Add **Azure Key Vault** for secrets management
- [ ] Replace sinks with **Delta Lake** format
- [ ] Add **data quality checks** and a failure/retry framework
- [ ] Connect the Gold layer to **Synapse Analytics / Power BI** dashboards
- [ ] Rebuild the transformations in **PySpark on Databricks**
- [ ] Add **CI/CD** using Azure DevOps or GitHub Actions
---
 
## 👨‍💻 Author
 
**Golu**
BCA Student | Aspiring Data Engineer | Azure • PySpark • SQL • Python
 
- 🔗 GitHub: [@Priyesh47data](https://github.com/Priyesh47data)
- 💼 LinkedIn: [Add your LinkedIn URL](ADD_LINKEDIN_URL_HERE)
- 📧 Email: ADD_YOUR_EMAIL_HERE
---
 
⭐ *If you found this project useful, please consider giving it a star!*
