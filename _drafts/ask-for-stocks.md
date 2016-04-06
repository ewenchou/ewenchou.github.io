---
layout: post
title: ASKing for Stock Quotes
categories: blog
tags: alexa alexa-skills-kit ASK AWS lambda python
---
Another skill I needed for my ["morning greeter"]({{site.baseurl}}/blog/2016/03/25/good-morning-alexa/) program was a way to get stock quotes for multiple companies at a time. Similar to [getting news]({{site.baseurl}}/blog/2016/04/05/asking-for-news/), there are many online resources from which to get this information. One popular resource is [Yahoo Finance](https://finance.yahoo.com) which also provides a webservice API which conveniently allows querying multiple ticker symbols in a single request and can return the data in JSON format.

{% highlight python %}
def get_url(ticker_list):
    tickers = ",".join(ticker_list)
    return "http://finance.yahoo.com/webservice/v1/symbols/{tickers}/" \
           "quote?format=json&view=detail".format(tickers=tickers)
{% endhighlight %}
