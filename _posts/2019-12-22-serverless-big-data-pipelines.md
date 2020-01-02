---
layout: post
title:  Big Data Pipelines as a Serverless Microservice
date:   2019-12-22 14:38:00 +0100
image:  stepfunc_data_pipeline.jpeg
author: jhole89
tags:   serverless, aws, apache spark, aws lambda, aws step functions
---

Apache Spark, Serverless, and Microservices are phrases you rarely heard spoken about together. When working with 
Apache Spark on "Big Data" applications I've found it common to be working with large Hadoop clusters (either on
premise or as part of an EMR cluster on AWS), which run up large bills as require infrastructure to be "always on", 
even though pipelines may be utilising a small amount of the cluster for only a portion of the time. On top of the cost
of running this, there is the added cost of the manpower required to maintaining, patching, upgrading, and monitoring
such a cluster. In contrast we have the Serverless movement, which aims to abstract away all of these issues with 
managed services, where you only pay for what you use (the misnomer behind Serverless meaning no servers is 
incorrect - they're still there, you're just paying someone else to run and maintain them). Examples of how Serverless
1.0 has materialised is with managed compute services such as AWS Lambda<sup>[1]</sup> and AWS Glue<sup>[2]</sup>, 
services where you simply submit small scripts and define an execution runtime, paying only for the execution time and 
memory requirement; and managed storage services such as AWS DynamoDB<sup>[3]</sup> and AWS Neptune<sup>[4]</sup>, 
services where you read and write data via an API, paying only for the storage time and I/O throughput. By marrying 
these two concepts together we are able to leverage the raw power of Apache Spark to not only deliver value via Big Data 
solutions without worrying about the maintenance of large Hadoop clusters, but also split stages into small microservices 
that can easily scale independently and diverge down different workstreams in parallel.

![post-thumb]({{site.baseurl}}/assets/images/blog/stepfunc_data_pipeline_numbered.jpeg){:class="img-fluid rounded float mr-5 mb-2"}

AWS Step Functions is a managed service that lets us coordinate multiple AWS services into serverless workflows. In the
diagram above, we illustrate a simple Big Data workflow of sourcing data and writing to our datalake, ETL'ing our data
from source format to Parquet, and using a pre-trained Machine Learning model to predict based on the new data.<sup>[5]</sup> 
This may seem complex with lots of diverging parallel workflows, but look what we've been able to achieve without 
writing much code at all - the only parts to this that actually require us to write application code are our Sourcing 
Lambda (1), Spark ETL (8), View Lambda (11), and Sagemaker Script (12); apart from that everything else remains a fully 
managed service which just need to be configured via a scripting tool such as Terraform. With such an architecture we 
are able to:

* Ingest files from any external sources, lang them in native format to S3, transform them with business logic into 
parquet (an efficient, query-able binary format) and write back to S3, use the new data to predict using an ML model and
store the results back to S3
* Persist all data to S3 and use AWS Glue Cralwers and AWS Athena to query the data at any stage
* Develop further pipelines quickly and in parallel by splitting the Sourcing Lambda, Spark ETL, View Lambda, and 
Sagemaker Scripts, into individual repositories
* Treat each ETL stage as a microservice which only requires data on S3 as its interface between services
* Capture any error's across the entire stack and route the error to a SNS topic, then onto any support team registered
to that topic
* Configure retries at a per service or entire stack level or at different rates
* Inspect any file movement or service state via a simple query or HTTP request to DynamoDB
* Tie each stage neatly together without having to worry about orchestrating times
* Orchestrate the entire pipeline on a CRON schedule or via event triggers
* Impose service level timeouts
* Ability to recreate services without worrying about underlying infrastructure (via Terraform)

Comparing this to a traditional single stack server based architecture, we would lose the flexibility and fast 
development times that our ETL stage based microservices give us, have to manage 





<sup>[1] AWS Lambda is priced at $0.20 per 1M requests, $0.000016667 for every GB-second (EU-Ireland region); and  natively 
supports runtime's of Java, Go, PowerShell, Node.js, C#, Python, and Ruby, along wide a Runtime API which allows you to 
use any additional programming languages.</sup>

<sup>[2] AWS Glue is priced at $0.44 per DPU-Hour (EU-Ireland region), billed per second; and supports Spark via Scala, and 
PySpark.</sup>

<sup>[3] AWS DynamoDB is a NoSQL key-value/document database priced at $1.4135 per million write request units, $0.283 
per million read request units, and $0.283 per GB-month storage (EU-Ireland region).</sup>

<sup>[4] AWS Neptune is a NoSQL graph database priced at $0.10 per GB-month storage and $0.22 per 1 million requests I/O
(EU-Ireland region).</sup>

<sup>[5] We achieve this by sourcing data from a number of external systems (API, FTP, SQL DB, FileSystem, Queue) via a 
lambda (1) and loading this in its raw format to (2) an S3 bucket, our datalake's landing area, whilst recording any 
files processed or state to (3) DynamoDB. Once this stage is complete the Step Function triggers two parallel 
workstreams, one triggering an AWS Glue Crawler (4) to crawl the landing bucket and update the AWS Glue Catalog (5), 
which in turn makes the data available to SQL-like querying via AWS Athena (6) for data consumers (7); whilst the other 
workstream starts an AWS Glue job (8) using Apache Spark to ETL the source data from it's raw format in the landing S3 
bucket (3) to parquet format in our trusted/refined datalake zone (9), another S3 bucket, which in turn records state 
and file progress to DynamoDB (3). Once this second stream has completed the Step Function triggers the next stage, 
another set of diverging parallel workstreams. The first mirrors the previous stream, whereby it triggers an AWS Glue 
Crawler (10) to crawl the trusted/refined bucket and update the AWS Glue Catalog (5), again in turn making the data 
available to AWS Athena (6); the second uses AWS Lambda (11) to execute a SQL statement on AWS Athena (6) to create a 
view on top of our refined data; whilst the third triggers AWS Sagemaker (12) to read our refined data and make a 
prediction using a pre-trained model, storing itsresults to our forecast datalake bucket (13). Once this third 
workstream has completed, the Step Function triggers it's final stage, another AWS Glue Crawler (14) over our forecast 
bucket (13) to update the AWS Glue Catalog (5) and make these results available to users (7) via Athena (6). On top of 
this we also catch any error happening within any stage of the Step Function and send the error to AWS SNS (15) which 
emails the error to our admin or support team (16) to address.</sup>