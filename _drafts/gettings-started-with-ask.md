---
layout: post
title: Alexa Skills Kit
categories: blog
tags: alexa alexa-skills-kit getting-started
---
In my [previous post]({{ site.baseurl }}/blog/2016/03/25/good-morning-alexa/), I talked about the things I usually check in the morning on my smartphone and my idea for building a greeter program that would run on a Raspberry Pi and read me the information.

Alexa gets close to what I want, but the information presented to me is not exactly what I'm looking for. So I needed a way to customize her capabilities. Amazon already has a way of doing this via Alexa Skills (think: Apps for Alexa).

> Alexa, the voice service that powers Amazon Echo, provides capabilities, or __skills__, that enable customers to interact with devices in a more intuitive way using voice. Examples of skills include the ability to play music, answer general questions, set an alarm or timer, and more.

Unfortunately, the Skills that were available didn't do exactly what I wanted either.

Fortunately, Amazon has made the Alexa Skills Kit (ASK) available to developers to build their own Skills.

 > The Alexa Skills Kit is a collection of self-service APIs, tools, documentation and code samples that make it fast and easy for you to add skills to Alexa. All of the code runs in the cloud – nothing is on any user device.

To get started with ASK, you will need to have a developer account with Amazon. This is super-easy (especially if you already have an Amazon account for shopping/apps/etc), you just need to visit the [Amazon Developers](https://developer.amazon.com) page and sign-up/activate an account.

Next, click on *Apps & Services* in the navigation menu across the top of the page and then click on *Alexa* in the sub-menu. You will see two "Get Started" sections for *Alexa Skills Kit* and [*Alexa Voice Service*]({{ site.baseurl }}/blog/2016/03/20/alexa-voice-service/). Click on the one for ASK.

![01]({{ site.baseurl }}/images/2016-03/ask-01.png)

You should see a list of your Skills (empty for now), and some useful links to help you get started. I highly recommend reading through [Getting Started with the Alexa Skills Kit](https://developer.amazon.com/appsandservices/solutions/alexa/alexa-skills-kit/getting-started-guide) to get familiar with the terminology and how things fit together in general.

The rest of this post will talk about how to create a Skill and host it using Amazon's Lambda service. So it's probably a good idea to also read [Developing an Alexa Skill as a Lambda Function](https://developer.amazon.com/appsandservices/solutions/alexa/alexa-skills-kit/docs/developing-an-alexa-skill-as-a-lambda-function).

Creating an Alexa Skill using Lambda is very easy. First you'll need to create an Amazon Web Services (AWS) account. Again, if you already have an Amazon account this is very straightforward and similar to the Developer Account you created earlier. Just visit the [AWS page](https://aws.amazon.com) and click on *Create an AWS Account*. If you already have an Amazon account, sign-in using your usual credentials. Read through and follow their instructions to get your AWS account setup.

<div class="post-note">
<b>NOTE:</b> Amazon will ask you for billing information. You should read through the details and decide whether AWS is the appropriate solution for you to host your Alexa Skills. You <b><u>do not</u></b> need to use AWS Lambda to create and host Alexa Skills. The getting started guide linked above has information on how to create your own web service/application for Alexa Skills so you can host it anywhere you want.
</div>

Lambda provides a [free tier](https://aws.amazon.com/lambda/pricing/) that is more than adequate for my purposes. I don't expect any of my skills to be executed millions of times per month, nor will they consume a lot of compute time.
