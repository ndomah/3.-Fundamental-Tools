# Snowflake for Data Engineers
## Choosing Data Stores
### Snowflake Basics

![fig1 - snowflake basics](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/img/fig1%20-%20snowflake%20basics.png)

**ETL Workflow**:
- **Extract**: Data is sourced from files/cloud storage
- **Transform & Load**: Data is staged, processed (tasks, pipes), and stored in tables
- **Consume**: Processed data is used by BI tools, Python applications, and Databricks

| **Step**         | **Component**      | **Description** |
|------------------|-------------------|----------------|
| **Data Sources** | Local File Source | Data from local storage (e.g., CSV, JSON). |
|                 | Cloud Storage      | Data from cloud storage (S3, Azure Blob, GCS). |
| **Staging**      | Stage              | Temporary storage inside Snowflake before loading into tables. |
| **Processing**   | Worksheet          | SQL interface for executing queries manually. |
|                 | Task               | Automates SQL execution (e.g., scheduled ETL jobs). |
|                 | Pipe               | Continuous data ingestion (e.g., Snowpipe for streaming data). |
| **Storage**      | Table              | Processed and structured data stored in Snowflake tables. |
| **Consumption**  | BI Tool (Power BI) | Business Intelligence tool for visualization. |
|                 | Python Client      | Access Snowflake data for analytics, ML, or automation. |
|                 | Databricks         | Advanced analytics and data processing. |


### Data Warehousing Basics

![fig2 - b,s,g tables](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/img/fig2%20-%20bsg%20table.png)


| **Layer**       | **Purpose**                                      | **Characteristics**                                         | **Example Data**                        |
|----------------|--------------------------------------------------|------------------------------------------------------------|-----------------------------------------|
| **Bronze**     | Raw data ingestion                               | - Unprocessed, raw data from sources                       | Logs, sensor data, raw CSV files       |
|                | Historical storage                               | - May contain duplicates, missing values, or inconsistencies | Unstructured JSON, API data            |
| **Silver**     | Cleaned and structured data                      | - Duplicates removed, schema validated, transformed        | Standardized customer transactions     |
|                | Enrichment and normalization                     | - May include metadata, deduplicated, lightly aggregated   | Sales transactions with correct formats |
| **Gold**       | Analytics-ready data for business use            | - Aggregated, modeled, and optimized for queries          | Sales reports by region, customer KPIs |
|                | Used by BI tools and reporting                   | - Designed for performance and final consumption          | Dashboard-ready financial summaries    |

### How Snowflake fits into Data Platforms

![fig3 - oltp, olap](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/img/fig3%20-%20oltp%2C%20olap.png)

