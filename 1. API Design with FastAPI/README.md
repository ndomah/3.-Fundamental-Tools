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

pip install fastapi[standard]

pip install uvicorn[standard]
```

### Test FastAPI
From the [FastAPI Tutorial](https://fastapi.tiangolo.com/tutorial/first-steps/):
- In `main.py`:
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
- Open browser at [http://127.0.0.1:8000/](http://127.0.0.1:8000/)
- JSON response will be:
  ```json
  {"message": "Hello World"}
  ```
- Interactive API docs at [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)
- Alternative API docs at [http://127.0.0.1:8000/redoc](http://127.0.0.1:8000/redoc)

### The Dataset We Use
- We will use [UK E-Commerce Data](https://archive.ics.uci.edu/dataset/352/online+retail) from UCI's Machine Learning Repository (alternative [Kaggle link](https://www.kaggle.com/datasets/carrie1/ecommerce-data))

![fig3 - dataset]()

### API Design
- `Customer`
  - Create new customer
  - Get customer by ID
  - Create invoice for customer
  - Get all invoices for customer
- `Invoice`
  - Get specific invoice details (plus stock codes)
  - Invoice add stock code
- `StockCode`
  - Get specific stock code details
 
## Hands-On
- Open [`main.py`]() in VS Code, and run `fastapi dev main.py` in the terminal:
```python
# You need this to use FastAPI, work with statuses and be able to end HTTPExceptions
from fastapi import FastAPI, status, HTTPException

# Both used for BaseModel
from pydantic import BaseModel
from typing import Optional

# You need this to be able to turn classes into JSONs and return
from fastapi.encoders import jsonable_encoder
from fastapi.responses import JSONResponse

# import json
# from os import times


class Customer(BaseModel):
    customer_id: str
    country: str
    # city: Optional[str] = None


class URLLink(BaseModel):
    #url: str
    url: Optional[str] = None
    

class Invoice(BaseModel):
    invoice_no: int
    invoice_date: str
    customer: Optional[URLLink] = None

fakeInvoiceTable = dict()

# this is important for general execution and the docker later
app = FastAPI()

# Base URL
@app.get("/")
async def root():
    return {"message": "Hello World"}

# Add a new Customer
@app.post("/customer")
async def create_customer(item: Customer): #body awaits a json with customer information
# This is how to work with and return a item
#   country = item.country
#   return {item.country}

    # You would add here the code for creating a Customer in the database

    # Encode the created customer item if successful into a JSON and return it to the client with 201
    json_compatible_item_data = jsonable_encoder(item)
    return JSONResponse(content=json_compatible_item_data, status_code=201)


# Get a customer by customer id
@app.get("/customer/{customer_id}") # Customer ID will be a path parameter
async def read_customer(customer_id: str):
    
    # only succeed if the item is 12345
    if customer_id == "12345" :
        
        # Create a fake customer ( usually you would get this from a database)
        item = Customer(customer_id = "12345", country= "Germany")  
        
        # Encode the customer into JSON and send it back
        json_compatible_item_data = jsonable_encoder(item)
        return JSONResponse(content=json_compatible_item_data)
    else:
        # Raise a 404 exception
        raise HTTPException(status_code=404, detail="Item not found")


# Create a new invoice for a customer
@app.post("/customer/{customer_id}/invoice")
async def create_invoice(customer_id: str, invoice: Invoice):
    
    # Add the customer link to the invoice
    invoice.customer.url = "/customer/" + customer_id
    
    # Turn the invoice instance into a JSON string and store it
    jsonInvoice = jsonable_encoder(invoice)
    fakeInvoiceTable[invoice.invoice_no] = jsonInvoice

    # Read it from the store and return the stored item
    ex_invoice = fakeInvoiceTable[invoice.invoice_no]
    
    return JSONResponse(content=ex_invoice) 



# Return all invoices for a customer
@app.get("/customer/{customer_id}/invoice")
async def get_invoices(customer_id: str):
    
    # Create Links to the actual invoice (get from DB)
    ex_json = { "id_123456" : "/invoice/123456",
                "id_789101" : "/invoice/789101" 
    }
    return JSONResponse(content=ex_json) 



