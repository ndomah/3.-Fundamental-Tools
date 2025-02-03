# Log Analysis with Elasticsearch
## Fundamentals
### Elasticsearch vs MySQL

![fig1- es vs mysql](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/6.%20Log%20analysis%20with%20Elasticsearch/img/fig1%20-%20ev%20vs%20mysql.png)

- **MySQL**: A relational database (RDBMS) used for structured data storage with relationships between tables.
  ```mysql
  SELECT * FROM users WHERE name = 'Alice';
  ```
  
- **Elasticsearch**: A distributed search and analytics engine optimized for full-text search and real-time data analysis.
  ```json
  GET users/_search
  {
    "query": {
    "match": { "name": "Alice" }
    }
  }
  ```

**Use Cases**
|**Feature**|**MySQL**|**Elasticsearch**|
|---|---|---|
|Structured Data|✅ Yes|❌ No|
|Full-Text Search|❌ No|✅ Yes|
|Aggregations|Limited|✅ Fast & Real-Time|
|Relationships (Joins)|✅ Yes|❌ No (Denormalized)|
|Horizontal Scaling|❌ Difficult|✅ Built-in|
|Real-Time Analytics|❌ Slow|✅ Fast|


### ETL Log Analysis & Debugging Problems

![fig2 - etl problems](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/6.%20Log%20analysis%20with%20Elasticsearch/img/fig2%20-%20etl%20problems.png)

**All Data Is No Longer Processed**
- **Reason**: The data source has changed (e.g., API structure changes, schema updates, data format modifications).
- **Impact**: The ETL process may fail silently, meaning no one notices missing or incomplete data until much later.
- **Common Cause**: APIs and databases are frequently updated without ETL pipelines being adjusted accordingly.

**ETL is Dropping Data**
- **Reason**: Data loss can occur at any stage of ETL:
  - During **extraction** if the data source is unstable.
  - During **transformation** if incorrect mappings are applied.
  - During **loading** if storage constraints or conflicts occur.
- **Impact**: Data integrity is compromised, leading to incorrect analysis and business decisions.

**ETL Takes Too Long**
- **Which step is slow?**
  - **Extract**: If the data source is large or APIs have rate limits, extraction can become a bottleneck.
  - **Transform**: Complex joins, aggregations, and data cleaning operations can slow down the process.
  - **Load**: Writing large amounts of data to a database or data warehouse may be inefficient.
- **Impact**: Slow ETL pipelines delay analytics and reporting, reducing the value of real-time insights.

**Debugging Log Files is Difficult**
- **Why?**
  - Logs may be **incomplete, hard to parse, or too verbose**.
  - Errors may not clearly indicate **where** in the ETL process things failed.
  - Searching through logs manually is time-consuming and error-prone.
- **Impact**: Troubleshooting ETL failures becomes painful and inefficient, leading to long downtimes.

### Streaming Log Analysis & Debugging Problems

![fig3 - stream and batch](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/6.%20Log%20analysis%20with%20Elasticsearch/img/fig3%20-%20stream%20and%20batch.png)

![fig4 - solutions](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/6.%20Log%20analysis%20with%20Elasticsearch/img/fig4%20-%20solutions.png)

- **Elasticsearch collects logs from multiple sources** for centralized monitoring.
- **Kibana enables real-time visualization** of pipeline performance.
- **Engineers can proactively detect and fix ETL issues** (data loss, API failures, processing delays).
- **Better debugging & alerting** prevent silent failures in the ETL pipeline.

