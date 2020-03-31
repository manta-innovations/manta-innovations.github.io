---
layout: post
title: Testing Serverless Workflows
date: 2020-03-27
image: Serverless-workflow-testable.png
author: jhole89
tags: serverless, testing, aws
---

### What is Serverless?
Serverless is a design pattern which aims to remove many issues typically found with running and maintaining your own servers 
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
and other services, we need to test the boundaries. This is often done by utilising mocks to simulate responses from 
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
server for each PR rather each commit, and would be calling the services from outside (where our CI server lives), 
rather than utilising local mocks.

![Testing Boundaries]({{site.baseurl}}/assets/images/blog/testing-boundaries.png){:class="img-fluid rounded float" :height="auto" width="60%"}

### The Problem with Managed Serverless
Now that we have a firm grasp of the different types of testing, lets work through a real world example. Given this demo
workflow:

 1. We have some simple code running inside an AWS Lambda to get data from an api, do some processing with it, and publish
  the results to an S3 bucket
 2. The S3 bucket has an event trigger that sends an alert to an SNS topic when new data is published to it
 3. The SNS topic sends an email to our users letting them know that the data is available to download from a link
 4. Users access the link, which is an AWS API Gateway endpoint to authorise access to the download
 
![Our demo workflow]({{site.baseurl}}/assets/images/blog/Serverless-workflow.png){:class="img-fluid rounded float" :height="auto" width="75%"}

We could expect the code for the Lambda to have unit and integration tests written alongside and included in the source 
repo. These may utilise mocks and utilities such as wiremock to run small configurable local only mock servers to test 
how this simple code would handle the various HTTP response codes and capturing the messages being sent to S3 - this is 
testing the boundaries of the Lambda.

In a traditional stack, where instead of Lambda, S3, SNS, and API-Gateway we were managing servers running docker, an 
FTP server, an SMTP server, and NGINX; we could regression test these by running containers for the FTP, Lambda code, 
SMTP server, NGINX, and wiremock on our CI box. By triggering the Lambda code with a range of wiremock api paths, we 
can regression check the local FTP and SMTP instances for side-effects and unexpected behaviour. However how do we do 
this with managed services? AWS SNS and a traditional SMTP server may be *similar*, but they're not the same. So there's
really no point trying to mimic this workflow with replacement services, however if we only test the Lambda code then we
are leaving much of our workflow untested. What happens if someone logs into the console and changes the SNS topic name?
The Lambda will still pass it's unit and integration tests, and it will still publish data to the S3 bucket. However 
the SNS topic will no longer receive the event, and won't be able to pass on alert to our users - our workflow is broken,
and even worse we're not aware of it.

This is the catch-22 of testing serverless - as our workflows become more complicated, we need rigorous testing, 
but the more managed services we include, the less testable our workflow becomes using only unit and integration tests. 
This is why regression/systems testing becomes more important with serverless workflows, and why it should become more 
of the norm.

![Worfkflow boundaries]({{site.baseurl}}/assets/images/blog/Serverless-workflow-testable.png){:class="img-fluid rounded float" :height="auto" width="75%"}

### Testing Serverless Workflows
So now that we understand what we want to test, and why its important, we need to find a way of testing it. The traditional
approach still used by many would be to deploy the stack onto an environment, where someone can manually trigger and 
evaluate the workflow. This is testing the happy path, as it doesnt evaluate all the permutations of different 
components changing. Additionally, due to the manual process involved we are unlikely to be able to evaluate this on each
PR, and instead may only do this once per release which could contain many changes. Should we find any regressions, 
it becomes harder to identify the root cause due to the multiple changes that have been implemented between releases.

So how do we do better? How do we thoroughly test the workflow for each PR commit, and ensure that our workflow remains
stable when individual components are able to change? Well, what we can do is take the same approach used for unit and 
integration tests, and look at how we can test our remit (in this case our entire workflow) as a black box. We can 
achieve this by spinning up infrastructure around our workflow, then run a suite of tests to start the workflow, assert 
on the results at the end of the workflow, and finally destroy our test infrastructure afterwards - to do which we need
to leverage IaC (Infrastructure as Code) tools such as [terraform](https://www.terraform.io/).

For our demo workflow, we would achieve this by deploying an instance of AWS API Gateway and an S3 Bucket (to provide test
data from), which the Lambda at the start of our workflow will connect to. We can then run our suite of tests to trigger 
the Lambda with a selection of endpoints passed from the API Gateway, and then assert the expected results exist in the 
workflow S3 bucket via the workflow API Gateway (or the correct error is raised).

![Regression workflow]({{site.baseurl}}/assets/images/blog/serverless-workflow-ci.png){:class="img-fluid rounded float" :height="auto" width="75%"}

Conceptually, this is very similar to running a local Wiremock server as part of our test suites - well, why can't we 
just do that instead of worrying about building infrastructure? The problem here is that running Wiremock within our test
suite would only be local to that test environment, and we wouldn't be able to expose the localhost Wiremock endpoint to
the Lambda - we would need a DNS for that. By launching API Gateway, we generate a public (or private) URL which we can
pass to our applications and test them from the outside in; compared to unit or integration tests, where we run tests
alongside our code.

With this approach, we can now automate the traditional manual QA testing, and ensure we cover a much wider spectrum of
BDD test cases, including scenarios such as *"What alert do/should our users receive if the API is unavailable?"*. In 
traditional unit/integration testing we wouldn't be able to answer or test for this, as this process is handled outside 
of the Lambda - we could test what happens to the Lambda in the event of the external API becoming available, but not 
how downstream processes would react - we'd be reliant on someone manually trying to mimic this scenario, which simply 
doesn't scale. Furthermore, utilising IaC we can run a huge barrage of these larger workflow tests in parallel (paying 
only for what resources we actually use), and easily scale these up to incorporate elements of load and chaos testing; 
instead of being reactive to our workflow breaking, we can push the limits to establish our redundancy prior to experience 
event outages. 

### Conclusions
Hopefully I've sold you on the idea of regression/systems testing, and why as we more to a more serverless world, we need
to establish a more holistic view on testing our systems as a whole, rather than only the components in isolation. That's
not to say that we should abandon the faithful unit test in favour of systems testing, but why we should not fall into
the fallacy that just because our "code" is tested, our systems and workflow's are also tested. This also highlights why 
Development, QA, and DevOps are not activities do be done in isolation by separate teams. Having a key understanding
of each is required to implement and test such a workflow, and that ideally both the workflow and test framework should 
be implemented by a single cross functional team, rather than throwing tasks over the fence.

If any of that sounds interesting and you'd like to know more, you can reach me at 
[joel@manta-innovations.co.uk](mailto:joel@manta-innovations.co.uk). There is a corresponding live demo that implements
this workflow and the ideals behind it, which can be shown on request; and feel free to reach out for assistance or training
with Cloud, Data, or DevOps solutions, or any of our other workstreams here at [Manta Innovations Ltd](https://manta-innovations.co.uk/).