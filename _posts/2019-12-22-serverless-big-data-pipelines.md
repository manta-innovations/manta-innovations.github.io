---
layout: post
title:  Big Data Pipelines as Serverless Microservice's
date:   2019-12-22 14:38:00 +0100
image:  stepfunc_data_pipeline.jpeg
author: jhole89
tags:   serverless, apache spark, aws step functions
---

### Apache Spark vs Serverless
Apache Spark, Serverless, and Microservice's are phrases you rarely hear spoken about together, but that's all about to 
change...

As someone who's 
worked as an SME in Apache Spark since 2016 I've found it common to be working with large Hadoop clusters 
(either on premise or as part of an EMR cluster on AWS), which can run up large bills either in server acquisition or 
running costs as they require clusters to be "always on", even though pipelines may be utilising a small amount of the 
cluster for only a portion of the time. On top of the monetary cost of running this, there is the added manpower cost 
required to maintaining, patching, upgrading, and monitoring such a cluster.

In contrast we have the Serverless movement, which aims to abstract away many of these issues with managed services, 
where you only pay for what you use. Examples of how Serverless 1.0 has materialised is with managed compute services 
such as AWS Lambda and AWS Glue, services where you simply submit small scripts and define an execution runtime, paying 
only for the execution time and memory requirement; and managed storage services such as AWS DynamoDB and AWS S3, 
services where you read and write data via an API, paying only for the storage time and I/O throughput.

If we can bring these two concepts together we can leverage the power of Apache Spark to deliver value via Big Data 
solutions without worrying about the maintenance of large Hadoop clusters, and also split stages into small 
micro-services that can easily scale independently and diverge down different work streams in parallel.

### Hello ~~world~~ AWS Step Functions
Welcome to AWS Step Functions, a managed service that lets us coordinate multiple AWS services into workflows. 
AWS Step Functions can be used for a number of use cases and workflows including sequence batch processing, transcoding 
media files, publishing events from serverless workflows, sending messages from automated workflows, or orchestrating 
big data workflows.

![post-thumb]({{site.baseurl}}/assets/images/blog/stepfunc_data_pipeline_numbered.jpeg){:class="img-fluid rounded float mr-5 mb-2 ml-5" :height="auto" width="90%"}
<center><sup>Full description of architecture at end **</sup></center>

In the diagram above, we illustrate a big data workflow of sourcing data into a datalake, ETL'ing our data from source 
format to Parquet, and using a pre-trained Machine Learning model to predict based on the new data.

This may seem complex with diverging parallel workflows, but we've been able to achieve a huge amount without writing 
much code at all - the only parts to this that actually require us to write application code are our Sourcing Lambda 
(1), Spark ETL (8), View Lambda (11), and Sagemaker Script (12), all of which may be less than 100 lines of code. Apart 
from that everything else remains a serverless managed AWS service which just need to be configured via a scripting 
tool such as Terraform.

On top of our primary use case of ingesting files from any external sources, landing them to S3, transforming them with 
business logic into parquet, and using the data to predict using an ML model; with such an architecture we have also 
achieved the ability to:

![post-thumb]({{site.baseurl}}/assets/images/blog/asl_example.png){:class="img-fluid rounded float-right ml-5" :height="auto" width="50%"}

* Use AWS Glue Crawlers and AWS Athena to query the data at any stage
* Handle any errors or timeouts across the entire stack, route the error to a SNS topic, then onto any support team
* Configure retries at a per service or entire stack level
* Inspect any file movement or service state via a simple query or HTTP request to DynamoDB
* Configure spark resources independently of each job, without worrying about cluster constraints or resource sharing 
via YARN
* Tie each stage neatly together without having to worry about orchestrating times via a simple configuration language
* Orchestrate the entire pipeline on a CRON schedule or via event triggers
* Ability to monitor ETL workflows via UI
* Reduce overall costs by only paying for consumed resources
* Develop further pipelines quickly and in parallel by splitting the Sourcing Lambda, Spark ETL, View Lambda, and 
Sagemaker Scripts into individual repositories
* Treat each ETL stage as a microservice which only requires data on S3 as its interface between services
* Ability to recreate services without worrying about underlying infrastructure (via Terraform)

