---
layout: post
title: Enterprise & Containerization - AWS Online Summit Series
date:   May 2020
image:  
author: jhole89
tags:   cloud, aws, conference
---

Welcome, this is part of my ongoing series on AWS’s recent Online Summit 2020, 
where I write about my thoughts and learnings from the range of topics presented. 
As always, the content here describes my own thoughts and understandings from the material presented, 
not the views of the presenters, who I do not speak for.

### What’s hard about containers?

I tend to work with a lot of enterprise clients and, as much as I believe in and desire modern workflows with Kubernetes, Flux, and GitOps; my experience has been that many enterprise clients are still stuck in the traditional delivery format and have a tentative understanding of containerization and microservices.

I believe this is due to the business concept of an “application” being easier to comprehend as a single monolithic codebase rather than a set of loosely coupled microservices. So I was interested to hear *AWS Senior Partner Solutions Architect Gaurav Arora’s*thoughts on how we as technologists and consultants deal with that, and dialled into his talk on *Enterprise cloud migration meets application containerization*.

Gaurav presented his approach to containerization of enterprise applications using a plan of:

- Prepare
- Discover
- Design
- Migrate
- Operation

### Prepare

The Prepare stage it’s all about understanding the enterprise viewpoint, and how to prepare them for containerization.

He spoke of his experience with enterprise clients and how while many of them may have heard of containerization and potentially even kubernetes, some are still in the dark as to why these would benefit them.

Those that were aware of the benefits cited such as “increase agility”, “productivity”, “cost optimisation”, and this is exactly the arguments we should hone in on when evangelising containerization for enterprise.

### Discover

Gaurav next spoke about the Discovery stage, and how when looking at pre-existing enterprise applications we need to assess what elements can be containerized.

Do we need binaries? How does the licence work inside a container? Do we bundle our own dependencies? Is it stateless? Can it be containerized?

This is something that I hadn’t appreciated. For so long I’ve been able to run any app inside a docker container and build my applications with docker in mind.

I’d forgotten that so much enterprise software was reliant on obscure versions of specific software that might be entirely closed source, what about if an application is only built to run on some Windows platform - how do I containerize that?!?!

![post-thumb]({{site.baseurl}}/assets/images/blog/Docker.png){:class="img-fluid rounded float mr-5 mb-2 ml-5" :height="auto" width="90%"}
<center><sup>Source: Enterprise cloud migration meets application containerization - Gaurav Arora, AWS Senior Partner Solutions Architect<center><sup>

### Design, Migrate & Operation

Gaurav talked about how for both Design and Migration, enterprise should consider a three tier/stage approach.

1. Stage one - would be the design and creation of the lowest level of the cloud, that of VPC’s, security groups, accounts, and tagging.

2. Stage two - would be the design and creation of the cluster environment, would you use ECS or Fargate, what about kubernetes, do you use ECR, what about load balancers. 

3. Stage three - would be the actual container architecture, which base image do you use, how many replicas should we run?

![post-thumb]({{site.baseurl}}/assets/images/blog/Containers.png){:class="img-fluid rounded float mr-5 mb-2 ml-5" :height="auto" width="90%"}
<center><sup>Source: Enterprise cloud migration meets application containerization - Gaurav Arora, AWS Senior Partner Solutions Architect<center><sup>

Overall, although I completely agree with Gaurav and understand how recent containerization is in the eyes of some enterprise business, more than anything I was left a little disheartened.

The fact that we are still talking about how we containerize enterprise applications shows how many applications there are out there that are still waiting to get, or just can’t be containerized.
