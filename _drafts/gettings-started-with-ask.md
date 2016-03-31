---
layout: post
title: All You Need to Do is ASK
categories: blog
tags: alexa alexa-skills-kit ask aws lambda getting-started
---
In my [previous post]({{site.baseurl}}/blog/2016/03/25/good-morning-alexa/), I talked about the things I usually check in the morning on my smartphone and my idea for building a greeter program that would run on a Raspberry Pi and read me the information.

Alexa gets close to what I want, but the information presented to me is not exactly what I'm looking for. So I needed a way to customize her capabilities. Amazon already has a way of doing this via Alexa Skills (think: Apps for Alexa).

> Alexa, the voice service that powers Amazon Echo, provides capabilities, or __skills__, that enable customers to interact with devices in a more intuitive way using voice. Examples of skills include the ability to play music, answer general questions, set an alarm or timer, and more.

Unfortunately, the Skills that were available didn't do exactly what I wanted either.

Fortunately, Amazon has made the Alexa Skills Kit (ASK) available to developers to build their own Skills.

 > The Alexa Skills Kit is a collection of self-service APIs, tools, documentation and code samples that make it fast and easy for you to add skills to Alexa. All of the code runs in the cloud – nothing is on any user device.

This post will walkthrough how to get started with Amazon ASK, setting up an example skill in AWS Lambda and connecting it with our Alexa device.

## Getting Started

