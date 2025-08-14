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
