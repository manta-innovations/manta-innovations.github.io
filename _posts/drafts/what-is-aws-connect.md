---
layout: post
title: Interested in a virtual call centre? Try AWS connect 
date: 2020-06-24
image: Aws connect logo_Connect-2.jpg
author: jhole89
tags: serverless, aws, cloud-native, remote teams, digital workplace
---

Recently I found myself spending some time with some of the less well known AWS services, and I wanted to draw attention to just how great some of these services are. 

One of them, AWS Connect, has proven to be an interesting use case. 

With the growing demand to work remotely, it has seen increased usage during the COVID-19 outbreak. It allows companies to create a virtual cloud based call centre, that enables and empowers staff to answer from anywhere they have access to a PC.

## A cloud based call centre?

AWS Connect markets itself as *“an omnichannel cloud contact center”*, but what does that really mean?

AWS Connect is a versatile way of building and managing a completely serverless call centre, and allows distributed teams to work remotely from anywhere over the world.

It can be used as a simple way of managing agents and connecting customers with them, or as a way of building complex routing systems that can use multiple customer inputs and diverging paths to route customers to different agent teams.

![post-thumb]({{site.baseurl}}/assets/images/blog/aws-connect-overview/Woman-wearing-earpiece-using-white-laptop-computer-210647.jpg){:class="img-fluid rounded float-right ml-5" :height="auto" width="50%"}
<center><sup>Source: https://www.pexels.com/photo/woman-wearing-earpiece-using-white-laptop-computer-210647/</sup></center>

What separates AWS Connect from a traditional call centre is its ability to create and scale call centres within minutes, and enables remote working as it relies on web interfaces rather than a traditional handset. 

AWS Connect provides an all in one service for:
- acquiring public phone numbers
- creating distributed teams
- creating operational hours 
- creating simple to complex call routing
- secure data storage and encryption of call logs on AWS
- Integration with AWS database for automatic logs and stats
  providing a simple user interface for agents to answer calls without needing a physical handset

## Just how simple is it?

AWS Connect is one of those well designed products that can be downright simple, or incredibly complex, depending on what you design and the components that you use. 

It is designed to be used by anyone, and doesn’t require developer experience to configure, though some knowledge of S3 buckets and data encryption via AWS KMS is beneficial.

![post-thumb]({{site.baseurl}}/assets/images/blog/aws-connect-overview/petr-machacek-BeVGrXEktIk-unsplash.jpg){:class="img-fluid rounded float-right ml-5" :height="auto" width="50%"}
<center><sup>Source: https://unsplash.com/photos/BeVGrXEktIk</sup></center>

Without any prior experience, on my first attempt with AWS Connect I was able to get a full serverless call centre up and running in under 15 minutes, that included:

- A public dial in number
- Secure encrypted data storage for call logs
- Seniority roles for managers and agents with different admin rights
- Reports on call metrics and stats
- Operational hours for agents
- A call routing that made use of keypad entry, and queue checking to place customers on hold if no agent was available
- 3 different user types (admin, managerial, agent) that could log in and receive inbound calls via PC and headset

## Complexity if you want it

On the other side, AWS Connect supports a huge range of customisation and supported services. 

AWS Connect can be integrated into AWS Lambda and AWS Lex, meaning scripts can be written that would enable some of the following features:

- **Speech-to-text translation** - providing agents with a summary of call
- **Integration with AWS database solutions** - providing queryable stats and metrics of calls
- **Language detection** - allowing key words and phrases to be identified and flagged during calls to help understand overall customer satisfaction

![post-thumb]({{site.baseurl}}/assets/images/blog/aws-connect-overview/chatbot.jpg){:class="img-fluid rounded float-right ml-5" :height="auto" width="50%"}
<center><sup>Source:http://anthillonline.com/wp-content/uploads/2018/07/chatbot.jpg</sup></center>

AWS Connect is an incredibly versatile and scalable platform that allows companies to build a customised and flexible virtual call centre. Enabling them to adapt to pressures of scale, flexibility, and distribution to overcome the obstacles and rigid structures of a traditional call centre.
 
This is just a high level summary of AWS connect, I’ll be looking to put together a more technical guide in the foreseeable future - so watch this space. 

More for information about AWS and Serverless feel free to check out my other [blogs](https://manta-innovations.co.uk/blog). 

