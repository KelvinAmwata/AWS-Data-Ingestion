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
- We create an S3 bucket where our data will be stored.
<img width="915" alt="Screenshot 2024-05-28 at 2 33 02 PM" src="https://github.com/KelvinAmwata/AWS-Data-Ingestion/assets/83902270/9a9f1084-9afe-44c3-ba8b-694f70f64068">

- Ensure block all public access option is enabled 

<img width="961" alt="Screenshot 2024-05-28 at 2 31 20 PM" src="https://github.com/KelvinAmwata/AWS-Data-Ingestion/assets/83902270/321c155b-f8eb-4acd-8cc9-5abc1b87f8af">

- The other options can be left as shown below: 

<img width="576" alt="Screenshot 2024-05-28 at 2 35 14 PM" src="https://github.com/KelvinAmwata/AWS-Data-Ingestion/assets/83902270/61e9a206-1cdc-4e37-9f63-c3d98ae34c4e">

- For advanced settings, do no enable object lock. That will mean that you can not alter objects in the bucket
  
<img width="591" alt="Screenshot 2024-05-28 at 2 36 11 PM" src="https://github.com/KelvinAmwata/AWS-Data-Ingestion/assets/83902270/ce49d887-1b82-4c68-a39e-8eb8ac82fc64">




## Create a firehose stream to which Lambda will ingest the data
- In your AWS account search bar, search Amazon data firehose:

<img width="1073" alt="Screenshot 2024-05-28 at 2 49 08 PM" src="https://github.com/KelvinAmwata/AWS-Data-Ingestion/assets/83902270/5a7935c1-d019-42c1-9720-4328e8048740">
- Click on create firehose stream and then select the direct put option. We want Lambda to put data directly to the firehose
<img width="655" alt="Screenshot 2024-05-28 at 2 51 43 PM" src="https://github.com/KelvinAmwata/AWS-Data-Ingestion/assets/83902270/0a3a247a-386c-4468-9987-5e95add97d9d">
- You will see a stream name of the firehose, copy it and save it on your clipboard. we will need this name when writing our lambda code

- Under destination settings, for the s3 bucket, you can browse the name of the S3 bucket you created when creating an S3 bucket
<img width="960" alt="Screenshot 2024-05-28 at 3 02 58 PM" src="https://github.com/KelvinAmwata/AWS-Data-Ingestion/assets/83902270/a54be720-aa9f-442f-b4d6-404e3e945a6f">

- For the buffer hints, we can leave the buffer size as 5 MiB and buffer interval as 60 seconds. This means that firehose will be ingesting data from lambda at an interval of 60 seconds and the data ingested is 5 MiB in that interval 

<img width="930" alt="Screenshot 2024-05-28 at 3 05 07 PM" src="https://github.com/KelvinAmwata/AWS-Data-Ingestion/assets/83902270/0ca4c769-4c0b-45ed-9e91-8ec9f4c83d15">
- The other options can be left as they are apart from the service role. 
- For the service role, we want to ensure that Firehose has permissions to interact with s3 and lambda. So we create a role which firehose assumes. We attach permissions to the role so that firehose can be able to push data to s3 
- Click on the role created, and then click on add permissions
<img width="1078" alt="Screenshot 2024-05-28 at 3 33 20 PM" src="https://github.com/KelvinAmwata/AWS-Data-Ingestion/assets/83902270/7e35f66d-0d93-481c-9dff-7c72df1dd0fa">

- Click on attach policies 

<img width="229" alt="Screenshot 2024-05-28 at 3 34 02 PM" src="https://github.com/KelvinAmwata/AWS-Data-Ingestion/assets/83902270/d650ed5b-e700-44f5-9817-d9f54d507e76">


- Select s3 full access policy and AWS Lambda Full Access 


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
## Setting up Athena 

-AWS athena is an AWS service that enables users to query data stored in S3. 
- After Kinesis firehose pushes the data into an S3 bucket, the next step is to query the data using Athena so that the data can be ready for visualizations and other business needs
- Search for Athena on the seach bar.
- Click on settings, and then click on manage settings. Here we want to set the location where we will store the results of our queries.  So open a duplicate and go back to S3 and create another bucket. Remember to adhere to s3 naming conventions
- Now we create the database. On the query editor,  enter DDL: CREATE DATABASE databasename. Replace databasename with the name you want for your database
-  We now have athena set up and we are ready to crawl the data
-  
## Crawling the Data
- After lambda has ingested the data and pushed it to data firehose which then pushes it to an S3 bucket, we want to crawl the data so that we can create a schema and a table which are important when writing a query.
- Search for AWS glue on the search bar.
- On the left side, look for crawler under data catalog:

