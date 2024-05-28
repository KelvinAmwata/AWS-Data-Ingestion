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
  #Project Steps: 
## Creating an AWS Account 
- The first step is to create an AWS account if you do not have one, please follow the steps highlighted here: [https://docs.aws.amazon.com/accounts/latest/reference/manage-acct-creating.html]
- Create a firehose stream to which Lambda will ingest the data
- 
  


