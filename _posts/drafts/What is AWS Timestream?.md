---
layout: post
title: What is AWS Timestream?
date: 2020-11-20
image: timestream/cover.jpg
author: jhole89
tags: startups, culture
---

Amazon Timestream is AWS’s newest addition to their storage offerings. It’s a fast, scalable, and serverless time series database; something in my 
experience both the community and businesses have been clamoring for.

Recently I spent an afternoon testing out Timestream and I thought I’d share what I learned during that time, and my initial impressions.

## What is a time series database? 

A time series database is a system optimized for storing and serving time series data. A time series being a sequence of records represented 
as data points over an interval. While time series data can be stored in a traditional relational database, these often experience scaling issues. 

Typical time series use cases include any type of data where we repeatedly measure values or metrics at regular intervals, this includes;
- IoT data (e.g. weather readings, device statuses)
- DevOps analytics (e.g. CPU utilisation, memory allocation, network transmission)
- App analytics (e.g. clickstream data, page load times, healthchecks, response times)

Timestream targets these use cases - in fact they even provide some sample IoT and DevOps data to play around with, which is exactly what I did.

## Secure Serverless Infrastructure <3

Setting up Timestream is incredibly easy.

As a completely serverless offering, there is little to configure and no sizing or throughput settings to worry about. Additionally, being serverless 
it follows a rolling release schedule, meaning you are able to take advantage of new features as they become available, rather than worry about version 
upgrades. Additionally, in line with other AWS managed solutions you simply pay for usage rather than the underlying infrastructure.

![post-thumb]({{site.baseurl}}/assets/images/blog/timestream/Timestream - database configuration.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

One of the few settings you specify is the encryption key. Timestream enforces data encryption and thankfully this setting cannot be turned off. 
Your options here allow you to specify how your data is encrypted (both at rest and in flight) using a CMK stored in AWS KMS.

## Intelligent data storage

The other main setting is how long your data lasts in each of Timestream’s storage options. 

Timestream currently has 2 types of storage:

1. A write optimized memory store; where data initially lands and is automatically deduplicated - I’ll talk about this more in a second.
2. A read optimized magnetic store; for cost effective long term storage.

![post-thumb]({{site.baseurl}}/assets/images/blog/timestream/Timestream - storage types.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

When setting up a Timestream table you’re required to set a retention policy to specify how long data should exist in each store before moving 
onto the next (from memory, to magnetic, to deletion), with the minimum values being 1hr for the memory store (up to a maximum 1 year) and 1 
day for the magnetic store (up to a maximum 200 years).

## Never worry about duplicate records again

I briefly mentioned data duplication and I want to focus on that a bit more. Data duplication is a big problem in traditional relational databases. 
Large CRM systems often may find themselves with multiple entries for identical data points if uniqueness is not enforced by the schema. 

Timestream deals with this with an interesting approach, in that if an identical record is received, the write optimized memory store deduplicates 
this into a single record. This uses a “first writer wins” approach, so whichever record is sent first will be written to disk, with the duplicate record being thrown away. 

As far as I’ve been able to test these duplicate records must be 100% identical, however I would love to see an option in the future to tweak this 
down to a lower similarity threshold (e.g. two records being treated as duplicates if they are 90% similar).

## The Timestream data model

Being a type of NoSQL database, Timestream has its own type of data model distinct from both traditional SQL data models, and many other 
NoSQL data models. Timestream is considered a schema-less database as there is no enforced schema. 

However it still uses concepts such as databases and tables, along with Timestream specific concepts, so lets define these:

- **Database**: a collection of tables;
- **Table**: an encrypted container that holds our time series records;
- **Record**: a combination of a timestamp, 1 or more dimensions, and a single measure;
- **Dimensions**: attributes that describe metadata of record (e.g. region, AZ, vpc, hostname for DevOps metric data) - always stored as varchar;
- **Measure**: the single named data value representing the measurement (e.g. cpu usage, memory allocation for DevOps metric data)- can be boolean, 
  bigint, varchar, or double;

![post-thumb]({{site.baseurl}}/assets/images/blog/timestream/Timestream - table Dimensions.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

![post-thumb]({{site.baseurl}}/assets/images/blog/timestream/Timestream - table measures.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

The Timestream UI presents this model in a familiar column wise structure, however due to the data model it doesn’t support the standard 
CRUD operations you might expect. While records can be created and read back, they cannot be updated or deleted. Instead records can only 
be removed when they reach the retention limit on the magnetic storage.

## Schema-less SQL on steroids

Despite Timestream being a schema-less NoSQL database, as mentioned it does present its data model as a column wise structure which anyone 
familiar with SQL will feel at home with. 

![post-thumb]({{site.baseurl}}/assets/images/blog/timestream/Timestream - SQL QUERY.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

Timestream enables data to be queried using standard SQL (supporting CTE’s, filtering, and aggregations), 
with over 250 scalar and aggregate functions and additional time series interpolation for data points that may be missing or lost in transmission. 
This means you can easily group data into different chunks of time and perform aggregates, even if certain points in time were missing data. 
The one limitation here is that while Timestream does support table joins, these can only be on the same table (a join back to itself), though this 
does make sense when you remember that tables are schema-less.

![post-thumb]({{site.baseurl}}/assets/images/blog/timestream/Timestream-SQL query results.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

## Integrations
Whilst having a handy SQL interface is great, for many this is not the best way to present data to users and stakeholders, 
especially when trying to highlight trends or patterns over time. Thankfully Timestream comes with a number of built in integrations, 
both within the AWS ecosystem and for third party tools. 

These include:
- Dashboards and charts via Amazon QuickSight or Grafana
- Data ingestion AWS SDK and CLI, from AWS IoT via IoT rules, from Kinesis Data Analytics streams, or from Telegraf
- Connecting traditional SQL workbench tools over JDBC

![post-thumb]({{site.baseurl}}/assets/images/blog/timestream/Timestream - Quicksights.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

## Closing thoughts

Overall, playing around with Timestream was very interesting. I think it’s a powerful service that further rounds out AWS’s storage offerings, 
and comes with some exciting features that are specific to Timestream. As mentioned, I was impressed by the deduplication of data, and I’d be 
keen to see this developed further, or being rolled out as a configurable option for other storage services - I think AWS could really be onto 
something with this feature. On top of that, having it being both schema-less and giving us an SQL interface is a nice middle ground for those 
not entirely sold on NoSQL data models.

There’s a lot to like with Timestream, and I think that it could potentially be a good fit for lots of use cases. While Amazon mentions use 
cases such as DevOps metrics and IoT data; I think it could also have great potential for clickstream, stock market, currency, and asset 
management data - really anything where we want to be taking repeated measurements over time. 

## What do you think?
I’m sure there’s many more use cases than the ones I mentioned above, so let me know what use cases you can think of, 
or perhaps are already using Timestream for. 

I’d also be interested to hear about how well Timestream scales for large datasets - it wasn’t something I was able to test that rigorously 
during my couple of hours with it. So any insight into performance and scaling would be great.

For more tech insight follow me on Twitter at [@JoelLutman](https://twitter.com/joellutman); where I tweet and blog about AWS, serverless, 
big data, and software best practice.

