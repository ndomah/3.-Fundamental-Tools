# Building APIs with FastAPI
## API Fundamentals
### What are APIs and what are they used for?
An **API** is a set of rules that allows different software applications to communicate with each other. It defines how requests and responses should be structured so that systems can exchange data and functionality efficiently.

**Use Cases of APIs in Data Engineering**
1.	**Data Extraction** – APIs pull data from external sources (e.g., REST APIs for weather data, financial market data).
2.	**Automation & Integration** – Connects different systems (e.g., ETL pipelines fetching data from APIs).
3.	**Cloud Services** – Interfaces with cloud storage, databases, and compute resources (e.g., AWS, GCP, Azure APIs).
4.	**Microservices** – Enables modular, scalable architectures where different services communicate via APIs.
5.	**Real-Time Data Processing** – APIs stream real-time data (e.g., Kafka REST API, WebSockets).

### Hosting an API

![fig1 - hosting an api]()

This image illustrates an **event-driven architecture** where a client interacts with an API, which then routes data to different backend components like a **database** and a **message queue.**

- **Develop the API**
  - Use a framework like **Flask (Python), FastAPI, Express.js (Node.js), or Spring Boot (Java)** to build the API.
  - Define **endpoints** to handle client requests (e.g., POST /data).
- **Deploy the API**
  - Choose a **hosting platform:**
    - Cloud providers: **AWS (Lambda, EC2, API Gateway), GCP (Cloud Run, App Engine), Azure (Functions, App Services)**
    - Containerized deployment: **Docker + Kubernetes**
  - Set up **domain & networking** (e.g., Nginx as a reverse proxy).
- **Connect to a Database**
  - Store client data in a **database** (SQL: PostgreSQL, MySQL; NoSQL: MongoDB, DynamoDB).
  - Use an ORM (**SQLAlchemy, Prisma**) or direct queries.
- **Integrate a Message Queue**
  - For event-driven processing, connect to a **message queue** like Kafka, RabbitMQ, or AWS SQS.
  - This ensures asynchronous processing, scalability, and fault tolerance.
- **Ensure Security & Scalability**
  - **Authentication & Authorization:** Use **JWT, OAuth, API keys.**
  - **Load Balancing & Caching:** Implement **Redis, CDN, or API Gateway** for performance.
  - **Monitoring & Logging:** Use **Prometheus, Grafana, ELK Stack** to track performance.

### Using an API

![fig2 - using an api]()

This diagram represents an **API-based batch processing pipeline** where a **Python job** queries an **API**, processes the data, and routes it to different destinations.

- **Querying the API**
  - The Python job acts as a client, making periodic requests to an API for data.
  - Uses **HTTP requests (GET, POST, etc.)** to fetch data.
- **Scheduling the Python Job**
  - The job is scheduled to run at **regular intervals** using:
    - **Cron jobs** (Linux-based scheduling)
    - **Apache Airflow** (for workflow orchestration)
    - **Cloud Scheduler** (AWS Lambda, GCP Cloud Functions)
- **Processing & Storing the Data**
  - Once fetched, data is **processed and routed** to:
    - **Database** (structured storage for analytics)
    - **Message Queue** (Kafka, RabbitMQ) for event-driven processing
    - **Data Lake** (S3, HDFS, or GCS) for batch processing & large-scale storage
- **End-to-End Workflow**
  - API is queried → Data is retrieved → Python job processes it → Sends it to storage destinations.
 
### API Methods (HTTP Verbs)
- `GET`: Retrieve data from the server (e.g., get user details).
- `POST`: Send new data to the server (e.g., submit a form).
- `PUT`: Update existing data (replace the resource).
- `PATCH`: Partially update existing data (modify part of the resource).
- `DELETE`: Remove data from the server.

### API Media Types
These specify the format of the data being exchanged

- `JSON (application/json)`: Most common; lightweight and easy for humans and machines.
- `XML (application/xml)`: More verbose, often used in legacy systems.
- `HTML (text/html)`: When APIs return web pages.
- `Plain Text (text/plain)`: Simple, unstructured data.
- `Form Data (application/x-www-form-urlencoded or multipart/form-data)`: Used in web forms, especially for file uploads.

### HTTP Response Codes
These codes indicate the result of an API request.

`1xx - Informational`
- `100 Continue`: Server received the request, continue sending.
- `101 Switching Protocols`: Server switching protocols as requested.
  
`2xx - Success`
- `200 OK`: Request was successful.
- `201 Created`: Resource successfully created.
- `204 No Content`: Request succeeded, but no response body.

`3xx - Redirection`
- `301 Moved Permanently`: Resource has a new URL.
- `302 Found (Temporary Redirect)`: Resource temporarily moved.
- `304 Not Modified`: Cached version should be used.

`4xx - Client Errors`
- `400 Bad Request`: Invalid request from the client.
- `401 Unauthorized`: Authentication required.
- `403 Forbidden`: Client doesn’t have permission.
- `404 Not Found`: Requested resource does not exist.
- `405 Method Not Allowed`: HTTP method not supported.
- `429 Too Many Requests`: Rate limiting applied.

`5xx - Server Errors`
- `500 Internal Server Error`: General server failure.
- `502 Bad Gateway`: Server received an invalid response from an upstream server.
- `503 Service Unavailable`: Server is overloaded or down.
- `504 Gateway Timeout`: Server took too long to respond.

## Data & Environment Setup
### Setup Environment with WSL2, VS Code, & FastAPI
- WSL2: [https://learn.microsoft.com/en-us/windows/wsl/install](https://learn.microsoft.com/en-us/windows/wsl/install)
- VS Code: [https://code.visualstudio.com/download](https://code.visualstudio.com/download)

**Important Command Line Comands**:
```bash
sudo apt update && upgrade

sudo apt install python3 python3-pip ipython3

pip install fastapi

pip install uvicorn[standard]
```

### Test FastAPI
From the [FastAPI Tutorial](https://fastapi.tiangolo.com/tutorial/first-steps/):
- In [`main.py`]():
  ```python
  from fastapi import FastAPI

  app = FastAPI()


  @app.get("/")
  async def root():
    return {"message": "Hello World"}
  ```
- Run the live server:
  ```bash
  fastapi dev main.py
  ```






