---
layout: post
title:  Best practice for AWS DevOps Services
date:   May 2020
image:  
author: jhole89
tags:   cloud, aws, conference
---
While I have my own opinion of best practice, I think it’s good to constantly check your standards against peers and industry leaders to ensure you haven’t fallen behind.

Therefore I decided to dial into *AWS Solution Architect Loh Yiang Meng’s talk: CI/CD at scale: Best practices with AWS DevOps services* at the AWS Summit Online in May 2020

Overall I felt that this talk was best pitched for those unfamiliar with the AWS CICD tools, as he gave a good overview of the AWS Developer tools (Code Commit/Build/Deploy/Pipeline), and how these integrate with each other. For more info on these check out the docs on [AWS CodePipeline](https://aws.amazon.com/codepipeline/).

### Codepipeline now supports integration with Bitbucket Cloud

![post-thumb]({{site.baseurl}}/assets/images/blog/Bitbucket.png){:class="img-fluid rounded float mr-5 mb-2 ml-5" :height="auto" width="90%"}
<center><sup>Source: CI/CD at scale: Best practices with AWS DevOps services - Loh Yiang Meng, AWS Solution Architect</sup></center>

One thing that he made a point of highlighting is that CodePipeline now supports integration with Bitbucket Cloud (I believe this went into Beta last December), which leaves GitLab as the only major git provider not supported. 

While I’ve used GitLab extensively in enterprise environments (and much prefer the experience over Bitbucket or CodeCommit), between this and all the great stuff GitHub is doing recently with Codespaces and Actions, I really can’t see any reason to not be using GitHub in 2020.

### Electrify’s Journey with AWS CICD

Lastly, Loh introduced Martin Lim, CEO, and Arshad Zackeriya, Senior DevOps Engineer, from Electrify Asia to talk about their CICD journey with AWS. 

![post-thumb]({{site.baseurl}}/assets/images/blog/AWS%20CLOUD.png){:class="img-fluid rounded float mr-5 mb-2 ml-5" :height="auto" width="90%"}
<center><sup>Source: CI/CD at scale: Best practices with AWS DevOps services - Loh Yiang Meng, AWS Solution Architect</sup></center>

Here they gave us an overview of their CICD pipeline, which followed Loh’s use of CodeCommit, CodeBuild, ECR, and CodePipeline for best practice CI. However they used a Lambda to deploy to their EKS cluster (deployment to EKS is something that CodeDeploy has yet to support), and then went further and built an Alexa skill to trigger deployments. 

While their design of sourcing (CodeCommit), building (CodeBuild), publishing (ECR), and orchestration (CodePipeline), followed best practice CI, and the Alexa skill definitely had the wow factor, this still involved some manual intervention to trigger deployments. Sure the Alexa skill made deployments easier, but is it really any different from someone clicking “run” on a jenkins job? 

I’m also not sure I’d trust Alexa with doing my production deployments - what happens if a colleague said the wrong release number?
![post-thumb]({{site.baseurl}}/assets/images/blog/Twitter%20alexa.png){:class="img-fluid rounded float-right mr-5" :height="auto" width="50%"}
<center><sup>Source: boredpanada.com </sup></center>

### “DevOps is not a product, but a culture”

That said Loh Yiang Meng was very engaging as a presenter and some of his comments on best practice definitely aligned with my own. In particular he highlighted that we should automate everything because humans make mistakes; that 

![post-thumb]({{site.baseurl}}/assets/images/blog/DevOPS.png){:class="img-fluid rounded float mr-5 mb-2 ml-5" :height="auto" width="90%"}
<center><sup>Source: CI/CD at scale: Best practices with AWS DevOps services - Loh Yiang Meng, AWS Solution Architect</sup></center>



