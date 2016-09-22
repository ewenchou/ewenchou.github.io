---
layout: post
title: Bye-Bye Yahoo (... Finance API)
categories: blog
tags: alexa alexa-skills-kit ASK AWS lambda python googlefinance
---
Thanks to [killerbutterfly2](https://github.com/killerbutterfly2) on Github, I found out that the Yahoo Finance API endpoint that I used when I wrote [ASKing for Stock Quotes]({{ site.base_url }}/blog/2016/04/08/asking-for-stocks/) has gone away. Which (of course) meant that my Alexa Skill stopped working as well.

After combing through Internet search results (all of them were out-of-date) for longer than I care to admit, I finally gave up trying to find an alternative API that was free and easy-to-use.

Then it dawned on me that there are tons of developers out there smarter than me (duh, Ewen). So instead of searching Google, I started searching Github and found the [`googlefinance`](https://github.com/hongtaocai/googlefinance) Python module. It's really easy to use and actually makes the Alexa Skill way simpler. 

I've updated my Alexa Skill [repo on Github](https://github.com/ewenchou/alexa-stock-quotes). Please feel free to check it out.