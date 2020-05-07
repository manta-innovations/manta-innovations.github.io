---
layout: post
title: Testing Serverless Workflow's
date: 2020-03-27
image: testing-serverless/Serverless-workflow-testable.png
author: jhole89
tags: serverless, testing, aws, cloud-native
---

Serverless is a design pattern which aims to remove many issues development teams typically face when maintaining 
servers or services, enabling them to focus on delivering value and benefit quickly and efficiently. Instead of spending 
time and money configuring and managing servers, we can turn to a serverless paradigm and offload 
this burden onto Cloud Native parties who specialise in these tasks. This frees up time and resources to focus on solving 
problems in our core domain.

However using a large amount of serverless resources also has it's drawbacks, in particular the difficulties in testing.
In this article I aim to discuss some of these problems, and propose a solution for testing heavily serverless workflow's.

### The Different Types of Testing
When building applications it's important that we write comprehensive test coverage to ensure our application behaves as
expected, and protects us from unexpected changes during iteration. In both traditional and serverless development, 
when building apps and workflow's that involve calls to other services, we need to test the boundaries. This is often 
done by utilising mocks to simulate responses from outside our app or workflow. A large amount of mocks often highlights 
a large amount of side-effects, which while something we can minimise by following functional programming paradigms, are 
often unavoidable. Before continuing it's important to understand the difference between unit, integration, and 
regression tests, as they are often easily mixed up:

* Unit test: the smallest type of test, where we test a function. When following Test Driven Development these are the 
kind of tests we write first, asserting what we expect our soon to be written function will do. We expect these tests 
to run automatically on a Continuous Integration server for each commit. Given a function
`def addOne(input: Int): Int = input + 1` we would expect a corresponding test which may look something like

    * `addOne(-1) shouldEqual 0`
    * `addOne(0) shouldEqual 1`

* Integration test: larger tests, where we test a workflow which may call many functions. These are more behaviour 
focused and target how our system expects to run given different inputs. As with unit tests, we expect these tests to 
run automatically on a Continuous Integration server for each commit. Given an application with an 
entrypoint `def main(args: Seq[String]): Unit` we may expect an integration test to look something like

    * `main(Seq("localhost:8000", "/fake-url", "30s")) shouldRaise 404`
    * `main(Seq("localhost:8000", "/mocked-url", "30s")) shouldNotRaiseException`

* Regression test: the largest type of test, also thought of as a systems test. While unit and integration tests look to 
test how our application behaves during changes to it, regression tests look to test how our systems behave due to our
application changing, and prevent unexpected regressions due to development. While integration testing of an api crawler 
may test what happens to the app when the api goes offline by utilising a local mock, regression testing should test what 
happens to all services should that api go offline. We except these tests to run automatically on a Continuous Integration 
server for each PR rather each commit.

![Testing Boundaries]({{site.baseurl}}/assets/images/blog/testing-serverless/testing-boundaries.png){:class="img-fluid rounded float" :height="auto" width="60%"}

### Problems with Testing Serverless Workflow's
Now that we have a firm grasp of the different types of testing, lets work through a real world example where we will see
that relying only on unit and integration tests is not enough for even simple serverless workflow's. Given this demo
workflow:

 1. We have some simple code running inside an AWS Lambda to get data from an api, do some processing with it, and publish
  the results to an S3 bucket
 2. The S3 bucket has an event trigger that sends an alert to an SNS topic when new data is published to it
 3. The SNS topic sends an email to our users letting them know that the data is available to download from a link
 4. Users access the link, which is an AWS API Gateway endpoint to authorise access to the download
 
![Our demo workflow]({{site.baseurl}}/assets/images/blog/testing-serverless/Serverless-workflow.png){:class="img-fluid rounded float" :height="auto" width="75%"}

We could expect the code for the Lambda to have unit and integration tests written alongside. These may utilise mocks 
and utilities such as wiremock to test how this simple code would handle the various HTTP response codes, and capture 
the messages being sent to S3. This is testing the boundaries of the Lambda, however this leaves much of our workflow 
untested.

In a traditional stack, where instead of utilising serverless we would be managing servers (e.g. an FTP server for 
S3, an SMTP server for SNS, NGINX for API-Gateway); we could regression test these by running containers for them 
alongside wiremock on our CI box. By triggering the application code with a range of api paths, we can regression test 
the local container instances for side-effects and unexpected behaviour. However how do we do this with managed/cloud-native 
services which are not available in the form of local containers?

AWS SNS and a traditional SMTP server may be *similar*, but they're not the same, and any tests using it as a replacement
would provide little benefit. However if we only test the Lambda code then we are leaving much of our workflow untested. 
What happens if someone logs into the console and changes the SNS topic name? The Lambda will still pass it's unit and 
integration tests, and it will still publish data to the S3 bucket. However the SNS topic will no longer receive the 
event, and won't be able to pass on alert to our users - our workflow is broken, and even worse we're not aware of it.

