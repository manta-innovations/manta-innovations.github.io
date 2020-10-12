---
layout: post
title: My top 10 AWS services
date: 2020-10-12
image: 
author: jhole89
tags: serverless, aws, cloud-native, aws services
---


AWS is huge. With its multitude of services and continuous updates, AWS is a playground for developers but the sheer scale of it can be overwhelming for newcomers. 

This is why I have put together a handy guide on my Top 10 AWS services, that I think all AWS developers should know, regardless of whether you are working on big data, 
machine learning, web apps, IoT, or networking, because you’ll likely need to interact with them at some point.

In no particular orderhe, here are my top 10: 

## EC2
**What is it?**
Scalable servers in the cloud.

**Why is it important?**

Ok, let’s get the big one done first. 

AWS EC2 is the backbone of AWS. It was one of the first services launched back in 2006 and took on the traditional concept of a data centre, but allows you to 
spin up and down servers with zero commitment at the click of a button. You can think of EC2 as a blank canvas in which you can install, configure, and run anything 
you want, even a minecraft server. Additionally you can launch preconfigured snapshots, called AMI’s, from the marketplace if you don’t want to install something yourself.

On top of that, EC2 forms a huge part of many of the AWS Certification exams...so learn EC2.

## ECS

**What is it?**

Scalable serverless container orchestration.

**Why is it important?**

Along with EC2, ECS is the other main way of running custom applications in AWS. 

It’s a managed (and can be completely serverless) container orchestration service. This means that instead of having to worry about the underlying hardware 
your app is running; you just have to ensure that your app can run inside a docker container. 

For a developer, this means their app can be easily ported to different cloud providers.  Security wise this means no patching of the host OS, and 
financially this means you only pay for the compute you require, rather than paying for the entire server as with EC2.

For application developers, knowing ECS is a must. Personally, ECS is my go to for custom applications. 

##RDS

**What is it?**

Managed Relational Databases

**Why is it important?**

Whether you prefer SQL or NoSQL, the reality is that SQL databases are a massive industry themselves. Many complex applications will require a relational 
database of some sort, and RDS is the best way to achieve this.

RDS takes away the pain of managing a relational DB yourself (along with the overhead cost of running a server to host it on) and supports numerous database 
engines including Oracle, MSSQL, MariaDB, MySQL, and of course the only real choice PostgreSQL. Having been around for a long time, RDS is another service that 
makes an appearance in AWS certifications, so make sure you spend some time understanding topics such as read-replicas and backing up from snapshots.

##DynamoDB

**What is it?**

Managed key-value and document database

**Why is it important?**

So we’ve talked about SQL in the form of RDS, now let's talk about NoSQL. DynamoDB is a managed serverless key value store, meaning once again you 
don’t worry about any underlying infrastructure, scaling, or maintenance. 

What makes DynamoDB unique is that rather than paying for the provisioned size of your database, you instead pay for the throughput required 
(how many reads/writes per second you require - which can be scaled up or down manually or on-demand) and the storage used. 

DynamoDB is schema-less, fast, resilient, and a great fit for any use case that wants a flat database hierarchy - it's your default NoSQL storage 
for AWS. As with some of the others, it's been around a long time and frequently pops up on many AWS certifications, though not to the same degree as 
RDS due to the decreased complexity.

##S3

**What is it?**

Simple scalable resilient object storage

**Why is it important?**

S3 is one of AWS’s simplest services. 

It is simply object storage, which you can store files on in the same way you would a traditional file system. What makes S3 important is that despite 
its simplicity it is incredibly flexible and used by many other AWS services as intermediary storage. Building a datalake? Use S3 for data storage. 
Want to use AWS CodePipeline - it uses S3 to store build artefacts. Want to query data on AWS Athena - it uses S3 to store query results.. 

On top of all of this, S3 also has a tiered pricing structure, where you only pay for storage that you use, but that cost depends on how quickly and 
frequently you need to access your data. 

With all of this it's no surprise that S3 also comes up frequently across all AWS certifications...learn S3, because you’ll definitely be using it.

##VPC

**What is it?**

Isolated virtual network for AWS resources

**Why is it important?**

VPC lets you provision a logically isolated section of the AWS Cloud which you can launch AWS resources in. 

Want to run an EC2 - you’ll need a VPC. Want to run an ECS cluster - you’ll need a VPC. Hosting a web application - you’ll need a VPC. 
VPC is an essential requirement for many AWS resources and includes everything from subnets and network gateways, through to route tables and NACL’s. 

Due to its complexity it forms a large part of many AWS certifications and it is a must know for anyone wishing to deploy to AWS.

##Lambda

**What is it?**

Run code without needing servers

**Why is it important?**

While EC2 and ECS are great for running continuous processes or applications, what about when you just want to run a small script either on a 
schedule or in response to an event? 

This is where Lambda comes into play. 

Rather than having to run an oversized server and orchestration tool, Lambda provides a serverless platform to orchestrate and run small scripts, as 
long as they complete within 15 minutes. Need a script to run in response to data being uploaded to S3? Use Lambda. Need a script to run every other morning 
at 10am? Use Lambda. 

Lambda is your go to for event driven processing and script execution.

##KMS

**What is it?**

Secure data encryption and key management

**Why is it important?**
Security is important, even more so on the cloud, where an incorrect setting can expose your resource to the rest of the world. 
KMS secures data and secrets in the cloud. 

Storing data on S3? Use a KMS key to encrypt it. Storing a confidential key in Secrets Manager - you need to use KMS for this.

Using KMS is crucial for building secure AWS native solutions.

##CloudWatch

**What is it?**

Logs, monitoring, and insights for resources

**Why is it important?**

Once we’ve got resources and applications running in the cloud, we need to be able to observe them and access their logs. If something goes down 
we need to know what exactly happened. 

This is where CloudWatch comes into play. 

With CloudWatch we can gather logs from both managed services and our own applications running on ECS and EC2. We can also use CloudWatch for event 
processing, and scheduling lambda events. 

So whether you’re deploying a service to AWS or scheduling event driven architecture, CloudWatch is crucial.

##AWS IAM

**What is it?**

User and permissions management

**Why is it important?**
Before you even start deploying a service to AWS you need to be thinking about IAM. IAM is how we assign privileges to both users and roles. 

So if you’re designing a service that requires access to a private S3 bucket, you’ll need to use IAM to assign s3 read access to the role your 
service is using. Learning IAM permissions is invaluable for application developers and security engineers alike. 

IAM is also another service that comes up frequently in AWS certifications so it’s worth familiarising yourself with the most common ones.

##Summary

Thanks for taking the time to read this guide - I hope it helps! As mentioned these are my own personal views, and the applications 
are not ranked in any particular order. 


If there is an AWS application that you swear by that hasn’t featured in this top 10 list, or you have any questions regarding these applications, 
I would love to hear from you. 

I will be sharing more information about AWS services and cloud computing, so follow me on Twitter [Joel Lutamn](https://twitter.com/joellutman) for more info on AWS. 