## Elasticsearch Hands-On
### ELK Stack Overview
[**Elastic Stack**](https://www.elastic.co/elastic-stack):
- *Elasticsearch*: A distributed, JSON-based search and analytics engine
- *Kibana*: Gives shape to your data and is the extensible user interface
- *Integrations*: Enable you to collect and connect your data with the Elastic Stack

### Elasticsearch Setup Limiting RAM & Environment Setup
**DockerHub**:
- [elasticsearch](https://hub.docker.com/_/elasticsearch)
- [kibana](https://hub.docker.com/_/kibana)

**Limiting Ram**
- Run the following in powershell:
  ```
  wsl --shutdown
  notepad "$env:USERPROFILE/.wslconfig"
  ```
- Within the `.wslconfig` file enter:
  ```
  [wsl2]
  memory=4GB   # Limits VM memory in WSL 2 up to 4GB
  ```
- Open the WSL command line and run: `sudo sysctl -w vm.max_map_count=262144`
- Add [`docker-compose.yml`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/6.%20Log%20analysis%20with%20Elasticsearch/scripts/docker-compose.yml) to project folder:
```yml
version: '3.7'

services:

  # Elasticsearch Docker Images: https://www.docker.elastic.co/
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.17.1
    container_name: elasticsearch
    environment:
      - xpack.security.enabled=false
      - discovery.type=single-node
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
    cap_add:
      - IPC_LOCK
    volumes:
      - elasticsearch-data17:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
      - 9300:9300

  kibana:
    container_name: kibana
    image: docker.elastic.co/kibana/kibana:7.17.1
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    ports:
      - 5601:5601
    depends_on:
      - elasticsearch

volumes:
  elasticsearch-data17:
    driver: local
```
- Run `docker compose up` in powershell (in project directory)
- In browser, go to [localhost:5601](localhost:5601) to view kibana dashboard:

![fig5 - kibana dashboard](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/6.%20Log%20analysis%20with%20Elasticsearch/img/fig5%20-%20kibana.png)

### Elasticsearch APIs & Creating an Index with Python
- Refer to [`01_Create_mapped_index.py`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/6.%20Log%20analysis%20with%20Elasticsearch/scripts/01_Create_mapped_index.py):
```python
from elasticsearch import Elasticsearch


es = Elasticsearch("http://localhost:9200")

request_body = {
	    "settings" : {
	        "number_of_shards": 1,
	        "number_of_replicas": 1
	    },
	    'mappings': {
	            'properties': {
	                'process': {'type': 'text'},
	                'date': {'format': 'yyyy-MM-dd', 'type': 'date'},
	                'processing_time': {'type': 'double'},
	                'records': {'type': 'integer'},
					'run': {'type': 'integer'},
					'msg': {'type': 'text'}
	            }}
	}

print("creating 'example_index' index...")
es.indices.create(index = 'etl_monitoring', body = request_body)
```
- In Kibana stack management > index management, check the `etl_monitoring` index
- Then index patterns > create index pattern, enter `etl**` into the name field, select the date timestamp field > create index pattern

![fig6 - kibana index pattern](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/6.%20Log%20analysis%20with%20Elasticsearch/img/fig6%20-%20kibana%20index%20pattern.png)

### Write JSON logs to Elasticsearch
- Refer to [`02_write_logs.py`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/6.%20Log%20analysis%20with%20Elasticsearch/scripts/02_write_logs.py):
```python
from elasticsearch import Elasticsearch

# initialize elasticsearch client
es = Elasticsearch("http://localhost:9200")

def index_first_run():
    # create logs for etl process run 1
    extract_json_1 = {
        'process': 'extract',
        'status' : 'success',
        'date': '2022-03-01',
        'processing_time': 5,
        'records': 5000,
        'run': 1
        }

    transform_json_1 = {
        'process': 'transform',
        'status' : 'success',
        'date': '2022-03-01',
        'processing_time': 5,
        'records': 5000,
        'run': 1
        }

    load_json_1 = {
        'process': 'load',
        'status' : 'success',
        'date': '2022-03-01',
        'processing_time': 5,
        'records': 5000,
        'run': 1
        }

    # write them to the index
    es.index(index = 'etl_monitoring',document = extract_json_1)
    es.index(index = 'etl_monitoring',document = transform_json_1)
    response = es.index(index = 'etl_monitoring',document = load_json_1)
    print(response)


# https://elasticsearch-py.readthedocs.io/en/latest/api.html?highlight=es.index#elasticsearch.Elasticsearch.index
# https://www.elastic.co/guide/en/elasticsearch/reference/7.16/docs-index_.html


# crate logs for etl run 2
def index_second_run():
    extract_json_2 = {
        'process': 'extract',
        'status' : 'success',
        'date': '2022-03-02',
        'processing_time': 10,
        'records': 10000,
        'run': 2
        }

    transform_json_2 = {
        'process': 'transform',
        'status' : 'success',
        'date': '2022-03-02',
        'processing_time': 10,
        'records': 10000,
        'run': 2
        }

    load_json_2 = {
        'process': 'load',
        'status' : 'success',
        'date': '2022-03-02',
        'processing_time': 10,
        'records': 9999,
        'run': 2
        }

    error_json_2 = {
        'process': 'load',
        'status' : 'error',
        'date': '2022-03-02',
        'processing_time': 5,
        'records': 1,
        'run': 2,
        'msg' : 'this is what happened'
        }

    # write them to the index
    es.index(index = 'etl_monitoring',document = extract_json_2)
    es.index(index = 'etl_monitoring',document = transform_json_2)
    es.index(index = 'etl_monitoring',document = load_json_2)
    response = es.index(index = 'etl_monitoring',document = error_json_2)
    print(response)

index_first_run()
index_second_run()

# write run 2 to the index
```
- Reload indices on Kibana
- View data: analytics > discover > set filter to account for date in json

![fig7 - discover](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/6.%20Log%20analysis%20with%20Elasticsearch/img/fig7%20-%20discover.png)

## Analyzing Logs with Kibana
### Create Kibana Visualization & Dashboards

![fig8 - dashboard](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/6.%20Log%20analysis%20with%20Elasticsearch/img/fig7%20-%20discover.png)


### Analyze Logs by Searching Elasticsearch
- We know there was an error in the load phase

![fig9 - errors](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/6.%20Log%20analysis%20with%20Elasticsearch/img/fig9%20-%20errors.png)
