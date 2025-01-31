# MongoDB Fundamentals
## MongoDB Basics
### Relational Schemas

![fig1 - relational schemas](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/5.%20MongoDB%20Fundamentals/img/fig1%20-%20relational%20schemas.png)

A **relational schema** is a blueprint that defines how data is structured in a relational database. It specifies the tables, their columns, and relationships between them using **primary keys (PK)** and **foreign keys (FK)**.

**Characteristics**:
- **Normalization**: Data is stored in separate tables to reduce redundancy.
- **Joins**: Data retrieval relies on joins (e.g., to get stock items in an invoice).
- **Constraints**: Ensures data integrity using **PKs** and **FKs**.

### MongoDB Documents Explained

![fig2 - mongodb doc](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/5.%20MongoDB%20Fundamentals/img/fig2%20-%20mongodb%20doc.png)

In MongoDB, data is stored in **documents**, which are JSON-like structures called **BSON (Binary JSON)**. Unlike relational databases that use tables, MongoDB documents are schema-flexible, allowing nested structures and arrays within a single record.

**Key Features**:
- **Embedded Documents**: Related data is stored together in a denormalized structure, reducing the need for joins.
- **Arrays of Sub-Documents**: Supports one-to-many relationships naturally within a document.
- **Flexible Schema**: Fields can vary across documents without strict table definitions.
- **Efficient Reads**: Querying a single document retrieves all related data without requiring multiple joins.

**Performance Optimization**
- **Indices**
  - Without an index, MongoDB **scans every document** in a collection to find a match (**full collection scan**).
  - With an index, MongoDB **uses the indexed field** to locate data faster.
  - Indices store a **sorted order** of the field values, making lookups significantly quicker.
- **Sharding**
  - When a **single server** cannot handle the workload, we **split** the data across **multiple servers**.
  - It allows **horizontal scaling**, unlike vertical scaling (adding more CPU/RAM to one server).

### Relational DBs vs MongoDB
|**Feature**|**Relational DB (SQL)**|**MongoDB (NoSQL)**|
|--:|---|---|
|**Data Storage**|Tables & rows|Documents (JSON-like)|
|**Relationships**|Joins across tables|Embedded documents & arrays|
|**Schema**|Fixed & predefined|Flexible & dynamic|
|**Performance**|Joins can be costly|Faster reads (denormalized data)|

## Development Environment & Dataset
### Setup the Development Environment
- Use [`docker-compose.yml`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/5.%20MongoDB%20Fundamentals/docker-compose.yml) to spin up both a MongoDB and Mongo-Express container:
```yml
version: '3.8'

services:

  mongo:
    container_name: mongo-dev
    image: mongo:4.4.6
    volumes: 
      - ~/dockerdata/mongodb:/data/db    
    restart: on-failure
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
      MONGO_INITDB_DATABASE: auth
    networks:
      - my-mongo

  mongo-express:
    image: mongo-express:1.0.0-alpha.4
    restart: unless-stopped
    ports:
      - "8081:8081"
    environment:
      ME_CONFIG_MONGODB_SERVER: mongo-dev
      ME_CONFIG_MONGODB_ADMINUSERNAME: root
      ME_CONFIG_MONGODB_ADMINPASSWORD: example
      ME_CONFIG_BASICAUTH_USERNAME: admin
      ME_CONFIG_BASICAUTH_PASSWORD: tribes
    networks:
      - my-mongo
    depends_on:
      - mongo

networks:
  my-mongo:
    driver: bridge
```
- Change directories to project folder and run `docker-compose up`

