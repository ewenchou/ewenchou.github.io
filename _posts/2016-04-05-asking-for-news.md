---
layout: post
title: ASKing for News
categories: blog
tags: alexa alexa-skills-kit ASK AWS lambda python
---
I wrote about the types of things I check in the morning in an [earlier post]({{site.baseurl}}/blog/2016/03/25/good-morning-alexa), and discussed how Alexa comes close to getting the info I want but ultimately fell short. However, Amazon does provide all the tools needed to [write your own Alexa Skills]({{site.baseurl}}/blog/2016/03/31/all-you-need-to-do-is-ask/) which allowed me to develop Skills that are customized to my liking.

In this post, I'll go over the skill I wrote to get the news headlines for my "morning greeter" program.

## News Headlines

I wanted my program to read me a few of the top headlines in the morning. Many news sources on the web provide RSS feeds so this was an obvious place to start. I looked at a few of them and found that the BBC News feed provided the simplest format, and the data was in plain text (many other sources had embedded HTML tags).

Parsing the feed for the headlines was very simple thanks to the [feedparser](https://pypi.python.org/pypi/feedparser) Python package.

{% highlight python %}
import feedparser

def get_headlines(url, num_headlines=3):
    rss = feedparser.parse(url)
    headlines = []
    for entry in rss['entries'][:num_headlines]:
        headlines.append(entry['description'])
    return headlines
{% endhighlight %}

So all I needed to do was create an Alexa Skill that could run this bit of code and read back the headlines I wanted.

> Read about developing Alexa Skills in my [earlier blog post]({{site.baseurl}}/blog/2016/03/31/all-you-need-to-do-is-ask/).


First, I needed to come up with the *intent schema*. This specifies  __"what"__ your skill can do. Here's the schema I came up with:

{% highlight json %}
{
  "intents": [
    {
      "intent": "GetNewsIntent"
    },
    {
      "intent": "GetNewsSectionIntent",
      "slots": [
        {
          "name": "Section",
          "type": "LIST_OF_SECTIONS"
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

The skill can do four things:

1. `GetNewsIntent`: Get the default news headlines
2. `GetNewsSectionIntent`: Get the headlines for a particular news section specified in the custom slot type `LIST_OF_SECTIONS`. This is a list of string values: `"tech", "U.S.", "world"`
3. `AMAZON.HelpIntent`: This is a built-in intent that will read help info to the user.
4. `AMAZON.CancelIntent`: Another built-in intent, this one exits the skill.

The next step is to define __"how"__ the user will ask for these things. In ASK, this is done by specifying *sample utterances* for each of the skill's intents.

*Note: The built-in Amazon intents already have predefined utterances, so you don't need to specify them.*

For example, here are the utterances I came up with for my news skill:

```
GetNewsIntent what is in the news
GetNewsIntent what is in the news today
GetNewsIntent what's in the news
GetNewsIntent what's in the news today
GetNewsIntent what's new today
GetNewsIntent what is in the headlines
GetNewsIntent what's in the headlines
GetNewsIntent read me the headlines
GetNewsIntent read today's headlines
GetNewsSectionIntent what is in the {Section} news
GetNewsSectionIntent what is in the {Section} news today
GetNewsSectionIntent what's in the {Section} news
GetNewsSectionIntent what's in the {Section} news today
GetNewsSectionIntent what's new in {Section}
GetNewsSectionIntent what's new in {Section} today
GetNewsSectionIntent what's new in the {Section}
GetNewsSectionIntent what's new in the {Section} today
GetNewsSectionIntent what is in the {Section} headlines
GetNewsSectionIntent what's in the {Section} headlines
GetNewsSectionIntent read me the {Section} headlines
GetNewsSectionIntent read today's {Section} headlines
```

Slots can be referenced using curly braces `{...}` with the name of the slot. In my intents schema, I used the name "Section". So when the user says one of the sample utterances that has `{Section}` in it, Alexa will try to match the word said to the list of sections I defined in my custom slot type.

Finally, I modified the AWS Lambda blueprint to run my code with the intents and utterances above. The result can be found in my [Github repository](https://github.com/ewenchou/alexa-news-headlines).

Here's a short video of the skill's output. Enjoy :)

<iframe width="560" height="315" src="https://www.youtube.com/embed/7j5LwL7RkSc" frameborder="0" allowfullscreen></iframe>
