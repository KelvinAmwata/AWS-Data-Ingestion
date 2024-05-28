# Project Overview  
The AWS-Data-ingestion project is about building a data pipeline in the AWS cloud environment. The project ingests data from an API using Lambda which is offered as Function as a service(FAAS) by AWS. Using Lambda, we load the data into  Amazon firehose which is a data ingestion service. Amazon Kinesis firehose then loads the data into an S3 bucket. We then use AWS glue to crawl the data so that we can have a schema and create database tables. We then employ AWS glue to create an ETL job which helps us in performing transformations to the data- data cleaning sort of. The data is then loaded into Amazon query service- Amazon athena- where we can perform SQL operations to manipulate the data in accordance with requirements. The homestretch of our project is to visualize the data so that we can derive onsights that can be used for business decisions, as need be. We achieve this by connecting AWS athena with a data visualization tool. In this project, we will be connecting Athena with PowerBi which is a business intelligence tool.
## Data Sources 

The source of data for this project is a publicly available open API for IBM stocks. API link: https://www.alphavantage.co/query?function=TIME_SERIES_INTRADAY&symbol=IBM&interval=5min&apikey=demo
### Tools,Services, and Software
- AWS Account
- AWS Lambda
- AWS Firehose
- AWS Athena
- AWS Glue
- AWS S3
- PowerBI (https://powerbi.microsoft.com/en-us/downloads/)

# Project Steps: 
## Creating an AWS Account 
- The first step is to create an AWS account if you do not have one, please follow the steps highlighted here: [https://docs.aws.amazon.com/accounts/latest/reference/manage-acct-creating.html]
## Create an S3 Bucket 
- We create an S3 bucket where 

<img width="961" alt="Screenshot 2024-05-28 at 2 31 20 PM" src="https://github.com/KelvinAmwata/AWS-Data-Ingestion/assets/83902270/321c155b-f8eb-4acd-8cc9-5abc1b87f8af">


## Create a firehose stream to which Lambda will ingest the data
- In your AWS account, search Amazon data firehose {https://us-east-1.console.aws.amazon.com/firehose/home?region=us-east-1#/streams}
- 
## Write a code to ingest the data(Lambda - FAAS)
  ~~~ python 

import boto3

import json 

import urllib3

def lambda_handler(event, context):
    http = urllib3.PoolManager()
    
    response = http.request("GET", "https://www.alphavantage.co/query?function=TIME_SERIES_INTRADAY&symbol=IBM&interval=5min&apikey=demo")
    
    response_dict = json.loads(response.data.decode(encoding="utf-8", errors="strict"))
    
    time_series = response_dict.get("Time Series (5min)")
    
    if time_series:
        for timestamp, data in time_series.items():
            processed_dict = {}
            processed_dict["time"] = timestamp  # Add timestamp to processed dictionary
            processed_dict["open"] = data.get("1. open")
            processed_dict["high"] = data.get("2. high")
            processed_dict["low"] = data.get("3. low")
            processed_dict["close"] = data.get("4. close")
            processed_dict["volume"] = data.get("5. volume")
            
            msg = json.dumps(processed_dict) + '\n'  # Convert dictionary to JSON string
            
            fh = boto3.client('firehose')
            
            reply = fh.put_record(
                DeliveryStreamName="PUT-S3-5FVQo",
                Record = {'Data': msg }
            )
    else:
        print("Time series data not found in the response.")

  ~~~
  