![post-thumb]({{site.baseurl}}/assets/images/blog/step_func_ui.png){:class="img-fluid rounded float-left" :height="auto" width="50%"}

<br/><br/><br/>
Comparing this to a traditional single stack server based architecture, we would lose the flexibility and fast 
development times that our ETL stage based micro-services give us; have to manage numerous servers, clusters, and 
docker containers to run the necessary services; code in custom error handling to each service and trigger an email 
alert (which would also need to be managed); manage and configure any databases for multi zone replication and 
fail-over; run a number of distributed Hadoop applications; lose the ability to track service failures or outages. On 
top of all of this we would also lack any UI to give developers or engineers any feedback or way of inspecting the state 
of an ETL workflow.

![post-thumb]({{site.baseurl}}/assets/images/blog/state_machines_private.png){:class="img-fluid rounded float mr-5 mb-2"}
<center><sup>State Machine UI showing run metrics</sup></center>

#### Not the only option, but...
It's worth pointing out that some of these benefits are not limited to just AWS Step Functions. Airflow, Luigi, and 
NiFi are all alternative orchestration tools that are able to provide us with a subset of these benefits, in particular 
scheduling and a UI; however these rely on running on top of EC2 instances which in turn would have to be maintained - 
if the server were to go offline our entire stack would be non-functional, which is not acceptable to any high 
performing business, and still lacks many of the other benefits discussed such as stack level error and timeout 
handling among others.

AWS Step Functions is a versatile service which allows us to focus on delivering value through orchestrating components. 
Used in conjunction with serverless applications we can avoid waterfall architecture patterns by easily swapping in 
different services to fulfil roles (e.g. we could easily swap DynamoDB out for AWS RDS without any architecture burden) 
during development and allow developers to focus on the core use case, rather than solutionising.

As we've demonstrated, it can be a powerful and reliable tool in leveraging big data within the serverless framework 
and should not be overlooked for anyone exploring orchestration of big data pipelines on AWS. Used in conjunction with 
the serverless framework, it can enable us to quickly deliver huge value to without the traditional big (data) headaches.

##### *Notes*
Full architecture description is as follows:
* Source data from a number of external systems (API, FTP, SQL DB, FileSystem, Queue) via a lambda (1) and loading this 
in its raw format to (2) an S3 bucket, our datalake's landing area, whilst recording any files processed or state to 
(3) DynamoDB.
* Trigger two parallel workstreams: an AWS Glue Crawler (4) to crawl the landing bucket and update the AWS Glue Catalog 
(5), which in turn makes the data available to SQL-like querying via AWS Athena (6) for data consumers (7). The other 
workstream starts an AWS Glue job (8) using Apache Spark to ETL the source data from it's raw format in the landing S3 
bucket (3) to parquet format in our trusted/refined datalake zone (9), another S3 bucket, which in turn records state 
and file progress to DynamoDB (3).
* Once this second stream has completed, trigger another set of diverging parallel workstreams. The first mirrors the 
previous stream, triggering an AWS Glue Crawler (10) to crawl the trusted/refined bucket and update the AWS Glue 
Catalog (5), again in turn making the data available to AWS Athena (6); the second uses AWS Lambda (11) to execute an 
SQL statement on AWS Athena (6) to create a view on top of our refined data; whilst the third triggers AWS Sagemaker 
(12) to read our refined data and make a prediction using a pre-trained model, storing its results to our forecast 
datalake bucket (13).
* Once this third workstream has completed, triggers the final stage, another AWS Glue Crawler (14) over our forecast 
bucket (13) to update the AWS Glue Catalog (5) and make these results available to users (7) via Athena (6).
* On top of this we also catch any error happening within any stage of the Step Function and send the error to AWS SNS 
(15) which emails the error to our admin or support team (16) to address.
