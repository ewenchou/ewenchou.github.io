---
layout: post
title: ASKing for Stock Quotes
categories: blog
tags: alexa alexa-skills-kit ASK AWS lambda python
---
Another skill I needed for my ["morning greeter"]({{site.baseurl}}/blog/2016/03/25/good-morning-alexa/) program was a way to get stock quotes for multiple companies at a time. Similar to [getting news]({{site.baseurl}}/blog/2016/04/05/asking-for-news/), there are many sources on the internet for getting this information. A popular website for getting stock quotes is [Yahoo Finance](https://finance.yahoo.com) which also happens to provide a webservice the fits my needs exactly. It allows querying multiple ticker symbols in a single request and can return the data in JSON format.

Here's a simple Python function that takes a list of ticker symbol strings and returns the URL for getting the stock quotes as JSON data.

{% highlight python %}
def get_url(ticker_list):
    tickers = ",".join(ticker_list)
    return "http://finance.yahoo.com/webservice/v1/symbols/{tickers}/quote?format=json&view=detail".format(tickers=tickers)
{% endhighlight %}

For example, you can get the URL for Apple, Facebook, and Netflix using the list:

`['AAPL', 'FB', 'NFLX']`

Which will return the URL string: `http://finance.yahoo.com/webservice/v1/symbols/APPL,FB,NFLX/quote?format=json&view=detail`

An HTTP GET request will return a response like this:

{% highlight json %}
{
  "list": {
    "meta": {
      "type": "resource-list",
      "start": 0,
      "count": 2
    },
    "resources": [
      {
        "resource": {
          "classname": "Quote",
          "fields": {
            "change": "1.489998",
            "chg_percent": "1.327747",
            "day_high": "113.809998",
            "day_low": "112.419998",
            "issuer_name": "Facebook, Inc.",
            "issuer_name_lang": "Facebook, Inc.",
            "name": "Facebook, Inc.",
            "price": "113.709999",
            "symbol": "FB",
            "ts": "1459972800",
            "type": "equity",
            "utctime": "2016-04-06T20:00:00+0000",
            "volume": "20814640",
            "year_high": "117.590000",
            "year_low": "72.000000"
          }
        }
      },
      {
        "resource": {
          "classname": "Quote",
          "fields": {
            "change": "-0.110001",
            "chg_percent": "-0.104822",
            "day_high": "106.440002",
            "day_low": "104.250000",
            "issuer_name": "Netflix, Inc.",
            "issuer_name_lang": "Netflix, Inc.",
            "name": "Netflix, Inc.",
            "price": "104.830002",
            "symbol": "NFLX",
            "ts": "1459972800",
            "type": "equity",
            "utctime": "2016-04-06T20:00:00+0000",
            "volume": "9605800",
            "year_high": "133.270000",
            "year_low": "61.184300"
          }
        }
      }
    ]
  }
}
{% endhighlight %}