To get started with ASK, you will need to have a developer account with Amazon. This is super-easy (especially if you already have an Amazon account for shopping/apps/etc), you just need to visit the [Amazon Developers](https://developer.amazon.com) page and sign-up/activate an account.

Next, click on *Apps & Services* in the navigation menu across the top of the page and then click on *Alexa* in the sub-menu. You will see two "Get Started" sections for *Alexa Skills Kit* and [*Alexa Voice Service*]({{site.baseurl}}/blog/2016/03/20/alexa-voice-service/).

![01]({{site.baseurl}}/images/2016-03/amazon-dev-ask.png)

We'll come back to the ASK page later after we've setup our AWS Lambda function.

## AWS Lambda

Creating an Alexa Skill using Lambda is very easy. First you'll need to create an Amazon Web Services (AWS) account. Again, if you already have an Amazon account this is very straightforward and similar to the Developer Account you created earlier. Just visit the [AWS page](https://aws.amazon.com) and click on *Create an AWS Account*. If you already have an Amazon account, sign-in using your usual credentials. Read through and follow their instructions to get your AWS account setup.

<div class="post-note">
<b>NOTE:</b> Amazon will ask you for billing information. You should read through the details and decide whether AWS is the appropriate solution for you to host your Alexa Skills. You <b><u>do not</u></b> need to use AWS Lambda to create and host Alexa Skills. The getting started guide linked above has information on how to create your own web service/application for Alexa Skills so you can host it anywhere you want.
</div>

Lambda provides a [free tier](https://aws.amazon.com/lambda/pricing/) that is more than adequate for my purposes. I don't expect any of my skills to be executed millions of times per month, nor will they consume a lot of compute time.

Once you have signed into the AWS Console, make sure you have selected the *US East (N. Virginia)* region in the top-right corner of th epage because it is the only one that supports ASK right now.

![Lambda region]({{site.baseurl}}/images/2016-03/lambda-region.png)

On the same page, you'll see all the AWS services listed. Near the top-right of the page you should see the *Compute* services. Click on the *Lambda* link to bring up your list of Lambda functions, and then click the *Create a Lambda function* button to get started.

![Lambda link]({{site.baseurl}}/images/2016-03/lambda-link.png)

### Step 1: Select blueprint

Blueprints are predefined Lambda functions that help you learn how to create your own. There are a few ASK blueprints to choose from and you can type in *alexa* in the *Filter* field to find them. Click the *alexa-skills-kit-color-expert-python* blueprint to select it.

![Lambda blueprint]({{site.baseurl}}/images/2016-03/lambda-blueprint.png)

### Step 2: Configure event sources

For *Event source type* select *Alexa Skills Kit* (it should already be pre-selected for you) and click *Next*.

![Lambda event source]({{site.baseurl}}/images/2016-03/lambda-event-source.png)

### Step 3: Configure function

Here is where you specify the code and some parameters for your Lambda function. First, enter a name for your new function.

![Lambda configure function]({{site.baseurl}}/images/2016-03/lambda-configure-function.png)

Because we selected a blueprint earlier, the code is already populated for us. This is the code that makes up the *Color Expert* skill. It is well documented and was pretty easy for me to figure out what it does and how to make my own skills.

Below the code section, you will need to specify a couple more parameters.

![Lambda role]({{site.baseurl}}/images/2016-03/lambda-role.png)

__Handler__

This is the name of the file plus the entry point for your Lambda function. Together it tells Lambda what to run when your function is invoked. It is already predefined for the blueprint as `lambda_function.lambda_handler`, where the Python file name is `lambda_function.py` and the entry function is `lambda_handler()`.

*Note: You can change the Handler to whatever you want as long as it matches up with your code, but it's easier to stick with these conventions, even when writing your own code and uploading as a zip.*

__Role__

For Alexa Skills, you only need a *Basic execution role*. Selecting it under *Create new role* will allow you to create a reusable role that you can select.

![Lambda execution role]({{site.baseurl}}/images/2016-03/lambda-exec-role.png)

__Advanced Settings__

Now that you've specified the *Handler* and *Role* you can also configure the *Advanced Settings*. The default values should be adequate for the majority of Alexa skills. Once you've reviewed your settings, click *Next* to continue.

<div class="post-note">
  <b>Note:</b> The amount of Memory affects how much you are billed (and uses up your free tier's allotment faster). So it's a good idea to select the lowest memory amount available (128 MB) which is plenty for the majority of Alexa skills which are very simple.
</div>

![Lambda configure function part 2]({{site.baseurl}}/images/2016-03/lambda-configure-function-2.png)

### Step 4: Review

This page is a summary of your new function. If everything looks OK, click the *Create function* button to finish.

![Lambda review]({{site.baseurl}}/images/2016-03/lambda-review.png)

Congratulations! You've created your first AWS Lambda function. Copy the *ARN* string on the top-right of the page. You will need this value to "hook up" your Alexa Skill.

![Lambda ARN]({{site.baseurl}}/images/2016-03/lambda-arn.png)

### Billing Alarm

Although the free tier should be enough for personal use, it's a good idea to setup an Alarm in AWS that will send you an email if/when you start getting charged for usage (just in case).

<div class="post-note">
  <b>Warning:</b> I'm new to AWS so there may be better ways to monitor your billing than what I outline below. This is just what I found and setup for myself.
</div>

Back on the AWS Console home page, there is a section for *Management Tools*. Click on the link for *Cloud Watch*.

![AWS Cloud Watch]({{site.baseurl}}/images/2016-03/aws-cloud-watch.png)

Click the *Create Alarm* button either in the *Alarm Summary* section of the main *Cloud Watch* page or on the *Alarms* page (click *Alarms* on the left menu).

![Cloud Watch Alarms]({{site.baseurl}}/images/2016-03/aws-alarms.png)

Next, you'll be asked to select metrics for your alarm. I kept it simple and selected *Total Estimated Charge* under the *Billing Metrics* category.

![Select metric]({{site.baseurl}}/images/2016-03/aws-billing-metrics.png)

After you click on *Total Estimated Charge*, click on the checkbox to select the metric for USD (there was only one for me) and click the *Next* button.

![Select metric 2]({{site.baseurl}}/images/2016-03/aws-billing-metrics-2.png)

Give your alarm a name and description. Remember to change the setting to `EstimatedCharges > 0`. Mine was set to `>= 0` by default.

In the *Actions* section, select:

* Whenever this alarm: State is ALARM
* Send notifiction to: NotifyMe
* Email list: put your email address

Click the *Create Alarm* button to finish.

![Create alarm]({{site.baseurl}}/images/2016-03/aws-alarms-create.png)


I highly recommend reading through [Getting Started with the Alexa Skills Kit](https://developer.amazon.com/appsandservices/solutions/alexa/alexa-skills-kit/getting-started-guide) to get familiar with the terminology and how things fit together in general.

The rest of this post will talk about how to create a Skill and host it using Amazon's Lambda service. So it's probably a good idea to also read [Developing an Alexa Skill as a Lambda Function](https://developer.amazon.com/appsandservices/solutions/alexa/alexa-skills-kit/docs/developing-an-alexa-skill-as-a-lambda-function).
Click on the one for ASK.
You should see a list of your Skills (empty for now), and some useful links to help you get started.
