---
layout: post
title: What is MLOps - AWS Online Summit Series
date: 2020-05-23
image: aws-mlops/ML_Dev_Ops.png
author: jhole89
tags: cloud, aws, conference, ml, machine-learning
---

Welcome, this is part of my ongoing series on AWS’s recent Online Summit 2020, where I write about my thoughts and
learnings from the range of topics presented. As always, the content here describes my own thoughts and understandings
from the material presented, not the views of the presenters, who I do not speak for.

### What is MLOps? - AWS Online Summit Series

Having originally come from a Data Science and ML background, before focusing on Cloud implementations and Serverless, I
was interested in AWS AI Specialist Solutions Architect Julian Bright’s talk on Machine learning ops: DevOps for data
science.

### Ops, Ops, Ops

MLOps (Machine Learning Ops) is another new term, following the pattern of DevOps and GitOps (not to forget DevSecOps,
DataOps, AIOps, and anything else you can append “Ops” onto), that I’m seeing more and more in the industry.

MLOps largely revolves around solving similar issues as DevOps does - deployments. The only difference here being that
instead of focusing on application deployment, MLOps is focused on model deployment.

If I'm honest, I'm not sure we need another “Ops” title just to differentiate between a model and an application. In the
end of the day a well written ML model is often a containerised application or binary object anyway, which are not that
dissimilar from a standard containerised app or jar.

But then again I work in the industry that brought us phrases such as “Python Ninja”, “10x Developer”, and recursive
mindbender "SPARQL" (SPARQL Protocol and RDF Query Language); so maybe I shouldn’t be too critical.

### ML still has a long way to go

Julian opened by giving us some interesting facts about Machine Learning in industry. In particular, he quoted an
Algorithma survey which found that “55% of companies have not deployed a machine learning model” (by “companies”
Algorithma are referring to enterprise business, of which they had 750 respondents, though they do not publish what
metric they used to classify a business as enterprise).

Having worked on both the data science and software sides, I’m honestly not that surprised.

ML is still a relatively novel concept to many enterprise businesses. From my personal experience many enterprise use
cases are much more BI focused, and have yet to understand and tap into what a ML model can do for them, over their
traditional dashboards and reports. The Algorithma survey shows 21% of the total survey were still evaluating use cases
to see if they even had a need for an ML model.

![post-thumb]({{site.baseurl}}/assets/images/blog/aws-mlops/algorithma.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}
<center><sup>Source: https://info.algorithmia.com/2020</sup></center>

In addition to this, the Algorithma survey, also found that of those 45% that had deployed a machine learning model,
approximately 68% took somewhere between 1 week to over 1 year to deploy a single model.

Keep in mind that in a best practice CI/CD workflow we deploy multiple times a day (and in GitOps we deploy each
commit). So a single deployment taking even a week should be unacceptable in modern software design.

![post-thumb]({{site.baseurl}}/assets/images/blog/aws-mlops/algorithma2.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}
<center><sup>Source: https://info.algorithmia.com/2020</sup></center>

### Why so slow?

Julian went on to talk about how the actual ML code is only a small part of an ML solution. Good machine learning
solutions require accurate data, which needs, among others; 
- collection 
- verification
- feature
- engineering
- metadata management
- infrastructure management
- automation
- process management
- team structure

All of these can introduce their own challenges. One of which he highlighted was that different teams could own parts of
the process, each requiring their own handoff, integration points, and development workflow.

![post-thumb]({{site.baseurl}}/assets/images/blog/aws-mlops/process.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

This is something that I’ve definitely seen across all aspects of software development, it is not specific to ML.

My own opinion on this matter is that the developer/engineer/scientist who develops the source code (whether that be an
app or a model), should be the one to take it through its entire lifecycle through to deployment. This in my opinion
speeds up the delivery, and provides a more coherent and consistent code base for the model, and avoids “throwing it
over the fence” to other teams.

### Deploying and Orchestrating ML Models

Julian went on to talk about how we can use the AWS Developer Tools (Code Build, Deploy, Pipeline, etc) not only for
deploying traditional apps but for ML models too, which follows the patterns demonstrated in [Loh Yiang Meng’s talk:
CI/CD at scale: Best practices with AWS DevOps services]({{site.baseurl}}/2020/05/22/cicd-at-scale/).

This did make me think, if we can use the same processes and tooling for both application and ML models, then why should
we treat ML models any differently to applications?

Anyway, I deviate. So now that we are able to deploy our model, how do we orchestrate it?

Compared to apps, many ML and Data Science models are written more as scripts than a service; and as highlighted, we may
need to perform some small steps such as data cleansing and validation prior to using our model.

### Serverless ML

Julian demonstrated how we can use a number of tools to orchestrate SageMaker scripts to perform these steps. He
mentioned a number of operators including Apache Airflow, Netflix Metaflow, Kubernetes, and AWS Step Functions (which
provides first class support for SageMaker scripts).

This was interesting to me, I'm a huge AWS Step Functions fan, having used it extensively within my serverless AWS
implementations.

![post-thumb]({{site.baseurl}}/assets/images/blog/aws-mlops/step_func.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}
<center><sup>Source: https://aws.amazon.com/step-functions/use-cases</sup></center>

Despite being around since 2015, AWS Step Functions does not have first class support for most other AWS services, and
requires you to write a small Lambda function to invoke the actual service. The more AWS services that Step Functions
gives first class support for, the better.

### Still some way to go

Overall I came away thinking that ML in enterprise still has a long way to go, and that we’re still seeing a lot of
gatekeeping in this area.

We have data engineers writing code to deliver the data, data scientists writing models, developers writing apps to turn
the model into a service, and operations deploying it to environments.

No wonder things are slow and complex when we have this many handoffs. If approaches such as MLOps can assist in this
then that's great, but to me much of the deployment issues feel more like business and process problems than technical
or tools based ones.

These are of course my own opinions, and I would welcome to hear your thoughts on MLOps?

#### For more on my AWS Summit series, check out the summaries on the talks I attended below;

[Kubernetes GitOps on AWS - Jason Umiker, AWS Solution Architect]({{site.baseurl}}/2020/05/20/aws-meets-gitops/)

[Enterprise cloud migration meets application containerization - Gaurav Arora, AWS Senior Partner Solutions Architect]({{site.baseurl}}/2020/05/21/enterprise-containerization/)

[CI/CD at scale: Best practices with AWS DevOps services - Loh Yiang Meng, AWS Solution Architect]({{site.baseurl}}/2020/05/22/cicd-at-scale/)

[What was it like to attend a virtual conference?]({{site.baseurl}}/2020/05/24/virtual-conf/)
