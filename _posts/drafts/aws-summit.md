---
layout: post
title:  Best practice for AWS DevOps Services
date:   May 2020
image:  
author: jhole89
tags:   cloud, aws, conference
---
While I have my own opinion of best practice, I think it’s good to constantly check your standards against peers and industry leaders to ensure you haven’t fallen behind.

Therefore I decided to dial into *AWS Solution Architect Loh Yiang Meng’s talk: CI/CD at scale: Best practices with AWS DevOps services* at the AWS Summit Online 2020

Overall I felt that this talk was best pitched for those unfamiliar with the AWS CICD tools, as he gave a good overview of the AWS Developer tools (Code Commit/Build/Deploy/Pipeline), and how these integrate with each other. For more info on these check out the docs on [AWS CodePipeline](https://aws.amazon.com/codepipeline/).

### Codepipeline now supports integration with Bitbucket Cloud

One thing that he made a point of highlighting is that CodePipeline now supports integration with Bitbucket Cloud (I believe this went into Beta last December), which leaves GitLab as the only major git provider not supported. 

While I’ve used GitLab extensively in enterprise environments (and much prefer the experience over Bitbucket or CodeCommit), between this and all the great stuff GitHub is doing recently with Codespaces and Actions, I really can’t see any reason to not be using GitHub in 2020.

### Electrify’s Journey with AWS CICD

Lastly, Loh introduced Martin Lim, CEO, and Arshad Zackeriya, Senior DevOps Engineer, from Electrify Asia to talk about their CICD journey with AWS. 

