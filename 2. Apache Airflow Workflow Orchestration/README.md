# Apache Airflow Workflow Orchestration
## Introduction
### Airflow Usage

![fig1 - airflow usage](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/2.%20Apache%20Airflow%20Workflow%20Orchestration/img/fig1%20-%20airflow%20usage.png)

This image represents an **Apache Airflow DAG (Directed Acyclic Graph)**, a key concept in orchestrating data workflows.

**1. DAG Structure in Airflow**
- The diagram shows **tasks** in an **ETL (Extract, Transform, Load)** pipeline.
- Tasks are connected using **dependencies** (arrows), defining execution order.

**2. Breakdown of Tasks**
- **extract** → Retrieves raw data.
- **transform** → Cleans or processes the extracted data.
- **load** → Stores the transformed data into a database or warehouse.
- **query_print** → Likely executes a query or prints results.

**3. Key Airflow Concepts Demonstrated**
- **Operators**: Each box represents a task, typically implemented using Airflow **operators** like:
  - `PythonOperator` (for Python functions)
  - `BashOperator` (for shell commands)
  - `SqlOperator` (for database queries)
- **Task Dependencies**:
  - `extract` runs first.
  - `transform` depends on `extract`.
  - `load` and `query_print` both depend on `transform`, meaning they run in parallel after it completes.
- **DAG Execution**:
  - Airflow schedules and executes tasks in the defined sequence, ensuring dependencies are respected.

## Airflow Fundamental Concepts
### Fundamental Concepts

![fig2 - fundamental concepts](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/2.%20Apache%20Airflow%20Workflow%20Orchestration/img/fig2%20-%20fundamental%20concepts.png)

**1. DAG (Directed Acyclic Graph)**
A **DAG** is the **overall workflow definition** in Airflow. It consists of multiple tasks and defines their dependencies.

- DAGs **do not execute tasks directly** but organize them.
- They define the **order of execution**.

**2. Tasks**
A **task** is a unit of work in an Airflow DAG. Each task performs a specific function and can be an:
- **Operator**: Executes a specific action (e.g., run a Python script, execute SQL).
- **Sensor**: Waits for an external condition (e.g., file arrival, database update).

**3. Operators**
Operators define **what a task does**. Some common types include:
- `PythonOperator`: Runs a Python function.
- `BashOperator`: Executes a shell command.
- `SqlOperator`: Runs SQL queries.

**4. Sensors**
Sensors are **special operators that wait for an event** before proceeding. Examples:
- `FileSensor`: Waits for a file to appear.
- `ExternalTaskSensor`: Waits for another Airflow task to complete.

**How These Concepts Connect**
- **Operators & Sensors** define **Tasks**.
- **Tasks** form **DAGs** by establishing dependencies.

This modular approach allows **dynamic, scheduled, and automated workflows** in **Apache Airflow**.

### Airflow Architecture

![fig3 - architecture](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/2.%20Apache%20Airflow%20Workflow%20Orchestration/img/fig3%20-%20architecture.png)


| **Component**         | **Description** | **How It Works** |
|----------------------:|----------------|------------------|
| **Webserver (UI)**   | A web-based UI for monitoring and managing DAGs. | Users interact with the UI to trigger DAGs, check logs, and monitor task execution. It fetches data from the Metadata Database. |
| **Scheduler**        | Determines task execution timing. | Periodically scans the DAGs in the **DAG Directory**, schedules tasks, and updates their status in the **Metadata Database**. |
| **Executor**         | Executes tasks defined in the DAGs. | Works with **Workers** to run tasks in parallel. Executors can be **Sequential**, **Local**, **Celery**, or **Kubernetes**. |
| **Workers**          | Perform actual task execution. | Receive tasks from the **Executor**, run them, and update the **Metadata Database** with the results. |
| **DAG Directory**    | Stores Python scripts defining DAGs. | The **Scheduler** reads DAG definitions from this directory to determine execution schedules. |
| **Metadata Database** | Stores execution history, DAG runs, and task states. | Airflow components read from and write to the database to track task execution, failures, and retries. |

