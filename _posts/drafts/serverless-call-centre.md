---
layout: post
title: Creating a Virtual Call Centre
date: 2020-04-04
image:
author: jhole89
tags: serverless, aws, cloud-native, remote teams, digital workplace
---

Today we're going to be setting up AWS Connect in under 20 minutes. This will enable you to have your own virtual call
centre, where your agents can log into and receive calls via their PC using only a pair of headphones. This demo does
require you to already have an AWS account set up, ideally with Admin level permissions to provision the required services.

First we're going to set up our identity management. If you want to manage your users, by which we mean our agents,
just in Connect, we use this first option; otherwise we can use the next two to manage our users via AWS AD, or non-AWS
Active Directory via SAML, respectively. This will also provide us with the url for our agents to login to.

![]({{site.baseurl}}/assets/images/blog/aws-connect/connect_id.png){:class="img-fluid rounded float" :height="auto" width="60%"}

Next we have the option to create an admin user; we're just going to skip as we can us my current IAM user instead; and
we'll configure the telephony options for both inbound and outbound calls too.

![]({{site.baseurl}}/assets/images/blog/aws-connect/connect_telephony.png){:class="img-fluid rounded float" :height="auto" width="60%"}

Lastly we're going to configure our data storage, which will contain our call and chat logs. By default Amazon Connect
has generated its own s3 buckets and kms key to use, but we can set this to use pre-existing buckets and keys should we
wish to.

![]({{site.baseurl}}/assets/images/blog/aws-connect/connect_storage.png){:class="img-fluid rounded float" :height="auto" width="60%"}

And now we just have a summary screen, so we can check through these options and if everything looks good we can
provision the instance. Once we have our instance, we'll log into the dashboard and follow the list like a tick box, and
that will provide us with everything needed to get our virtual call centre configured and running.

| --- | --- |
| ![]({{site.baseurl}}/assets/images/blog/aws-connect/connect_review.png){:display="inline" height="auto" width="100%"} | ![]({{site.baseurl}}/assets/images/blog/aws-connect/connect_success.png){:display="inline" height="auto" width="100%"} |

The first thing need to do is claim a phone number to take calls on, and we can register this in any country from the
list, regardless of which region our instance is located in. I'm currently in Vancouver, so I'll chose a Canadian number.
While we now have that number, from my experience with Amazon Connect all configuration changes can take up to 15 minutes
to be pushed out, so if we were to call this right now we might not get through.

| --- |
| ![]({{site.baseurl}}/assets/images/blog/aws-connect/claim_number.png){:display="inline" height="auto" width="100%"} | ![]({{site.baseurl}}/assets/images/blog/aws-connect/claim_number_2.png){:display="inline" height="auto" width="100%"} |

Next we're going to set our hours of operation, which is when we expect our agents to be able to take calls. We can have
multiple hours of operation if we wanted to represent multiple groups or group remote teams by timezones. I'll set all of
our times in Pacific Standard Time, and I'm going to extend our hours into the evening a little. After that the next
thing we need to do is setup our queues. A queue here is not a queue in terms of a waiting queue, but rather a workflow
queue that callers will transit through. As with the hours of operation, we can have multiple queues per call centre,
for different workflows, and we can transfer our callers between queues in the same way you might traditionally transfer
callers between departments.

| --- |
| ![]({{site.baseurl}}/assets/images/blog/aws-connect/hours_operation.png){:display="inline" height="auto" width="100%"} | ![]({{site.baseurl}}/assets/images/blog/aws-connect/vancouver_queue.png){:display="inline" height="auto" width="100%"} |

Next, we could create or upload our own prompts, which are audio files we may wish to playback to callers. I'm not going
to be using any audio prompts so will skip past this. Now we get to the biggest and most complex bit - contact flows.
This is how you design the flow that a customer may take, and that can be a complete end to end flow, or what im going
to call a sub flow, which can be composed into a larger flow - in this way we can used Software Engineering principles
of composition and DRY (Don't Repeat Yourself) to create reusable flow elements. For now I'm going to create a single
end to end flow.

![]({{site.baseurl}}/assets/images/blog/aws-connect/flow_example.png){:class="img-fluid rounded float" :height="auto" width="60%"}

Here we've just set up a very simple flow whereby we check those basic settings we've configured (opening hours, staff
availability, and queue availability) and try to transfer the customer to an agent. If any of these fail we respond to
the customer letting them know why (e.g. outside of opening hours) prior to terminating the call, and if they can't be
immediately transferred due to the queue being at capacity, we've implemented a loop to wait 5 minutes and try again. In
this way we've been able to set up a very simple complete end to end flow for a call center, using a simple drag and drop
UI. Connect flows can become a lot more complex, and we could have used things such as keypad entry, Lex skills, and even
trigger an AWS Lambda (which in turn can be used to trigger many other AWS services via an SDK call).
