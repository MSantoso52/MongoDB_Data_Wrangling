# MongoDB_Data_Wrangling
# *Overview*

This repository contains a step-by-step guide and example commands for a data wrangling project using MongoDB. The goal is
to take a raw sales_records.json file, import it into a MongoDB collection, and then perform a series of data observation,
cleaning, and analysis steps to prepare the data for reporting and insight generation. This project demonstrates a typical
workflow for a data engineer working with NoSQL databases.

Prerequisites
To follow along with this project, you will need:
- MongoDB: A running instance of a MongoDB server by checking status (linux terminal)
  ```bash
  systemctl status mongod 
- Jupyterlab with pymongo library installed.
  ```bash
  pip install pymongo
- sales_records.json: The JSON data file to be imported.
  
# *Project Flow*
The project structure is following:
1. Data Import
   To import the json file into mongodb database there are steps need to be done;
   - python connection to mongodb database, create database, create collection
     ```python
     import pymongo

     conn = pymongo.MongoClient(MONGODB_URI)
     db = conn[DATABASE_NAME]
     collection = db[COLLECTION_NAME]
   - load JSON file
     ```python
     import json

     with open('sale_records.json','r') as f:
       data = json.load(f)

   - import JSON into mongodb
     ``` python
     collection.insert_many(data)
3. Data Observation
   - Checking result of imported data & schema
     ```python
     collection.find().limit(1)
   - Checking inconsistent data type
     ```python
     # quanity data type should be int
     qty_not_int = collection.find({'quantity':{'$not':{'$type':'int'}}},{'_id':0, 'order_id':1, 'quantity':1})

     qty_not_int_df = pd.DataFrame(qty_not_int)
     qty_not_int_df['quanity'].size
   - Checking missing
   - Checking duplicate
5. Data Cleaning
6. Insight Generation