This architecture ensures **scalability, reliability, and efficient workflow automation**.

### Example Pipelines
#### SQL, Kubernets, Slack

![fig4 - sql,kubernetes,slack](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/2.%20Apache%20Airflow%20Workflow%20Orchestration/img/fig4%20-%20sql%2Ckubernetes%2Cslack.png)

| **Component**        | **Role in the Pipeline** |
|---------------------|------------------------|
| **SQL Sensor**     | Monitors an SQL Database for specific conditions (e.g., new data arrival, query completion). |
| **Kubernetes Operator** | Executes tasks within a **Kubernetes** cluster (e.g., running a containerized workload for processing data). |
| **Slack Operator** | Sends notifications to a **Slack** channel (e.g., alerts on task completion or failure). |

**Workflow Execution**
1. **SQL Sensor** waits for an event (e.g., new data in the SQL database).
2. Once the condition is met, the **Kubernetes Operator** triggers a task within Kubernetes (e.g., processing the new data).
3. After the Kubernetes job completes, the **Slack Operator** sends a notification (e.g., success or failure message) to Slack.

This **event-driven workflow** automates tasks across multiple platforms, ensuring efficient orchestration of data pipelines.

#### S3, GCP Dataproc, BigQuery

![fig5 - s3,gcp dataproc,bigquery](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/2.%20Apache%20Airflow%20Workflow%20Orchestration/img/fig5%20-%20s3%2C%20gcp%20dataproc%2C%20bigquery.png)

| **Component**        | **Role in the Pipeline** |
|---------------------|------------------------|
| **AWS S3 Sensor**  | Monitors an **Amazon S3** bucket for new data. |
| **Dataproc Operator** | Triggers a **Google Cloud Dataproc** job to process data (e.g., Spark or Hadoop tasks). |
| **Filestore** | Stores intermediate results from **Dataproc** before loading into BigQuery. |
| **BigQuery Operator** | Loads processed data from **Filestore** into **Google BigQuery** for analysis. |

**Workflow Execution**
1. **AWS S3 Sensor** detects new data in **Amazon S3**.
2. The **Dataproc Operator** starts a processing job on **Google Cloud Dataproc**.
3. Processed data is stored in **Filestore** for staging.
4. The **BigQuery Operator** loads the data from **Filestore** into **BigQuery** for querying and analytics.

This pipeline enables **automated, cloud-based data processing** across multiple platforms.

### Spotlight 3rd Party Operators

Apache Airflow's extensibility is enhanced through **third-party operators**, which are part of **provider packages**. These operators enable seamless integration with various external systems and services, expanding Airflow's capabilities beyond its core functionalities.

**Provider Packages**

Provider packages include integrations with third-party projects and are versioned and released independently of the Apache Airflow core. They encompass a range of components such as operators, hooks, sensors, and transfer operators to communicate with external systems. :contentReference[oaicite:0]{index=0}