![Worfkflow boundaries]({{site.baseurl}}/assets/images/blog/testing-serverless/Serverless-workflow-testable.png){:class="img-fluid rounded float" :height="auto" width="75%"}

This is the catch-22 of testing managed/cloud-native serverless - as our workflow's become more complicated, we need rigorous testing, 
but the more services we include, the less tested our workflow becomes. This is why regression/systems testing 
becomes more important with serverless workflow's, and why it should become more of the norm.

### Regression Testing Serverless Workflow's
So now that we understand what we want to test, and why its important, we need to find a way of testing it; and to 
achieve this we need to be using regression tests.

The traditional approach still used by many would be to deploy the stack onto an environment, where someone can manually 
trigger and evaluate the workflow. This is testing the happy path, as it doesnt evaluate all the permutations of different 
components changing. Additionally, due to the manual process involved we are unlikely to be able to evaluate this on each
PR, and instead may only do this once per release which could contain many changes. Should we find any regressions, 
it becomes harder to identify the root cause due to the multiple changes that have been implemented between releases. 
This also doesn't scale well when we have more complex workflow's that utilise parallel and diverging streams (for an 
example of such read my blog on [building serverless data pipelines]({{site.baseurl}}/2019/12/22/serverless-big-data-pipelines/)).

So how do we do better? How do we thoroughly test the workflow and ensure that our workflow remains stable when 
individual components are able to change? Well, what we can do is take the same approach used for unit and 
integration tests, and look at how we can test our remit (in this case our entire workflow) as a black box. We can 
achieve this by spinning up infrastructure around our workflow, then run a suite of tests to start the workflow, assert 
on the results at the end of the workflow, and finally destroy our test infrastructure afterwards - to do which we need
to leverage IaC (Infrastructure as Code) tools such as [terraform](https://www.terraform.io/).

For our demo workflow, we would achieve this by deploying managed/cloud-native services, which the Lambda at the start of our 
workflow will connect to, in lieu of the real external API. We can then run a suite of tests to trigger the Lambda, and 
assert the expected results exist at the end of our workflow via the via the workflow API Gateway.

![Regression workflow]({{site.baseurl}}/assets/images/blog/testing-serverless/serverless-workflow-ci.png){:class="img-fluid rounded float" :height="auto" width="75%"}

Conceptually, this is very similar to running a local wiremock server as part of our test suites - well, why can't we 
just do that instead of worrying about building infrastructure? The problem here is that running wiremock within our test
suite would only be local to that process, and we wouldn't be able to expose the wiremock endpoint to the Lambda - we 
would need a DNS for that. By launching API Gateway, we generate a public (or private) URL which we can pass to our 
applications and test them from the outside in; compared to unit or integration tests, where we run tests alongside our 
code.

With this approach, we can now automate the traditional manual QA testing, and ensure we cover a much wider spectrum of
BDD test cases, including scenarios such as *"What alert do/should our users receive if the API is unavailable?"*. In 
traditional unit/integration testing we wouldn't be able to answer or test for this, as this process is handled outside 
of the Lambda. We could test what happens to the Lambda in the event of the external API becoming available, but not 
how downstream processes would react - we'd be reliant on someone manually trying to mimic this scenario, which doesn't 
scale. Furthermore, utilising IaC we can run a huge barrage of these larger workflow tests in parallel, and 
easily scale these up to incorporate elements of load and chaos testing. Instead of being reactive to our workflow 
breaking, we can push the limits to establish our redundancy prior to experiencing event outages. 

### Conclusions
Hopefully I've sold you on the idea of regression/systems testing, and why as we move to a more serverless world, we need
to establish a more holistic view on testing our systems as a whole, rather than only the components in isolation. That's
not to say that we should abandon the faithful unit test in favour of systems testing, but why we should not fall into
the fallacy that just because our "code" is tested, our systems and workflow's are also tested. This also highlights why 
Development, QA, and DevOps are not activities do be done in isolation by separate teams. Having a key understanding
of each is required to implement and test such a workflow, and that ideally both the workflow and test framework should 
be implemented by a single cross functional team, rather than throwing tasks over the fence.

If any of that sounds interesting and you'd like to know more, you can reach me at 
[joel@manta-innovations.co.uk](mailto:joel@manta-innovations.co.uk). There is a corresponding live demo that implements
this workflow and the ideals behind it, which can be shown on request. Feel free to reach out for assistance or training
with Cloud, Data, or DevOps solutions, or any of our other workstreams here at [Manta Innovations Ltd](https://manta-innovations.co.uk/).