<img width="261" alt="Screenshot 2024-05-28 at 3 44 00 PM" src="https://github.com/KelvinAmwata/AWS-Data-Ingestion/assets/83902270/064b9f6d-e9ac-4ec3-bc48-436c124bfe53">

- Click data source and under s3 path, select the s3 bucket name you created as a data source. Sometimes you may see an error after browing the name of your bucket. In such a case, ensure you put a forward slash(/) after the name for it to work. You can also press escape key. Either of those options will work

<img width="609" alt="Screenshot 2024-05-28 at 3 50 05 PM" src="https://github.com/KelvinAmwata/AWS-Data-Ingestion/assets/83902270/1c39f266-d037-4a0a-83a5-9d713fdb91e0">

- We've now created our crawler and all that it needs for it  to crawl the S3 bucket is to grant it permissions via a role.
- Under configuration, click create IAM role 
<img width="786" alt="Screenshot 2024-05-28 at 4 05 57 PM" src="https://github.com/KelvinAmwata/AWS-Data-Ingestion/assets/83902270/0405dc0f-a89f-40fc-be00-189fee0d9361">
- We then select the database name we created when setting up athena. You can give the prefix of the table you want the crawler to create after it has crawled the data:
  
- <img width="828" alt="Screenshot 2024-05-28 at 4 44 46 PM" src="https://github.com/KelvinAmwata/AWS-Data-Ingestion/assets/83902270/98b87f2d-5a1b-43e8-a4db-5cbf8b041d19">

- Our crawler is now all set and we click review and update
- Click run crawler and it will take a few seconds to complete:
  
<img width="1049" alt="Screenshot 2024-05-28 at 4 54 53 PM" src="https://github.com/KelvinAmwata/AWS-Data-Ingestion/assets/83902270/78f3f2e5-e71b-4e56-bb5e-bbf17353d843">

  ## Results
- Switch back to athena to query data
- Select the database we created
- The crawler must have already created a table. If you cannot see it, click the refresh button. If you cannot still see it, go back to the I AM role assigned to the crawler and ensure that it has all the necessary permissions: Athena Full Access and S3 full access
- Click on the table and select preview table:
- Athena automatically creates a query which you can run to see the results of the output
  ~~~
  SELECT * FROM "kevdatabase"."kevstocks_project_fh" limit 100;

<img width="906" alt="Screenshot 2024-05-28 at 5 05 21 PM" src="https://github.com/KelvinAmwata/AWS-Data-Ingestion/assets/83902270/a1c8c5af-0d24-46df-95a2-809e84fa860d">


## Transforming the Data
- One of the important things when creating a data pipeline is to transform the data
- We can achieve this by creating an ETL job.
- Go back to glue
- Click on ETL jobs and then script editor:

<img width="739" alt="Screenshot 2024-05-28 at 5 24 10 PM" src="https://github.com/KelvinAmwata/AWS-Data-Ingestion/assets/83902270/d374f438-4497-498a-a9a1-1c759537e226">

- We will write our code in SQL but then use a Python "wrapper" that will allow AWS Glue to submit our queries to Athena



~~~
import boto3

client = boto3.client('athena')

# Refresh the table
queryStart = client.start_query_execution(
    QueryString="""
    CREATE TABLE kevdatabase.stocks_analysis
    WITH (
        external_location = 's3://paquetbucket/',
        format = 'PARQUET',
        write_compression = 'SNAPPY',
        partitioned_by = ARRAY['time']
    )
    AS
    SELECT
        open,
        high,
        low,
        time
    FROM kevdatabase.kevstocks_project_fh;
    """,
    QueryExecutionContext={
        'Database': 'kevdatabase'
    },
    ResultConfiguration={
        'OutputLocation': 's3://dataingestionkev-athena/'
    }
)

print(queryStart)

~~~
- Click run
- Our job has now run and we can go back to athena to check the new table created with the modifications

~~~
SELECT * FROM "kevdatabase"."stocks_analysis" limit 10;

<img width="892" alt="Screenshot 2024-05-28 at 5 41 49 PM" src="https://github.com/KelvinAmwata/AWS-Data-Ingestion/assets/83902270/bded9c5d-5b39-47ca-a68e-bf65021f3012">

~~~

# Visualizing the Data 

- Having extracted the data, the next thing is to visualize it for business needs
- For this project, we use PowerBI which is a widely used business intelligence tool
- To connect PowerBI to Athena, you can watch this video to get the steps:
- [https://www.youtube.com/watch?v=ClBQ3_p7T_A]
-  [https://www.youtube.com/watch?v=FKdCr6vmq-o&t=326s] 