For a comprehensive list of available provider packages and their functionalities, you can refer to the [Providers packages reference](https://airflow.apache.org/docs/apache-airflow-providers/packages-ref.html).

To explore the official documentation and learn more about integrating third-party operators into your workflows, visit the [Apache Airflow Documentation](https://airflow.apache.org/docs/).

By leveraging these third-party operators, you can enhance your Airflow workflows to interact with a wide array of external systems, thereby creating more dynamic and robust data pipelines.

### Airflow XComs

![fig6 - xcoms](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/2.%20Apache%20Airflow%20Workflow%20Orchestration/img/fig6%20-%20xcoms.png)

XCom (**Cross-Communication**) is a mechanism in **Apache Airflow** that enables tasks to share information asynchronously. Tasks can **push (write)** and **pull (read)** data using XComs, which are stored in the **Metadata Database**.

| **Component**          | **Description** |
|------------------------|----------------|
| **Task1**             | Generates and **writes** data to XCom. |
| **Metadata Database** | Stores the XCom data written by **Task1**. |
| **Task2.1 & Task2.2** | Retrieve (read) the stored XCom data from the **Metadata Database** to use in execution. |

**How XCom Works**
1. **Task1 writes an XCom** (`task_instance.xcom_push()`), storing data in the **Metadata Database**.
2. **Task2.1 and Task2.2 read the XCom** (`task_instance.xcom_pull()`) to retrieve and use the stored data.

*NOTE*: Airflow is not a data processing framework. It is designed for **workflow orchestration**, not for directly processing large-scale data.

## Hands-On Setup
### Setup

![fig7 - setup](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/2.%20Apache%20Airflow%20Workflow%20Orchestration/img/fig7%20-%20setup.png)

This **Airflow DAG** automates weather data collection and storage, making it useful for **data analytics, forecasting, and monitoring weather trends**.

**Execution Flow**
1. The **Extract** task makes an API request to the **Weather API** to fetch weather data.
2. The **Transform** task processes and formats the extracted data (e.g., removing null values, structuring JSON).
3. The **Load** task inserts the transformed data into **PostgreSQL** for storage and further analysis.

### Docker Setup

![fig8 - docker](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/2.%20Apache%20Airflow%20Workflow%20Orchestration/img/fig8%20-%20docker%20setup.png)

**How the Dockerized Airflow Setup Works**
1. The **Webserver UI** allows users to manage DAGs.
2. The **Scheduler** scans the **DAG Directory (`/dags`)** and determines which tasks need execution.
3. The **Executor** delegates tasks to **Celery Workers** for parallel processing.
4. **Redis** acts as a **message broker**, queuing tasks for workers.
5. **Celery Workers** pull tasks from Redis and execute them.
6. The **PostgreSQL Metadata Database** stores the task execution results.
7. The **Adminer UI** provides a simple way to interact with the PostgreSQL database.

**Why Use Docker for Airflow?**
- **Easier Deployment**: All components run in separate containers.
- **Scalability**: Celery Workers can be scaled up/down dynamically.
- **Persistence**: The PostgreSQL database ensures task history and logs are retained.

### Docker Compose and Starting Containers
- Change directory to project folder and run `docker compose up` in powershell

### Checking Services
- Go to [localhost:8080](localhost:8080) (Airflow UI) on browser and login

![fig9 - localhost](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/2.%20Apache%20Airflow%20Workflow%20Orchestration/img/fig9%20-%20localhost.png)

- We will use the ETL DAGs later on
- Go to [localhost:9000](localhost:9000) (Portainer) on browser and login

![fig10 - portainer](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/2.%20Apache%20Airflow%20Workflow%20Orchestration/img/fig10%20-%20portainer.png)

### Setup WeatherAPI
- Login to [WeatherApi](weatherapi.com) and copy API Key to put in scripts

### Setup PostgreSQL
- Go to [localhost:8081](localhost:8081) on browser
- Choose PostgreSQL and rename server to postgres:5432
- After logging in, create the WeatherData database and temperature table

![fig11 - postgresql](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/2.%20Apache%20Airflow%20Workflow%20Orchestration/img/fig11%20-%20postgresql.png)

# Airflow Weather ETL Pipeline
## Project Overview
This project demonstrates an **ETL pipeline using Apache Airflow**, designed to extract weather data from an API, transform it, and load it into PostgreSQL for storage and analysis. The pipeline is orchestrated using Airflow running in Docker with a CeleryExecutor.

## Folder Structure
```
Airflow/
│── docker-compose.yml
│── dags/
│   ├── 00_ETLWeatherPrintAirflow2.py
│   ├── 01-ETLWeatherPrint.py
│   ├── 02-ETLWeatherPostgres.py
│   ├── 03-ETLWeatherPostgresAndPrint.py
│   ├── SimpleHTTPOperator.py
│   ├── transformer.py
```

## Setting Up the Environment

### Start Airflow Using Docker-Compose
Ensure the correct `.env` file (if needed), and then run:
```sh
docker-compose up -d
```
This will start the Airflow webserver, scheduler, and worker processes along with PostgreSQL and Redis.

### Access Airflow Web UI
Once all services are running, open Airflow's UI at:
```
http://localhost:8080
```
Login credentials (default):
```
Username: airflow
Password: airflow
```

## DAGs in This Project
### 1. **00_ETLWeatherPrintAirflow2.py**
- [00_ETLWeatherPrintAirflow2.py](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/2.%20Apache%20Airflow%20Workflow%20Orchestration/dags/00_ETLWeatherPrintAirflow2.py)
- Extracts weather data from an API
- Transforms the data using a custom transformer
- Loads the data by logging it in Airflow

### 2. **01-ETLWeatherPrint.py**
- [01-ETLWeatherPrint.py](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/2.%20Apache%20Airflow%20Workflow%20Orchestration/dags/01-ETLWeatherPrint.py)
- Implements Airflow's **TaskFlow API** for better readability
- Uses Python decorators for defining tasks
- Logs the extracted and transformed data

### 3. **02-ETLWeatherPostgres.py**
- [02-ETLWeatherPostgres.py](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/2.%20Apache%20Airflow%20Workflow%20Orchestration/dags/02-ETLWeatherPostgres.py)
- Extracts weather data from an API
- Transforms it using `transform_weatherAPI`
- Loads the data into a **PostgreSQL** database

### 4. **03-ETLWeatherPostgresAndPrint.py**
- [03-ETLWeatherPostgresAndPrint.py](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/2.%20Apache%20Airflow%20Workflow%20Orchestration/dags/03-ETLWeatherPostgresAndPrint.py)
- Similar to the previous DAG but also **prints the transformed data** for verification

### 5. **SimpleHTTPOperator.py**
- [SimpleHTTPOperator.py](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/2.%20Apache%20Airflow%20Workflow%20Orchestration/dags/SimpleHTTPOperator.py)
- Uses `SimpleHttpOperator` to fetch API data directly
- Simplifies API calls within Airflow

## Transformer Module
- [`transformer.py`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/2.%20Apache%20Airflow%20Workflow%20Orchestration/dags/transformer.py)
- This script processes the API data and extracts key fields like location, temperature, wind speed, and timestamps.
- Uses Pandas for JSON normalization.

## DAG Flowchart
Below is a flowchart representation of the DAG workflow:

![fig12 - airflow etl graph](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/2.%20Apache%20Airflow%20Workflow%20Orchestration/img/fig12%20-%20airflow%20etl%20graph.PNG)

## Database Schema (PostgreSQL)
The data is stored in a PostgreSQL table named **temperature** with the following schema:
```sql
CREATE TABLE temperature (
    location TEXT,
    temp_c FLOAT,
    wind_kph FLOAT,
    time TIMESTAMP
);
```

## Airflow Connection Configuration
To connect Airflow to PostgreSQL, use the following configuration:

![fig13 - postgres connection](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/2.%20Apache%20Airflow%20Workflow%20Orchestration/img/fig13%20-%20postgres%20connection.PNG)

## Troubleshooting
### Permission Issues with Airflow Logs
If you encounter permission errors related to volume mounting in Docker, check the logs and ensure the necessary directories exist and have the correct permissions.

![fig14 - problem volumes](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/2.%20Apache%20Airflow%20Workflow%20Orchestration/img/fig14%20-%20problem%20volumes.PNG)

## Running a DAG
To trigger a DAG manually:
1. Open Airflow UI (`http://localhost:8080`)
2. Find the DAG from the list
3. Click the **Trigger DAG** button

## Stopping Airflow
To stop all services:
```sh
docker-compose down
```