### The Dataset
We will use the [Online Retail dataset from Kaggle](https://www.kaggle.com/datasets/tunguz/online-retail).

![fig3 - dataset](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/5.%20MongoDB%20Fundamentals/img/fig3%20-%20dataset.png)

### Schema Design

![fig4 - schema design](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/5.%20MongoDB%20Fundamentals/img/fig4%20-%20schema%20design.png)

- Goals for querying the data:
  - Access data by `Customer_id`
  - Query invoice by `invoiceNo`
  - Query by `Customer_id` and `InvoiceDate` (individual or sorted)
- Indices for `Customer_id`, `InvoiceNo`, and `InvoiceDate`
- Single or Multiple Docs option  

## Working with MongoDB
### Write Documents
- Refer to [`01_write_mongodb.py`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/5.%20MongoDB%20Fundamentals/scripts/01_write_mongodb.py):
```python
import pymongo

# Connect to mongodb
myclient = pymongo.MongoClient("mongodb://localhost:27017/",username='root',password='example')

mydb = myclient["test"] # select the database
mycol = mydb["testcol"] # select the collection

# Function to insert single document
def insert_single():
   # create a dummy document
    mydict = { "Cusomter_id": "A85123", "Country": "UK" }

    # write the document to the collection
    x = mycol.insert_one(mydict)

    # this is the id field of the new document
    print(x.inserted_id) 

# insert single document
#insert_single()


# Function to insert multiple documents at once
def insert_multiple():
   # create a dummy document
    mylist = []

    mylist.append({ "Cusomter_id": "B85123", "Country": "DE" })
    mylist.append({ "Cusomter_id": "C85123", "Country": "US" })

    # write the documents to the collection
    x = mycol.insert_many(mylist)

    # this is the id field of the new document
    print(x.inserted_ids) 

insert_multiple()
```
- First insert a single document: `insert_single()`
- Then insert multiple documents: `insert_multiple()`
- The id's will be updated in `testcol` in Mongo-Express:

![fig5 - 3 id's](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/5.%20MongoDB%20Fundamentals/img/fig5%20-%203%20ids.png)

### Read Documents
- Refer to [`02_read_mongodb.py`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/5.%20MongoDB%20Fundamentals/img/fig2%20-%20mongodb%20doc.png):
```python
import pymongo

myclient = pymongo.MongoClient("mongodb://localhost:27017/",username='root',password='example')
mydb = myclient["test"]
mycol = mydb["testcol"]

# find these documents where the customer_id equals A85123
myquery = {"Cusomter_id": "A85123"}
mydoc = mycol.find( myquery )

# return only specific parts of the document
#myreturnonly = { "_id": 0, "Cusomter_id": 1}
#mydoc = mycol.find( myquery, myreturnonly )

#print out doucument
for x in mydoc:
  print(x)

# how to sort the data that you will retrieve
# find order ASC .sort("Cusomter_id") or .sort("Cusomter_id", 1)
# find order DSC .sort("Cusomter_id",-1)  
```
- The script will print out all documents for `Customer_id`: A85123

![fig6 - mongo read](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/5.%20MongoDB%20Fundamentals/img/fig6%20-%20mongo%20read.png)

### Update Documents
- Refer to [`03_update_mongodb.py`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/5.%20MongoDB%20Fundamentals/scripts/03_update_mongodb.py):
```python
import pymongo

# Connect to mongodb
myclient = pymongo.MongoClient("mongodb://localhost:27017/",username='root',password='example')

mydb = myclient["test"] # select the database
mycol = mydb["testcol"] # select the collection

# add more attributes to a document
def add_more_to_document():
    myquery = { "Cusomter_id": "A85123" }
    newvalues = { "$set": { "Name": "Andreas" } }

    x = mycol.update_one(myquery, newvalues)   

    for x in mycol.find(myquery):
        print(x) 

#add_more_to_document()

# change attributes of a document
def update_document():
    myquery = { "Cusomter_id": "A85123" }
    newvalues = { "$set": { "Name": "Andreas Kretz" } }

    x = mycol.update_one(myquery, newvalues)   

    for x in mycol.find():
        print(x) 

update_document()
```
- The `add_more_to_document()` function adds more information to `Customer_id`: A85123

![fig7 - add attributes](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/5.%20MongoDB%20Fundamentals/img/fig7%20-%20add%20attributes.png)

- The `update_document()` function updates the contents

![fig8 - update document](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/5.%20MongoDB%20Fundamentals/img/fig8%20-%20update%20document.png)

### Delete Documents
- Refer to [`04_delete_mongodb.py`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/5.%20MongoDB%20Fundamentals/scripts/04_delete_mongodb.py):
```python
import pymongo

# Connect to mongodb
myclient = pymongo.MongoClient("mongodb://localhost:27017/",username='root',password='example')

mydb = myclient["test"] # select the database
mycol = mydb["testcol"] # select the collection

# Function to delete a single document
def delete_single():
   # find the document
    mydict = { "Cusomter_id": "A85123" }

    # delete one document to the collection, if there are MULTIPLE ones it only deletes ONE
    x = mycol.delete_one(mydict)

    # this is the id field of the new document
    print(x.deleted_count, "deleted")

# delete single document
delete_single()
```

![fig9 - delete](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/5.%20MongoDB%20Fundamentals/img/fig9%20-%20delete.png)

### Working with Sub-Documents
- Refer to [`05_sub_documents_mongodb.py`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/5.%20MongoDB%20Fundamentals/scripts/05_sub_documents_mongodb.py):
```python
import pymongo

# Connect to mongodb
myclient = pymongo.MongoClient("mongodb://localhost:27017/",username='root',password='example')

mydb = myclient["test"] # select the database
mycol = mydb["testcol"] # select the collection

# Look into: https://docs.mongodb.com/manual/reference/operator/update-field/

# Function to insert multiple documents with sub documents at once
def insert_multiple():
   # create a dummy document
    mylist = []

    mylist.append({ "Cusomter_id": "15311", "Country": "US", "simple_order_items": {"StockCode":"21791","UnitPrice":0.65, "Description":"STARS GIFT TAPE" ,"Quantity" : 3} })
    mylist.append({ "Cusomter_id": "14527", "Country": "US", "simple_order_items": {"StockCode":"21792","UnitPrice":2.95, "Description":"WHITE METAL LANTERN" ,"Quantity" : 1} })



    # write the document to the collection
    x = mycol.insert_many(mylist)

    # this is the id field of the new document
    print(x.inserted_ids) 

#insert_multiple()

def query_sub_document():
    # find these documents where the StockCode is 21791
    myquery = {"simple_order_items.StockCode": "21791"}
    mydoc = mycol.find( myquery)

    # return only specific parts of the document
    #myreturnonly = { "_id": 0, "name": 1}
    #mydoc = mycol.find( myquery, myreturnonly )

    #print out doucument
    for x in mydoc:
        print(x)

#query_sub_document()


# update a stock code of a sub document
def update_sub_document():
    myquery = { "Cusomter_id": "15311"}
    newvalues = { "$set": { "simple_order_items.StockCode": "22778" } }

    x = mycol.update_one(myquery, newvalues)   

    for x in mycol.find():
        print(x) 

#update_sub_document()
```
- `insert_multiple()`:

![fig10 - insert multiple](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/5.%20MongoDB%20Fundamentals/img/fig10%20-%20insert%20multiple.png)

- `query_sub_document()`:

![fig11 - query sub document](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/5.%20MongoDB%20Fundamentals/img/fig11%20-%20query%20sub%20document.png)

- `update_sub_document()`:

![fig12 - update sub document](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/5.%20MongoDB%20Fundamentals/img/fig12%20-%20update%20sub%20document.png)

### Working with Arrays of Sub-Documents
- Refer to [`06_arrays_mongodb.py`](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/5.%20MongoDB%20Fundamentals/scripts/06_arrays_mongodb.py):
```python
import pymongo

# Connect to mongodb
myclient = pymongo.MongoClient("mongodb://localhost:27017/",username='root',password='example')

mydb = myclient["test"] # select the database
mycol = mydb["testcol"] # select the collection

# Look into: https://docs.mongodb.com/manual/reference/operator/update-array/


# Function to insert multiple documents with sub documents at once
def insert_array():
   # create a dummy document

    myquery = {"Cusomter_id": "C84029", "complex_order_items": [{"StockCode" : "22622", "UnitPrice":0.65, "Description":"STARS GIFT TAPE" ,"Quantity" : 3},{"StockCode" : "22326", "UnitPrice":2.95, "Description":"WHITE METAL LANTERN" ,"Quantity" : 1} ]}


    # write the document to the collection
    x = mycol.insert_one(myquery)

    # this is the id field of the new document
    print(x.inserted_id) 

#insert_array()

def query_by_subdocument():
    # find these documents where the StockCode is 22622
    myquery = {"complex_order_items.StockCode": "22622"}
    mydoc = mycol.find( myquery)

    #print out doucument
    for x in mydoc:
        print(x)

#query_by_subdocument()

def add_subdocument_to_array():
    myquery =  {"Cusomter_id": "C84029"}
    newcar = { "$push": {"complex_order_items": {"StockCode" : "22637", "UnitPrice":2.55, "Description":"PIGGY BANK RETROSPOT" ,"Quantity" : 2  }} }

    x = mycol.update_one(myquery,newcar)

    print( x.matched_count , "updated")

    for x in mycol.find(myquery):
        print(x) 

#add_subdocument_to_array()
```

- `insert_array()`:

![fig13 - insert array](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/5.%20MongoDB%20Fundamentals/img/fig13%20-%20insert%20array.png)

- `query_by_subdocument()`:

![fig14 - query by subdocument](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/5.%20MongoDB%20Fundamentals/img/fig14%20-%20query%20by%20subdocument.png)

- `add_subdocument_to_array()`:

![fig15 - add subdocument](https://github.com/ndomah/3.-Fundamental-Tools/blob/main/5.%20MongoDB%20Fundamentals/img/fig15%20-%20add%20subdocument.png)
