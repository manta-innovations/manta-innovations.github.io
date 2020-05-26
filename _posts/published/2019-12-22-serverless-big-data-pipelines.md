---
layout: post
title: Serverless Big Data Pipelines
date: 2019-12-22 14:38:00 +0100
image: serverless-data-pipelines/stepfunc_data_pipeline.jpeg
author: jhole89
tags: serverless, apache spark, aws step functions
---

### Apache Spark vs Serverless

Apache Spark, Serverless, and Microservice's are phrases you rarely hear spoken about together, but that's all about to
change...

As someone who works as a SME in Apache Spark it's been common for me to be working with large Hadoop clusters
(either on premise or as part of an EMR cluster on AWS), which run up large bills even though the clusters are mostly
idle, seeing short periods of intense compute when pipelines run. On top of the monetary cost of running this, the added
burden of keeping these clusters healthy eats into time and energy that should be spent on adding value to the data.

In contrast we have the Serverless movement, which aims to abstract away many of these issues with managed services,
where you only pay for what you use. Examples of how Serverless 1.0 has materialised is with managed compute services
such as AWS Lambda and AWS Glue, services where you simply submit small scripts and define an execution runtime, paying
only for the execution time and memory requirement; and managed storage services such as AWS DynamoDB and AWS S3,
services where you read and write data via an API, paying only for the storage time and I/O throughput.

If we can bring these two concepts together we can leverage the power of Apache Spark to deliver value via big data
solutions without the overheard of large Hadoop clusters. In doing so we can split stages into small micro-services
that can easily scale independently regardless of cluster limitations, and orchestrate entire workflows (beyond apache
spark) to make use of parallel execution.

### Hello ~~world~~ AWS Step Functions

Welcome to AWS Step Functions, a managed service that lets us coordinate multiple AWS services into workflows.
AWS Step Functions can be used for a number of use cases and workflows including sequence batch processing, transcoding
media files, publishing events from serverless workflows, sending messages from automated workflows, or orchestrating
big data workflows.

![post-thumb]({{site.baseurl}}/assets/images/blog/serverless-data-pipelines/stepfunc_data_pipeline_numbered.jpeg){:class="img-fluid rounded float mr-5 mb-2 ml-5" :height="auto" width="90%"}

<center><sup>Full description of architecture at end **</sup></center>

In the diagram above, we illustrate a big data workflow of sourcing data into a datalake, ETL'ing our data from source
format to Parquet, and using a pre-trained Machine Learning model to predict based on the new data. This may seem
complex with diverging parallel workflows, but we've been able to achieve a huge amount without writing
much code at all - the only parts to this that actually require us to write application code are our Sourcing Lambda
(1), Spark ETL (8), View Lambda (11), and Sagemaker Script (12), all of which may be less than 100 lines of code. Apart
from that everything else remains a serverless managed AWS service which just need to be configured via a scripting
tool such as Terraform.

On top of our primary use case of ingesting files from any external sources, landing them to S3, enriching them with
business logic and transforming the data to parquet, and using the data to predict using an ML model. Leveraging such an
architecture we have achieved the ability to:
![post-thumb]({{site.baseurl}}/assets/images/blog/serverless-data-pipelines/step_func_ui.png){:class="img-fluid rounded float-right ml-5" :height="auto" width="50%"}

- Query the data at any stage via AWS Athena
- Handle any errors or timeouts across the entire stack, route the error to a SNS topic, then onto any support team
- Configure retries at a per service or entire stack level
- Inspect any file movement or service state via a simple query or HTTP request to DynamoDB
- Configure spark resources independently of each job, without worrying about cluster constraints or YARN resource
  sharing
- Orchestrate stages neatly together in many different ways (sequential, parallel, diverging)
- Trigger the entire pipeline on a CRON schedule or via events
- Monitor ETL workflows via UI

Comparing this to a traditional single stack server based architecture, we also gain numerous advantages for both the
development process and service management:

- Increase development velocity and flexibility by splitting the Sourcing Lambda, Spark ETL, View Lambda, and Sagemaker
  Scripts into micro-service's or monorepo's
