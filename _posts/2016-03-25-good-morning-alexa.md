---
layout: post
title: Good Morning, Alexa
categories: blog
tags: alexa
---
I included some [example code](https://github.com/ewenchou/alexa-tts-demo) in a [previous post]({{ site.baseurl }}/blog/2016/03/21/chatting-with-alexa/) to demonstrate how to interact with Alexa programmatically. This gave me the building blocks needed to build one of my ideas, a program that will greet me in the morning with useful information.

So the first step was to list the things that I usually check on my smartphone when I wake up.

* [Weather report](#weather-report)
* [News headlines](#news-headlines)
* [Stock quotes](#stock-quotes)
* [Traffic conditions](#traffic-conditions)

## Weather Report

Checking weather was easy, I only needed to ask Alexa,

> "How's the weather?"

By default, it will return the weather for Seattle (clever, as Amazon's HQ is in Seattle). But you can easily add the place you want. For example,

> "How's the weather in San Jose, California?"

Would return the weather for San Jose.

## News Headlines

Asking for news didn't work out-of-the-box. Asking Alexa,

> "What's new?"

or

> "What's in the news?"

Returned a cryptic response,

> "In NPR news from TuneIn"

After Googling around, it turns out this was because of the default setting for Alexa's "Flash Briefing" which can be configured under the "Settings" menu in the Amazon Alexa mobile app or [web app](http://alexa.amazon.com).

![Alexa App Settings]({{ site.baseurl }}/images/alexa-app-settings.png)

![Alexa App Flash Briefing]({{ site.baseurl }}/images/alexa-app-flash-briefing.png)

![Alexa App NPR News]({{ site.baseurl }}/images/alexa-app-npr-news.png)

I turned off NPR News because I won't be streaming the audio to my device.

*Note: I'm not sure if this is possible with AVS. There was a section in the developer portal that asks whether the AVS device will use Amazon Music and the device's streaming capabilities. But I didn't try it.*

There is a "News Headlines" section further down that allows turning on different  categories for text-to-speech news.

![Alexa App NPR News]({{ site.baseurl }}/images/alexa-app-news-headlines.png)

Now when I ask Alexa, "what's new?" she will read me *a single headline* from the first category I chose (i.e. U.S. news). According to [Amazon's help page](https://www.amazon.com/gp/help/customer/display.html?nodeId=201601880) I'm supposed to be able to say, "Next" to get the next headline but this didn't work for me. Alexa will just say,

> "Sorry, I can't skip ahead right now."

Maybe this works for the real Amazon Echo, but I was not able to get it to work with my Raspberry Pi. So getting news headlines (that I want) seems like it will require some extra work.

## Stock Quotes

Next, I wanted to be able to lookup stock prices. So I asked Alexa,

> "What's the stock price of Apple?"

which returned the response,

> "To lookup stock prices, enable a stock skill like Fidelity in the Alexa app. Then use the skill's phrases in the description."

OK, fair enough. So I headed to the "Skills" menu in the Alexa app and searched for Fidelity and enabled the skill. I was able to ask for different stock quotes but the response wasn't quite to my liking. They were very verbose, and I didn't want it to keep prompting me for what I want next. It makes sense for the Amazon Echo, but for the project I'm building I don't want to give verbal responses back.

What I want is to be able to provide a list of companies and get their quotes back all at once. So it looks like stock quotes were also going to need more work.

## Traffic Conditions

Getting traffic also required some additional setup in the Alexa app. Navigating to `Settings > Traffic`, you can set a start address and a destination. It was smart enough to have pre-filled the start address with my home address (from my Amazon account) but it needed me to set a destination. After doing so, I was able to ask Alexa,

> "How's traffic?"

And she responded with the approximate travel time and recommended route. It also allowed me to add multiple stops along my route. Pretty good!

However, what it didn't let me to do was setup separate destinations. For example, I may want to know the traffic to work, or I may want to know the traffic to my daughter's school. Or maybe my wife wants to ask for traffic to her workplace instead of mine.

Having to open the Alexa app and changing the destination address every time is too much of a hassle. At that point, I may as well just open the Maps app on my smartphone.

So again, looks like getting the traffic information I want is going to take more work.

## Coming Soon

Stay tuned for my next posts where I will go through the solutions I came up with for each of my morning routine's requirements.

Thanks again for reading.
