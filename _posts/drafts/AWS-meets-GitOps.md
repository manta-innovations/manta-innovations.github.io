---
layout: post
title:  AWS meets GitOps - AWS Online Summit Series
date:   May 2020
image:  
author: jhole89
tags:   cloud, aws, conference
---

Welcome, this is part of my ongoing series on AWS’s recent Online Summit 2020, 
where I write about my thoughts and learnings from the range of topics presented. 
As always, the content here describes my own thoughts and understandings from the material presented, 
not the views of the presenters, who I do not speak for.

### What is GitOps?

As someone who’s spending more and more time with kubernetes but has only dipped my toe into GitOps, I was interested to hear what the AWS approach would be to GitOps.

Therefore, I dialled into *AWS Solution Architect Jason Umiker’s Kubernetes GitOps on AWS*, at the AWS Summit Online, May 2020

This talk did not disappoint. 

### I’ve become a Flux convert.

After covering the basics concepts of CICD we went straight into an overview of Flux, the GitOps operator for Kubernetes and part of the CNCF; and what GitOps actually means to a workflow, mainly being able to control deployments via Pull Requests to your master/release branch.

![post-thumb]({{site.baseurl}}/assets/images/blog/GitOps%20-%20image.png){:class="img-fluid rounded float mr-5 mb-2 ml-5" :height="auto" width="90%"}
<center><sup>Source: Kubernetes GitOps on AWS - Jason Umiker, AWS Solution Architect</center></sup>

### A convincing argument for GitOps

GitOps is a very new approach for release management and deployment, especially to those Enterprise clients, many of whom are still struggling with CICD and remain on traditional timed release cycles.

He highlighted that all developers already use git for many great reasons that apply to not only development of software but release management too; namely a single source of truth, audit trail, built in peer review, and ease in gatewaying change.

By tying the actual release management and deployment to git, we can now have a single tool in control of not only our development and iteration, but also our deployment.

![post-thumb]({{site.baseurl}}/assets/images/blog/GitOps%20-%20cycles.png){:class="img-fluid rounded float mr-5 mb-2 ml-5" :height="auto" width="90%"}
<center><sup>Source: Kubernetes GitOps on AWS - Jason Umiker, AWS Solution Architect</center></sup>

### Ghost in the Machine

Jason went on to explain and demonstrate how GitOps with Flux could be achieved on AWS using AWS CodeBuild and CodePipeline, alongside external kubernetes operators to deploy a change to his Ghost service running on EKS. 

Here he merged a PR that changed the RDS definition which the Ghost app used for storage (an AWS resource managed by the AWS CDK) and a change to his Ghost deployment (a kubernetes resource defined by the manifest). Because he is using GitHub as a source, CodePipeline is able to monitor the repo for changes and initiate a simple pipeline of Source (from git) and CodeBuild only, with the trick being that the CodeBuild stage is actually doing our deployment.

This CodeBuild stage actually has a very simple buildspec.yml that just issues the `cdk deploy` (for those not familiar with AWS CDK this is the equivalent of a `terraform apply`) which applies the change to the RDS resource. At the same time we have Flux monitoring the same repository via a webhook, which has performed a new deployment for the change to the Ghost manifest yaml.

And there we had it, in a single PR he had committed, reviewed, and deployed a change to both the AWS managed infrastructure, and the Kubernetes managed service.

![post-thumb]({{site.baseurl}}/assets/images/blog/GitOps%20-%20flux%20overview.png){:class="img-fluid rounded float mr-5 mb-2 ml-5" :height="auto" width="90%"}
<center><sup>Source: Kubernetes GitOps on AWS - Jason Umiker, AWS Solution Architect</center></sup>


This talk was great and made me realise that I need to spend more time with Flux, especially in light of the Argo Flux collaboration which happened back in November, as this is the exact CICD workflow I’ve always desired.

From a developers point of view being able to finish my tasks with a PR is the ideal. I don’t need to worry if my PR made it into the “Friday release”, or whether there were any issues during deployment, if it’s merged it’s done.
