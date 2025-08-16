# MongoDB_Data_Wrangling
# *Overview*

This repository contains a step-by-step guide and example commands for a data wrangling project using MongoDB. The goal is
to take a raw sales_records.json file, import it into a MongoDB collection, and then perform a series of data observation,
cleaning, and analysis steps to prepare the data for reporting and insight generation. This project demonstrates a typical
workflow for a data engineer working with NoSQL databases.

# *Prerequisites*
To follow along with this project, you will need:
* MongoDB: A running instance of a MongoDB server by checking status (linux terminal)
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

     # note: same process to price_per_unit (double), total_price (double) 
   - Checking missing
     ```python
     # checking field with None or null value simple way is find distinct value
     collection.disctinct('payment_method')

     # note: same process to region, status     
   - Checking duplicate
     ```python
     # In sales order duplicate that not allowed in order_id
      order_id_pipeline = [
        {
          '$group':{'_id':'$order_id', 'count':{'$sum':1} }
        },
        {
          '$match':{'count':{'$gt':1}}
        }
      ]

     duplicates_order = list(collection.aggregate(order_id_pipeline))
     duplicates_order
5. Data Cleaning
   - Data type correction & checking the result
     ```python
     # data type correction
     collection.update_many({'quantity':{'$not':{'$type':'int'}}}, [{'$set':{'quantity':{'$toInt':'$quantity'}}}])

     # checking the result
     qty_not_int = collection.find({'quantity':{'$not':{'$type':'int'}}},{'_id':0, 'order_id':1, 'quantity':1})
     qty_not_int_list = list(qty_not_int)
     qty_not_int_list
   - Update None type data & checking the result
     ```python
     # Update None data type on payment_method to Cash because the value close to it
     collection.update_many({'payment_method':None},{'$set':{'payment_method':'Cash'}})

     # Checkinh the result ensure None type no longer exist on payment
     payment = collection.aggregate([
      {'$group':
         {'_id':'$payment_method', 'total_price':{'$sum':'$total_price'}}       
      },
        {'$sort':{'total_price':-1}}
     ])

     payment_df = pd.DataFrame(payment)
     payment_df
   - Missing value handling & checking the result
     ```python
     # Reshasing note to special_handling to YES/NO & add it for missing field notes
     collection.update_many({'notes':{'$exists':True}},{'$rename':{'notes':'special_handling'}})
     collection.update_many({'special_handling':{'$exists':True}},{'$set':{'special_handling':'YES'}})
     collection.update_many({'special_handling':{'$exists':False}},{'$set':{'special_handling':'NO'}}) 
7. Insight Generation
   - Best selling item
     ```python
     best_selling_pipeline = [
       {
         '$group': {'_id':'$item_name', 'total_price':{'$sum':'$total_price'} }
       },
       {'$sort':{'total_price':-1} },
       {'$limit':5 }    
     ]
      best_selling = list(collection.aggregate(best_selling_pipeline))
   - Sales distributon by age
     ```python
     age_group_pipeline = [
        {'$bucket':{
            'groupBy':'$customer_info.age',
            'boundaries':[0, 18, 25, 35, 50, 65, 100],
            'default': 'Other/Unknown',
            'output':{'total_price':{'$sum':1}}
            }
       },
       {'$sort':{'_id':1}}
     ]
     age_group_result = list(collection.aggregate(age_group_pipeline))
   - Sales distribution by region
     ```python
     region_distribution_pipeline = [
      { '$group':{
            '_id':'$region', 'total_sale':{'$sum':'$total_price'}
        }
      },
      {'$sort':{'total_sale':-1}},
      {'$limit':5}
     ]
     region_distribution = list(collection.aggregate(region_distribution_pipeline))
   - Status transaction breakdown
     ```python
     status_pipeline = [
      {'$group':{
            '_id':'$status', 'total_sale':{'$sum':'$total_price'}
        }
      },
      {'$sort':{'total_sale':-1}}
     ]
     status_pipeline_result = collection.aggregate(status_pipeline)
   - Payment Method to total sale
     ```python
     payment_method_pipeline = [
      {'$group':{
            '_id':'$payment_method', 'total_sale':{'$sum':'$total_price'}
        }
      },
      {'$sort':{'total_sale':-1}}
     ]
     payment_method = collection.aggregate(payment_method_pipeline)