- Treat each ETL stage as a standalone service which only requires data in S3 as the interface between services
- Recreate our services quickly and reproducibly by leveraging tools such as Terraform
- Create and manage workflows in a simple readable configuration language
- Avoid managing servers, clusters, databases, replication, or failure scenario's
- Reduce our cloud spend and hidden maintenance costs by consuming resources as a service

![post-thumb]({{site.baseurl}}/assets/images/blog/serverless-data-pipelines/asl_example.png){:class="img-fluid rounded float-right mr-5" :height="auto" width="50%"}

### Configuration as Code

As mentioned, one of the clear benefits of using AWS Step Functions is being able to describe and orchestrate our
pipelines with a simple configuration language. This enables us to remove any reliance on explicitly sending signals
between services, custom error handling, timeouts, or retries, instead defining these with the Amazon States Language -
a simple, straightforward, JSON-based, structured configuration language.

With the states language we declare each task in our step function as a state, and define how that state transitions
into subsequent states, what happens in the event of a state's failure (allowing for different transitions depending
on the type of failure), and how we want a state to execute (sequential or in parallel alongside other states).

### Not the only option, but...

It's worth pointing out that some of these benefits are not limited to just AWS Step Functions. Airflow, Luigi, and
NiFi are all alternative orchestration tools that are able to provide us with a subset of these benefits, in particular
scheduling and a UI; however these rely on running on top of EC2 instances which in turn would have to be maintained - 
if the server were to go offline our entire stack would be non-functional, which is not acceptable to any high
performing business, and still lacks many of the other benefits discussed such as stack level error, timeout
handling, and configuration as code, among others.

![post-thumb]({{site.baseurl}}/assets/images/blog/serverless-data-pipelines/state_machines_private.png){:class="img-fluid rounded float mr-5 mb-2"}

<center><sup>State Machine UI showing run metrics</sup></center>

AWS Step Functions is a versatile service which allows us to focus on delivering value through orchestrating components.
Used in conjunction with serverless applications we can avoid waterfall architecture patterns by easily swapping in
different services to fulfil roles (e.g. we could easily swap DynamoDB out for AWS RDS without any architecture burden)
during development and allow developers to focus on the core use case, rather than solutionising.

As we've demonstrated, it can be a powerful and reliable tool in leveraging big data within the serverless framework
and should not be overlooked for anyone exploring orchestration of big data pipelines on AWS. Used in conjunction with
the serverless framework, it can enable us to quickly deliver huge value to without the traditional big (data) headaches.

##### _Notes_

Full architecture description is as follows:

- Source data from a number of external systems (API, FTP, SQL DB, FileSystem, Queue) via a lambda (1) and loading this
  in its raw format to (2) an S3 bucket, our datalake's landing area, whilst recording any files processed or state to
  (3) DynamoDB.
- Trigger two parallel workstreams: an AWS Glue Crawler (4) to crawl the landing bucket and update the AWS Glue Catalog
  (5), which in turn makes the data available to SQL-like querying via AWS Athena (6) for data consumers (7). The other
  workstream starts an AWS Glue job (8) using Apache Spark to ETL the source data from it's raw format in the landing S3
  bucket (3) to parquet format in our trusted/refined datalake zone (9), another S3 bucket, which in turn records state
  and file progress to DynamoDB (3).
- Once this second stream has completed, trigger another set of diverging parallel workstreams. The first mirrors the
  previous stream, triggering an AWS Glue Crawler (10) to crawl the trusted/refined bucket and update the AWS Glue
  Catalog (5), again in turn making the data available to AWS Athena (6); the second uses AWS Lambda (11) to execute an
  SQL statement on AWS Athena (6) to create a view on top of our refined data; whilst the third triggers AWS Sagemaker
  (12) to read our refined data and make a prediction using a pre-trained model, storing its results to our forecast
  datalake bucket (13).
- Once this third workstream has completed, triggers the final stage, another AWS Glue Crawler (14) over our forecast
  bucket (13) to update the AWS Glue Catalog (5) and make these results available to users (7) via Athena (6).
- On top of this we also catch any error happening within any stage of the Step Function and send the error to AWS SNS
  (15) which emails the error to our admin or support team (16) to address.