## Loading CSVs from your PC
### Our Dataset & Goals
- We are using [Online Retail](https://archive.ics.uci.edu/dataset/352/online+retail) data:

![fig4 - dataset](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/img/fig4%20-%20dataset.png)

**Staging data from local drive**

![fig5 - stage data](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/img/fig5%20-%20stage%20data.png)

*NOTE*: We are skipping silver tables for this tutorial... DON'T DO IN PROD

### Setup Snowflake Database
- Run [`1_0_before_loading_csv.sql`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/worksheets/1_0_before_loading_csv.sql) in Snowflake Worksheet:
```sql
CREATE OR REPLACE  WAREHOUSE SMALLWAREHOUSE
WAREHOUSE_SIZE = 'XSMALL';

CREATE OR REPLACE DATABASE TESTDB;

CREATE OR REPLACE SCHEMA TESTDB.ECOMMERCE;

-- 1 CREATE FORMAT
-- REF https://docs.snowflake.com/en/sql-reference/sql/create-file-format.html
-- REF https://docs.snowflake.com/en/sql-reference/sql/alter-file-format.html
-- follows: "DATABASENAME"."SCHEMANAME".FORMATNAME 
CREATE FILE FORMAT "TESTDB"."ECOMMERCE".ECOMMERCECSVFORMAT 
-- add SET if ALTER (insted io create)
COMPRESSION = 'AUTO' 
FIELD_DELIMITER = ',' 
RECORD_DELIMITER = '\n' 
SKIP_HEADER = 1 
FIELD_OPTIONALLY_ENCLOSED_BY = 'NONE' 
TRIM_SPACE = FALSE 
TIMESTAMP_FORMAT = 'MM/DD/YYYY HH:MI'
ERROR_ON_COLUMN_COUNT_MISMATCH = TRUE 
ESCAPE = 'NONE' 
ESCAPE_UNENCLOSED_FIELD = '\134'
NULL_IF = ('\\N');

-- "DATABASENAME"."SCHEMANAME".DATANAME
-- 2. CREATE DATA
create or replace TABLE TESTDB.ECOMMERCE.DATA (
	INVOICENO VARCHAR(38),
	STOCKCODE VARCHAR(38),
	DESCRIPTION VARCHAR(60),
	QUANTITY NUMBER(38,0),
	INVOICEDATE TIMESTAMP,
	UNITPRICE NUMBER(38,0),
	CUSTOMERID VARCHAR(10),
	COUNTRY VARCHAR(20)
);
```
### Preparing the Upload File
- Run [`simplify.py`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/simplify.py):
```python
"""
Remove lines that give problems to snowflake.
Tested with python 3.8.5 and pandas 1.4.0
"""
from zipfile import ZipFile
import numpy as np
import pandas as pd


def row_to_skip(row: pd.Series) -> bool:
    descr = row['Description']
    if pd.isnull(descr):
        return False
    if ',' in descr:
        return True
    return False


if __name__ == "__main__":
    in_fp = 'archive.zip'
    out_fp = 'upload.csv'
    dtypes = { "InvoiceNo": str,
            "StockCode": str,
            'Description': str,
            'Quantity': np.int32,
            'UnitPrice': np.float64,
            'CustomerID': str,
            'Country': str
    }
    stream = ZipFile(in_fp).open('data.csv', 'r')
    df = pd.read_csv(stream, encoding_errors='replace', dtype=dtypes)
    drop_rows = df.apply(func=row_to_skip, axis=1)
 
    print(drop_rows.head())

    # tilda because we don't want to select any of these rows
    df = df.loc[~drop_rows, :]
    
    """
    if you want to upload df using the GUI console,
    consider calling something df.head(2000) or dropping at least around 50% of the rows
    """
    df.to_csv(out_fp, index=False)
    print(f'File to upload available at {out_fp}')
```
- This will create the `upload.csv` file

### Using Internal Stages with SnowSQL
- Start [SnowSQL](https://www.snowflake.com/en/developers/downloads/snowsql/) in PowerShell: `snowsql -a bl46735.us-east-2.aws -u NILESHD -w SMALLWAREHOUSE -d TESTDB`
- Run commands from [`1_1 Load with snowsql.sql`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/worksheets/1_1%20Load%20with%20snowsql.sql):
```sql
-- set the warehouse manually
USE WAREHOUSE SMALLWAREHOUSE;

-- set database manually
USE DATABASE TESTDB;

-- select the schema
use schema ECOMMERCE

-- create stage use the file format
create stage my_upload 
    file_format = ECOMMERCECSVFORMAT;

-- stage file
put file://\opt/snowflake/upload.csv @my_upload;

-- describe the stage to check parameters
DESCRIBE STAGE my_upload;

-- validate before copy with 2 rows
copy into DATA from @my_upload validation_mode = 'RETURN_2_ROWS';

--copy staged file into table
copy into ECOMMERCE.DATA from @my_upload on_error = CONTINUE;

-- remove staged files, because copy always copies everything
remove @my_upload 

-- see your table is populated now
SHOW TABLES;

---

-- Do this in case you don't have a format specified
-- create stage
create stage my_upload FILE_Format = (TYPE = CSV skip_header = 1);

-- alter timestamp format
alter session set timestamp_input_format='MM/DD/YYYY HH24:MI';
```
- The data will now be uploaded in the table

![fig6 - data preview](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/img/fig6%20-%20data%20preview.png)

### Splitting a Data Table into 2 Tables

- Run [`2_split_table.sql`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/worksheets/2_split_table.sql) in Snowflake Worksheet:
```sql
CREATE OR REPLACE TABLE TESTDB.ECOMMERCE.INVOICES AS( SELECT DISTINCT CUSTOMERID, COUNTRY, INVOICEDATE, INVOICENO
               FROM TESTDB.ECOMMERCE.DATA
              );

CREATE OR REPLACE TABLE TESTDB.ECOMMERCE.ITEMS AS ( SELECT STOCKCODE, DESCRIPTION, UNITPRICE,QUANTITY, INVOICENO
               FROM TESTDB.ECOMMERCE.DATA
              );

-- expected n rows 25905
SELECT COUNT(*) FROM TESTDB.ECOMMERCE.INVOICES;

-- expected n rows 537113
SELECT COUNT(*) FROM TESTDB.ECOMMERCE.ITEMS;
```
## Visualizing Data
### Creating a Visualization Worksheet
- From [`3_visualize.sql`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/worksheets/3_visualize.sql):
```sql
-- INVOICES TABLE
SELECT COUNT(DISTINCT COUNTRY) AS NUMBER_COUNTRIES FROM INVOICES;
```
![fig7](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/img/fig7.png)
```sql
-- TOP 2-10 countries with most clients
SELECT COUNTRY, 
       COUNT(DISTINCT CUSTOMERID) AS N_CLIENTS
   -- REMOVE UK AS IT HAS TOO MANY CLIENTS COMPARED TO OTHER COUNTRIES
FROM INVOICES
WHERE UPPER(COUNTRY) NOT LIKE 'UNITED%'
GROUP BY COUNTRY
ORDER BY N_CLIENTS DESC
LIMIT 10;
```
![fig8](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/img/fig8.png)
```sql
-- TOP clinets with most invoices
SELECT CUSTOMERID, COUNT(DISTINCT INVOICENO) AS N_ORDERS
FROM INVOICES
GROUP BY COUNTRY, CUSTOMERID
ORDER BY N_ORDERS DESC
LIMIT 10;
```
![fig9](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/img/fig9.png)
```sql
-- most ordered items
SELECT STOCKCODE,DESCRIPTION,SUM(QUANTITY) AS TOTAL_QUANTITY
FROM ITEMS
GROUP BY STOCKCODE, DESCRIPTION
ORDER BY TOTAL_QUANTITY DESC
LIMIT 10;
```
![fig10](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/img/fig10.png)
```sql
-- overview of unit prices
WITH TEMP AS (
    SELECT DESCRIPTION, UNITPRICE
    FROM ITEMS
    GROUP BY STOCKCODE, DESCRIPTION, UNITPRICE
    ORDER BY UNITPRICE DESC)
SELECT COUNT(*), 
       AVG(UNITPRICE),
       MIN(UNITPRICE),
       MAX(UNITPRICE)
FROM TEMP;
```
![fig11](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/img/fig11.png)
```sql
--  Which customers bought a WHILE METAL LANTERN?
SELECT DISTINCT INVOICES.CUSTOMERID
FROM ITEMS
JOIN INVOICES ON ITEMS.INVOICENO=INVOICES.INVOICENO
WHERE ITEMS.DESCRIPTION = 'WHITE METAL LANTERN' 
AND INVOICES.CUSTOMERID IS NOT NULL;
```
![fig12](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/img/fig12.png)
```sql
-- Which ITEMS are the most revenue generating per country outside of UK?
SELECT ITEMS.DESCRIPTION, AVG(ITEMS.UNITPRICE) * SUM(ITEMS.QUANTITY) AS TOTAL_REVENUE, INVOICES.COUNTRY
FROM ITEMS
JOIN INVOICES ON ITEMS.INVOICENO=INVOICES.INVOICENO
WHERE UPPER(INVOICES.COUNTRY) NOT LIKE 'UNITED%'
GROUP BY ITEMS.DESCRIPTION, INVOICES.COUNTRY
ORDER BY TOTAL_REVENUE DESC, INVOICES.COUNTRY, ITEMS.DESCRIPTION;
```
![fig13](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/img/fig13.png)

### Creating Dashboard
- From [`5_dashboard.sql`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/worksheets/5_dashboard.sql):
```sql
SELECT STOCKCODE,DESCRIPTION,SUM(QUANTITY) AS TOTAL_QUANTITY
FROM ITEMS
GROUP BY STOCKCODE, DESCRIPTION
ORDER BY TOTAL_QUANTITY DESC
LIMIT 10;
```

![fig14 - dashboard](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/img/fig14%20-%20dashboard.png)


### Connect PowerBI to Snowflake
- In PowerBI connect to Snowflake warehouse using DirectQuery

![fig15](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/img/fig15%20-%20powerbi.png)

### Query Data with Python
- Run [`connect.py`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/connect.py):
```python
"""
With this snippet you can connect to snowflake with python
More docs
https://docs.snowflake.com/en/user-guide/python-connector-pandas.html
In your .bashrc or .zshrc make sure you add the following variables
export user='MYUSERNAME'
export password='MYPASSWORD'
export account='hi68877'
Dependecies to run the script:
pip install "snowflake_connector_python[pandas]==2.8.2"
pip install jwt==1.3.1
To run it (after doing the other tasks):
python worksheets/connect.py
"""
import os
import pandas as pd
from snowflake import connector

print('starting connection')
snowflake_id = os.environ['SNOWFLAKE_ACCOUNT']
ctx = connector.connect(
    user=os.environ['SNOWFLAKE_USER'],
    password=os.environ['SNOWFLAKE_PASSWORD'],
    account=snowflake_id,
    warehouse='SMALLWAREHOUSE',
    database='TESTDB',
    schema='ECOMMERCE',
    autocommit=True
)
print('ok')

db_cursor_eb = ctx.cursor()
res = db_cursor_eb.execute("""
SELECT CUSTOMERID, COUNT(DISTINCT INVOICENO) AS N_ORDERS
FROM INVOICES
GROUP BY COUNTRY, CUSTOMERID
ORDER BY N_ORDERS DESC
LIMIT 10;
"""
)
# Fetches all records retrieved by the query and formats them in pandas DataFrame
df = res.fetch_pandas_all()
print(df)
```

![fig16](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/img/fig16.png)

## Automation
### Create Import Task
- From [`4_1_task_import.sql`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/worksheets/4_1_task_import.sql):
```sql
-- Clean our stage
list @my_upload;

remove @my_upload;

create or replace TABLE TESTDB.ECOMMERCE.DATA (
	INVOICENO VARCHAR(38),
	STOCKCODE VARCHAR(38),
	DESCRIPTION VARCHAR(60),
	QUANTITY NUMBER(38,0),
	INVOICEDATE TIMESTAMP,
	UNITPRICE NUMBER(38,0),
	CUSTOMERID VARCHAR(10),
	COUNTRY VARCHAR(20)
);

create or replace task TESTDB.ECOMMERCE.MY_import_from_stage
	warehouse=SMALLWAREHOUSE
	schedule='1 MINUTE'
	as copy into ECOMMERCE.DATA from @my_upload
              ;

-- create a dependent task on the first one
create or replace task TESTDB.ECOMMERCE.MY_clean_stage
	warehouse=SMALLWAREHOUSE
	after TESTDB.ECOMMERCE.MY_import_from_stage
	as remove @my_upload
              ;

-- RESUME to let it run / SUSPEND (default) to stop it
ALTER TASK TESTDB.ECOMMERCE.MY_clean_stage RESUME;
ALTER TASK TESTDB.ECOMMERCE.MY_import_from_stage RESUME;
```
- Rerun `PUT` command in Snowflake command line
  - The `TASK` will place the csv in the stage, upload it, and then remove it from the stage 


### Create Table Refresh Task
- From[`4_2_task_runner.sql`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/7.%20Snowflake%20for%20Data%20Engineers/worksheets/4_2_task_runner.sql):
```sql
create or replace task TESTDB.ECOMMERCE.SPLIT_TABLE_AUTOMATIC
	warehouse=SMALLWAREHOUSE
	schedule='1 MINUTE'
	as CREATE OR REPLACE TABLE TESTDB.ECOMMERCE.INVOICES AS( SELECT DISTINCT CUSTOMERID, COUNTRY, INVOICEDATE, INVOICENO
               FROM TESTDB.ECOMMERCE.DATA
              );

-- create a dependent task on the first one
create or replace task TESTDB.ECOMMERCE.SPLIT_TABLE_AUTOMATIC_SECOND
	warehouse=SMALLWAREHOUSE
	after TESTDB.ECOMMERCE.SPLIT_TABLE_AUTOMATIC
	as CREATE OR REPLACE TABLE TESTDB.ECOMMERCE.ITEMS AS ( SELECT STOCKCODE, DESCRIPTION, UNITPRICE,QUANTITY, INVOICENO
               FROM TESTDB.ECOMMERCE.DATA
              );
               
-- RESUME to let it run / SUSPEND (default) to stop it
ALTER TASK TESTDB.ECOMMERCE.SPLIT_TABLE_AUTOMATIC SUSPEND;
ALTER TASK TESTDB.ECOMMERCE.SPLIT_TABLE_AUTOMATIC_SECOND SUSPEND;
```
