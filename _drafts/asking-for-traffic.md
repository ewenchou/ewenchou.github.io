---
layout: post
title: ASKing for Traffic
categories: blog
tags: alexa alexa-skills-kit ASK AWS lambda python
---
Now that we have ways to get the [weather]({{site.baseurl}}/blog/2016/03/25/good-morning-alexa#weather-report), [news]({{site.baseurl}}/blog/2016/04/05/asking-for-news/), and [stock quotes]({{site.baseurl}}/blog/2016/04/08/asking-for-stocks/), the last piece of information needed for my ["morning greeter"]({{site.baseurl}}/blog/2016/03/25/good-morning-alexa) program is the [traffic]({{site.baseurl}}/blog/2016/03/25/good-morning-alexa#traffic-conditions).

There are many ways to get traffic conditions but one of the most popular (if not *the* most popular) ways is [Google Maps](https://maps.google.com). A quick web search revealed that it provides an [API](https://developers.google.com/maps/) with various [price plans](https://developers.google.com/maps/pricing-and-plans/) depending on the platform.

For this project, I will be using the Google Maps Directions API webservice, which at the time of writing has the Standard pricing model below:

> Free up to 2,500 requests per day $0.50 USD / 1,000 additional requests, up to 100,000 daily

The free limit of 2,500 requests per day is more than enough for my personal use.

To get started, you need to sign in with your Google account and obtain an API key. More info on how to do this can be found on Google's [documentation page](https://developers.google.com/maps/documentation/directions/)

Once you have an API key, it's easy to use Google's [Python client library for Google Maps API Web Services](https://github.com/googlemaps/google-maps-services-python) to query for directions which will provide the estimated time in traffic as part of the response.

{% highlight python %}
import googlemaps
import datetime

GMAPS_API_KEY = "your-secret-api-key"

# Lookup the start and dest addresses
start_addr = "your-starting-address"
dest_addr = "your-destination-address"

# Use Google Maps API client to get directions
gmaps = googlemaps.Client(key=GMAPS_API_KEY)
departure_time = datetime.datetime.now()

# Default directions
res_1 = gmaps.directions(
    start_addr, dest_addr, departure_time=departure_time)

# Alternate directions avoiding highways
res_2 = gmaps.directions(start_addr, dest_addr,
    departure_time=departure_time, avoid=['highways'])

# Get the traffic time in minutes for both routes
mins_1 = res_1[0]['legs'][0]['duration_in_traffic']['value'] / 60
mins_2 = res_2[0]['legs'][0]['duration_in_traffic']['value'] / 60
{% endhighlight %}

Now I needed to adapt the above code for use with an Alexa skill.

First, I wrote down how I wanted to interact with this skill (which got expanded into sample utterances later).

* How's traffic to work?
* How's traffic to work for Bob?
* How's traffic to school from home?

Next, I designed the intent schema for my skill.

{% highlight json %}
{
  "intents": [
    {
      "intent": "GetTrafficIntent",
      "slots": [
        {
          "name": "Person",
          "type": "AMAZON.US_FIRST_NAME"
        },
        {
          "name": "Start",
          "type": "LIST_OF_PLACES"
        },
        {
          "name": "Destination",
          "type": "LIST_OF_PLACES"
        }
      ]
    },
    {
      "intent": "AMAZON.HelpIntent"
    },
    {
      "intent": "AMAZON.CancelIntent"
    }
  ]
}
{% endhighlight %}

I only needed one `GetTrafficIntent` intent but with multiple slots.

* `Person`: This is the name of the person for which to get traffic times (optional, with default)
* `Start`: This is the name of the starting place (optional)
* `Destination`: This is the name of the destination place (required)

{% highlight python %}
# Dictionary mapping names of people to their addresses.
# Customize with your own values.
PEOPLE_AND_PLACES = {
    "Ewen": {
        "work": {
            "address": "3 W. Plumeria Dr, San Jose, CA 95131",
            "start": "home"
        },
        "home": {
            "address": "4451 Cherico Ln, Dublin, CA 94568",
            "start": "work"
        },
        "school": {
            "address": "11900 Silvergate Dr, Dublin, CA 94568",
            "start": "home"
        }
    },
    "Venus": {
        "work": {
            "address": "2527 Camino Ramon, San Ramon, CA 94583",
            "start": "home"
        },
        "home": {
            "address": "4451 Cherico Ln, Dublin, CA 94568",
            "start": "work"
        },
        "school": {
            "address": "11900 Silvergate Dr, Dublin, CA 94568",
            "start": "work"
        }
    }
}

# Default if skill is launched without specifying a person
DEFAULT_PERSON = "Ewen"
{% endhighlight %}