# Return a specific invoice
@app.get("/invoice/{invnoice_no}")
async def read_invoice(invnoice_no: int):
    # Option to manually create an invoice
        #ex_inv = Invoice(invoice_no = invnoice_no, invoice_date= "2021-01-05", customer= URLLink(url = "/customer/12345"))
        #json_compatible_item_data = jsonable_encoder(ex_inv)
    
    # Read invoice from the dictionary
    ex_invoice = fakeInvoiceTable[invnoice_no]

    # Return the JSON that we stored
    return JSONResponse(content=ex_invoice)


#get a specific stock code on the invoice
@app.get("/invoice/{invnoice_no}/{stockcode}/")
async def read_item(invnoice_no: int,stockcode: str):
    return {"message": "Hello World"}

# Add a stockcode to the inovice
@app.post("/invoice/{invnoice_no}/{stockcode}/")
async def add_item(invnoice_no: int ,stockcode:str):
    return {"message": "Hello World"}
```
- Open server at [http://127.0.0.1:8000/](http://127.0.0.1:8000/) and documentation at [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)

**API Endpoints**
- `POST /customer`
  - Creates a new customer 
  - Request body:
    ```json
    {
      "customer_id": "12345",
      "country": "Germany"
    }
    ```
  - Response (201 Created):
    ```json
    {
      "customer_id": "12345",
      "country": "Germany"
    }
    ```

![fig4 - post customer]()


- `GET /customer/{customer_id}`: Read customer
  - Retrieves customer details based on `customer_id`
  - Path parameter: `{customer_id}` (e.g., `12345`)
  - Example request: `GET /customer/12345`
  - Response:
    ```json
    {
      "customer_id": "12345",
      "country": "Germany"
    }
    ```
  - If the customer does not exist:
    ```json
    {
    "detail": "Item not found"
    }
    ```

![fig5 - get customer]()


- `POST /customer/{customer_id}/invoice`
  - Creates a new invoice for a given customer
  - Path parameter: `{customer_id}`
  - Request body:
    ```json
    {
      "invoice_no": 123456,
      "invoice_date": "2021-01-05"
    }
    ```
  - Response:
    ```json
    {
      "invoice_no": 123456,
      "invoice_date": "2021-01-05",
      "customer": {
        "url": "/customer/12345"
      }
    }
    ```

![fig6 - post invoice]()


- `GET /invoice/{invoice_no}`
  -  Retrieves a specific invoice by its number
  -  Path parameter: `{invoice_no}`
  -  Example request: `GET /invoice/123456`
  -  Response (if invoice exists):
     ```json
     {
       "invoice_no": 123456,
       "invoice_date": "2021-01-05",
       "customer": {
        "url": "/customer/12345"
      }
     }
     ```

![fig7 - get invoice]()


- `GET /customer/{customer_id}/invoice`
  - Returns a list of invoices for a given customer
  - Path parameter: `{customer_id}`
  - Example request: `GET /customer/12345/invoice`
  - Response:
    ```json
    {
      "id_123456": "/invoice/123456",
      "id_789101": "/invoice/789101"
    }
    ```  

![fig 8 - get customer invoice]()

## Deploying and Testing FastAPI with Docker and Postman
### Setup Docker and Deploy on WSL2
- On VS Code, build image from [Dockerfile.yaml]():
  ```yaml
  FROM tiangolo/uvicorn-gunicorn-fastapi:python3.7

  COPY ./app /app
  ```
- Name image as `apiswithfastapi:latest`
- In the docker extension of the VS Code sidebar navigate to:
  - IMAGES -> apiswithfastapi -> latest (run this)
- This will show apiswithfastapi:latest as running under CONTAINERS
- You will see the interactive docs again at [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)

### Testing the APIs with Postman
- Download [Postman](https://www.postman.com/downloads/) and import [`FASTApiTest.postman_collection.json`]()
- Test by sending localhost/customer/12345 under `GET Customer`:

![fig9 - postman]()



