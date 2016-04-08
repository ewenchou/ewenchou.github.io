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

Once we have the data, we need to parse it and convert it into speech-friendly text for Alexa.

First, we need to send an HTTP GET and parse the JSON data.

<div class="post-note">
<b>Note:</b> I access the Python dictionary keys directly in the code snippets below, which can raise KeyError exceptions if the data is not exactly as expected. Use try/except block(s) as appropriate in your final code.
</div>

{% highlight python %}
import urllib2
import json

# Get the URL for the ticker symbols
url = get_url(['AAPL', 'FB', 'NFLX'])

# Send HTTP GET
res = urllib2.urlopen(url)

# Read the response and convert JSON to Python dict
data = json.loads(res.read())

# Loop through the list of resources and parse the data
resources = data['list']['resources']
stocks = []
for r in resources:
    # Parse the data here

{% endhighlight %}

Inside the for loop, I'll extract the info that I want:

* Name of the company
* Is the stock up/down?
* What's the price?

And convert it into speech-friendly output for Alexa.

The name of the company can contain some extra text that's not needed. Some simple string replacement is done to clean it up:

{% highlight python %}
    name = r['resource']['fields']['name']
    name = name.replace("Common Stock", '')
    name = name.replace(' Inc','').replace(',','').replace('.','')
    name = name.replace('(NS) O','')
    name = name.replace("(The) Commo", "")
    name = name.strip()
{% endhighlight %}

The change percentage is a positive/negative floating point number, but it's returned as a string:

{% highlight python %}
# This is a string
change = r['resource']['fields']['chg_percent']

# Check if postive/negative, i.e. up/down
if "-" in change:
    up_down = "is down"
else:
    up_down = "is up"

# Convert string to decimal number, get the absolute value, and round to one decimal place
change = round(abs(decimal.Decimal(change)), 1)

# Check if there is a non-zero change
if change == 0:
    spoken_change = "is trading at"
else:
    # Avoid saying things like, "two point zero percent"
    splits = str(change).split('.')
    if splits[1] == '0':
        spoken_change = "{up_down} {change}% at".format(
            up_down=up_down, change=splits[0])
    else:
        spoken_change = "{up_down} {change}% at".format(
            up_down=up_down, change=change)
{% endhighlight %}

And finally, parse the price and put everything together:

{% highlight python %}
price = round(decimal.Decimal(r['resource']['fields']['price']), 2)
price = round(decimal.Decimal(price), 2)
spoken_price = "${}".format(price)

# Put all the pieces together
spoken = "{name} {change} {price}.".format(
    name=name, change=spoken_change, price=spoken_price)
{% endhighlight %}

You can find the full code for the skill in my [Github repository](https://github.com/ewenchou/alexa-stock-quotes).
