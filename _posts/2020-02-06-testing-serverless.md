---
layout: post
title: Testing Serverless (...with Serverless)
date: 2020-02-01
image:  
author: jhole89
tags: serverless, testing, aws
---

### What is Serverless?
Serverless is a movement which aims to remove many issues typically found with running and maintaining your own servers 
or services by instead utilising managed cloud services. This allows development teams to worry less about updating 
services for security vulnerabilities, managing disaster recovery and redundancy, scaling services for unseen increases 
in demand, or federating users across services; and instead allows them to focus on delivering value and benefit quickly 
and efficiently.

Instead of running our own services which require large upfront purchasing and procurement costs, along with the 
ongoing maintenance cost (both monetary and developer time), in serverless we eschew this cost and only pay for what 
we use. Examples of how Serverless 1.0 has materialised is with managed compute services such as AWS Lambda and AWS 
Glue, services where you simply submit small scripts and define an execution runtime, paying only for the execution 
time and memory requirement; and managed storage services such as AWS DynamoDB and AWS S3, services where you read 
and write data via an API, paying only for the storage and I/O throughput.

### Unit, Integration, Regression Testing
In both traditional and serverless development, when building apps and workflows that involve calls to databases, api's, 
and other services, we need to test the extremities. This is often done by utilising mocks to simulate responses from 
outside our app or workflow. A large amount of mocks often highlights a large amount of side-effects, which while 
something we can minimise by following functional programming paradigms, are often unavoidable. Mocks are most 
frequently found in unit and integration tests, where we just want to test small functions or workflows, however they 
can also be found in regression tests where we wish to control the outside world. Before continuing it's worthwhile to 
define the difference between these, as integration and regression tests are often poorly understood:

* Unit test: the smallest type of test, where we test a function or class method. Every function should always have at 
least one accompanying unit test to ensure a function acts as expected. When following Test Driven Development these 
are the kind of tests we write first, asserting what we expect our soon to be written function will do. We expect 
these tests to run automatically on a Continuous Integration server for each commit. Given
```scala
def addOne(input: Int): Int = input + 1
```
we would expect a corresponding test which may look something like
```scala
addOne(-1) shouldEqual 0
addOne(0) shouldEqual 1
```
* Integration test: larger tests, where we test a workflow within our app which may call many functions. Integration 
tests should be more behaviour focused and target how our system expects to run given different inputs.  We expect 
these tests to run automatically on a Continuous Integration server for each commit. Given an application with a single 
entrypoint signature of `def main(args: Seq[String]): Unit` we may expect an integration test to look something like 
(note the use of a mocked url)
```scala
main(Seq("localhost:8000", "/fake-url", "30s")) shouldRaise 404
main(Seq("localhost:8000", "/mocked-url", "30s")) shouldNotRaiseException
```
* Regression test: the largest type of test, also thought of as a systems test. While unit and integration tests look to 
test how our application behaves, regression tests look to test how our systems behave and prevent unexpected regressions 
due to development. This can also be scaled into more advanced types of testing such as load testing or chaos engineering. 
While integration testing of an api crawler may test what happens to the crawler when the api goes offline by utilising 
a local mock, regression testing should test what happens to all services should that api go offline, often on a 
close to real life test environment. We would except regression tests to run automatically on a Continuous Integration 
server for each PR rather each commit, and would be calling the services from outside, rather than utilising local mocks.

### The problem with serverless
Now that we have a firm grasp of the different types of testing, lets work through a real world example. Given some simple
Python code that runs inside AWS Lambda as the initial stage of a CRON scheduled AWS Step Function to get data from an 
api and record the results to AWS DynamoDB. We could expect the code for this lambda to have unit and integration tests 
written alongside and included in the source repo, which utilise mocks and utilities such as wiremock to run small 
configurable local only mock servers to test how this simple code would handle the various HTTP response codes and 
capturing the messages being sent up to DynamoDB. 

In a traditional stack, where instead of DynamoDB and Lambda we were managing a server running MongoDB and docker, we 
could regression test these by running containers for MongoDB, Python, and wiremock on our CI box. By triggering the 
Python code with a range of wiremock api paths, we can regression check the local MongoDB instance for side-effects 
and excepted behaviour. However how do we do this with managed services?