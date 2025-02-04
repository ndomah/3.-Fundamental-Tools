# dbt for Data Engineers
## dbt Introduction & Setup
### The Modern Data Experience

![fig1]()

| Raw Data Ingestion | Develop, Test & Document, Deploy | Processed Datasets | Data Utilization |
|--------------------|--------------------------------|--------------------|-----------------|
| Data is collected from various sources (databases, APIs, logs, etc.). | dbt enables data transformation through SQL-based models. | Transformed data is stored in Snowflake, Redshift, BigQuery, or Databricks. | BI Tools (Tableau, Looker) use the data for reporting. |
| Raw data is unstructured and messy. | dbt runs tests to ensure data integrity and documents transformations. | Cleaned datasets become the single source of truth. | ML models utilize structured data for predictions. |
| Needs transformation before it becomes useful. | dbt schedules and deploys transformations in production. | Data is now ready for analytics and AI workloads. | Operational Analytics systems leverage the data for real-time decisions. |

### Introducing dbt

![fig2]()

| Step | Description | Tools Involved |
|------|------------|---------------|
| **1. Developer → dbt Core** | The developer writes and tests SQL-based transformations locally using dbt Core. | dbt Core |
| **2. Push to GitHub** | The developer pushes changes to GitHub for version control and collaboration. | GitHub |
| **3. Push to dbt Cloud** | GitHub integrates with dbt Cloud, allowing managed execution and online development. | dbt Cloud, GitHub |
| **4. Execution on Snowflake** | dbt runs transformations directly in the Snowflake data warehouse. | Snowflake, dbt Cloud/Core |
| **5. Continuous Workflow** | Developers iterate, test, and deploy changes in an automated pipeline. | dbt Core, dbt Cloud, GitHub, Snowflake |

**Benefits of dbt**
- SQL-based, allows a wide range of data practicioners to do the modeling
- Models can be chained together and executed in order where you have dependencies (DAGs)
- Allows implementing software engineering practices such as modular code, version control, testing, and continuous integration/deployment (CI/CD)

### Goals

![fig3]()

| Stage      | Goal | Concepts Covered |
|------------|------|------------------|
| **Connect (Ingest Data)** | Learn how to load raw data from different sources into a staging area. | - Connecting local files to **Snowflake Stage**  <br> - Using **dbt project repositories** <br> - Setting up **staging tables** in Snowflake |
| **Process (Transform Data with dbt)** | Understand how to use dbt to apply transformations as per business logic. | - Writing SQL models in **dbt** <br> - Implementing **staging & fact/dimension models** <br> - Applying **dbt tests, macros, and documentation** |
| **Store (Persist Data in Snowflake)** | Learn how to structure and store transformed data for analytics. | - Writing data into **staging tables** first <br> - Storing results in **dimension tables** <br> - Optimizing **Snowflake** for performance |
| **Visualize (Use Data for Analytics)** | Utilize transformed data in BI tools for reporting and decision-making. | - Connecting **dimension tables** to BI tools <br> - Using **Tableau, Looker, Power BI** <br> - Understanding how well-structured data improves analytics |

- We will create a series of pipelines for an e-commerce company using **dbt Core**, **dbt Cloud**, and **Snowflake**

![fig4]()

| Stage        | Description | Tables |
|-------------|------------|--------|
| **Raw Tables (Source Data)** | Store unprocessed data from the e-commerce platform in Snowflake. | `countries`, `ecommerceraw.DATA` |
| **Base Tables (Transformed Staging Layer)** | Clean and structure raw data using dbt models. | `invoices`, `items` |
| **Mart Tables (Business Intelligence Layer)** | Aggregate data into business-specific marts for analytics. | `owt`, `RFM Analysis` |

















