---
layout: post
title: ASKing for Traffic
categories: blog
tags: google-maps alexa alexa-skills-kit ASK AWS lambda python
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

Next, I designed the intent schema for my skill. I used one intent with multiple slots:

* `Person`: This is the name of the person for which to get traffic times (optional, with a default value)
* `Start`: This is the name of the starting place (optional)
* `Destination`: This is the name of the destination place (required)

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

The `Person` slot uses the built-in `AMAZON.US_FIRST_NAME` slot type. You can optionally extend these built-in slot types with your own values (useful for people, like me, who have uncommon names).

The `Start` and `Destination` slots use a custom slot type `LIST_OF_PLACES` which is just a list of names for the predefined places the skill will support.

Both of these slots can be configured in the *Interaction Model* section of your Alexa skill in Amazon's developer portal.

<div class="post-note">
<b>Psst...</b> In case you missed it, I wrote a <a href="/blog/2016/03/31/all-you-need-to-do-is-ask/">blog post</a> that walks through how to create an Alexa Skill using the developer portal and AWS Lambda.
</div>

The skill will have the people and places predefined so that you can say the name of the place instead of the address.

Here's the sample data as an example:

{% highlight python %}
# Dictionary mapping names of people to their addresses.
# Customize with your own values.
PEOPLE_AND_PLACES = {
    "Mark": {
        "work": {
            "address": "1 Hacker Way, Menlo Park, CA 94025",
            "start": "home"
        },
        "home": {
            "address": "3660 21st St, San Francisco, CA 94114",
            "start": "work"
        }
    },
    "Tim": {
        "work": {
            "address": "1 Infinite Loop, Cupertino, CA 95014",
            "start": "home"
        },
        "home": {
            "address": "Webster St, Palo Alto, CA ",
            "start": "work"
        },
        "cafe": {
             "address": "10591 N De Anza Blvd, Cupertino, CA 95014",
             "start": "work"
        }
    }
}

# Default if skill is launched without specifying a person
DEFAULT_PERSON = "Mark"
{% endhighlight %}

The `PEOPLE_AND_PLACES` dictionary maps the names of people to the names of places. Each place has an address and the name of its default starting place. I've also defined the `DEFAULT_PERSON` to use if the skill does not receive a value in the intent.

This allows different combinations of spoken commands (utterances) like the following:

```
How's the traffic to {Destination}
How's the traffic to {Destination} for {Person}
How's the traffic to {Destination} from {Start}
How's the traffic to {Destination} from {Start} for {Person}
```

The rest of the code involves extracting the value(s) from the slots and setting up some speech-friendly output which I will not cover in detail here. But, the code for the full skill can be found in my [Github](https://github.com/ewenchou/alexa-traffic-time) and there are comments throughout and is quite easy to understand. Feel free to take it for a test drive :)

<iframe width="560" height="315" src="https://www.youtube.com/embed/6s4xeIsxv4o" frameborder="0" allowfullscreen></iframe>
