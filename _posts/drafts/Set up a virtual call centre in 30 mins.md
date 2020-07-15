---
layout: post
title: Set up a virtual call centre in 30 minutes
date: 2020-07-14
image: aws-connect/image_10_setting_up_call_centre.png
author: jhole89
tags: serverless, aws, cloud-native, remote teams, digital workplace
---

*This is a step by step guide on how to set up Amazon Connect in under 30 mins*

Amazon Connect enables you to have your own virtual call centre, where agents can log into and receive calls from 
clients via a web portal using only a pair of headphones. If this is the first time you’ve heard of Amazon Connect then I suggest you checkout 
my recent [high level summary](https://manta-innovations.co.uk/2020/06/30/Interested-in-a-virtual-call-centre-Try-AWS-Connect/) on it first.

This demo does require you to already have an AWS account set up, ideally with admin level permissions to provision the required services. 

If you’ve got that then login to the AWS Console and head to the Amazon Connect page and we’ll get started. If not, you will need 
to create an account [here](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Fportal.aws.amazon.com%2Fbilling%2Fsignup%2Fresume&client_id=signup).

#### 1. First you’re going to set up your identity access management. 

If you want to manage your agents within Amazon Connect use the first option *“Store users with Amazon Connect”*, and personalise the URL. 
If you already have and wish to use Active-Directory, you can use the second two options; to manage users via AWS AD, or non-AWS AD via SAML, respectively. 

This stage will also provide you with the URL for your agents to login with.

![post-thumb]({{site.baseurl}}/assets/images/blog/aws-connect/image_1_setting_up_call_centre.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

#### 2. Next you have the option to create an admin user. 

   I suggest skipping this step for this walkthrough as you can use your IAM user instead, 
   however you can take this chance to add other administrators here if you wish too.

#### 3. Next you’'ll configure the telephony options for both inbound and outbound calls. 

   I’ve selected both options here as I want to receive and make calls.

![post-thumb]({{site.baseurl}}/assets/images/blog/aws-connect/image_2_connect_telephony.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

#### 4. The last step of the initial set up is  to configure your data storage; which will contain call and chat logs.

   By default Amazon Connect generates its own S3 buckets and KMS keys to use for secure data encryption, 
   but you can set this to use pre-existing buckets and keys should you wish to.

![post-thumb]({{site.baseurl}}/assets/images/blog/aws-connect/image_3_setting_up_virtual_call_centre.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

#### 5. Now that you have done the initial setup you will be presented with a summary screen.  Check through the options and if everything looks good, create the instance.


![post-thumb]({{site.baseurl}}/assets/images/blog/aws-connect/Image 4_connect_review.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

#### 6. Once your Amazon Connect instance has been created,  you can log into the dashboard and customise your virtual call centre. 

![post-thumb]({{site.baseurl}}/assets/images/blog/aws-connect/image_5_connect_success.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

#### 7. The first thing you need to do is claim a phone number to receive calls on. 

   This can be from any country Amazon Connect supports, regardless of which region our instance is located in. 

   I'm currently in Canada, so I chose a North American number, and opted for ‘Toll free’.

![post-thumb]({{site.baseurl}}/assets/images/blog/aws-connect/image_6_setting_up_call_centre.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

#### 8. Next you will be presented with a screen advising you to claim the number. 

   It advises you to dial the number, however from my experience with Amazon Connect all configuration changes can take up to 15 minutes to be pushed out. 
   If you were to call at this stage you might not get through, but that doesn’t stop you continuing the setup.
   
![post-thumb]({{site.baseurl}}/assets/images/blog/aws-connect/Image 7_claim_number_2.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

#### 9. Next you can set the hours of operation; which is when you expect agents to be able to take calls.

   You can have multiple hours of operation if you want to represent multiple groups or group remote teams by time zones. 
   I set all of my operational hours in Pacific Standard Time, and extended the hours into the evening a little. 
   If your call centre isn’t operational at the weekend, you can remove these from the operational hours.

![post-thumb]({{site.baseurl}}/assets/images/blog/aws-connect/image_8_setting_up_call_centre.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

#### 10. Following this, the next thing you need to do is set up queues. 

   A queue here is not a queue in terms of a waiting queue, but rather a workflow queue that callers will transit through. 

   As with the hours of operation, you can have multiple queues per call centre, for different workflows, 
   and callers can be transferred between queues in the same way you might traditionally transfer callers between departments.

   If your call centre requires more than one workflow, add additional queues with the *“add new queue”* button. I created an additional queue called *“VanQueue”*.

![post-thumb]({{site.baseurl}}/assets/images/blog/aws-connect/image_9_setting_up_call_centre.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

#### 11. Next, you will be given the option to create or upload your own prompts, which are audio files you may wish to playback to callers.

   I didn’t want to use any custom audio prompts, so I skipped this stage but feel to check them out, 
   or add your own and apply them in your contact flow, speaking of which...

#### 12. The next stage is the biggest and most complex bit - contact flows.

   This is how you design the flow that a customer may take, and that can be a complete end to end flow, or a small flow which can be composed into a larger flow. 

   In this way you can use Software Engineering principles of composition and DRY (Don't Repeat Yourself) to create reusable flow elements. 
   As an example, I have created a single end to end flow.
   
 ![post-thumb]({{site.baseurl}}/assets/images/blog/aws-connect/image_10_setting_up_call_centre.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

   Here I've just set up a very simple flow whereby I check those basic settings 
   I've configured (opening hours, staff availability, and queue availability) and try to transfer the customer to an agent. 

   If any of these fail, the system responds to the customer letting them know why (e.g. outside of opening hours) prior to terminating the call. 
   If they can't be immediately transferred due to the queue being at capacity, I've implemented a loop to wait 5 minutes and try again.

   In this way I've been able to set up a very simple complete end to end flow for a call center, using a simple drag and drop UI. 
   Flows can become a lot more complex, and I could have used things such as keypad entry, Lex skills, and even trigger an AWS Lambda 
   (which in turn can be used to trigger many other AWS services via an SDK call).

#### 13. Following this, you will need to set up a routing profile. 

Routing profiles act as the link between our agents and answer queues.

![post-thumb]({{site.baseurl}}/assets/images/blog/aws-connect/routing_profile_1.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

![post-thumb]({{site.baseurl}}/assets/images/blog/aws-connect/routing_profile_2.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

#### 14. Once the routing profile is in place, you can now start creating users and assigning them to the profile.

![post-thumb]({{site.baseurl}}/assets/images/blog/aws-connect/image_13_add_user.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

   When creating an agent/user you need to assign them both a routing profile (which we just spoke about above) and a security profile. 
   Security profiles dictate the access control the agent has within AWS Connect, and can be selected from default options of *Admin, 
   Agent, CallCenterManager, or QualityAnalyst*.

   Alternatively you can create our own Security Profiles and assign agents to them.

#### 15. The last thing you need to do is to switch your inbound number onto the correct contact flow. 

   The reason to do this last is to ensure that everything related to that contact flow is set up and agents are available before making the flow live. 
   If you switched the number onto the flow at the start, but hadn’t yet created agents to answer, or the correct operational hours, 
   then clients may start calling in and receiving unexpected responses or be left waiting for an agent.

   We do this simply by going back to the phone number management screen and attaching it to our new contact flow.

   For instance, I switched it from ‘Sample inbound flow’ to ‘Call centre’ which is the name I gave my demo contact flow.
   
![post-thumb]({{site.baseurl}}/assets/images/blog/aws-connect/image_14_setting_up_call_centre.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

#### 16. Once that’s done, you are ready to go. 

   You have successfully set up a virtual call centre in (hopefully) under 30 minutes. Clients can now dial in, 
   and after making their way through our contact flow they’ll be passed to an available agent.

   You’ll be able to log onto your virtual call centre by clicking the phone logo in the top right.

![post-thumb]({{site.baseurl}}/assets/images/blog/aws-connect/image_17_user_loging.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

![post-thumb]({{site.baseurl}}/assets/images/blog/aws-connect/image_18_setting_up_call_centre.png){:class="img-fluid rounded float mx-auto mb-2" :height="auto" width="60%"}

This was just a simple quick walkthrough of setting up Amazon Connect.

Amazon Connect is a powerful tool and it can become complex when we start using some of the more interesting features such as AWS Lex and Lambda support.

If you find yourself in need of some advice or just want to find out more, then feel free to reach out to me on Twitter ([@joellutman](https://twitter.com/joellutman)), 
email [joel@manta-innovations.co.uk](mailto:joel@manta-innovations.co.uk) or via my [site](http://manta-innovations.co.uk/). 









