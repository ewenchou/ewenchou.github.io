---
layout: post
title: Putting It All Together
categories: blog
tags: raspberry-pi alexa bluetooth python flask
---
I've now blogged about most of the pieces I needed for my [morning greeter project]({{ site.base_url }}/blog/2016/03/25/good-morning-alexa/). So now it's time to write about putting it all together into a single, working solution.

Here's a quick recap of the pieces so far:

1. Raspberry Pi with Bluetooth USB Adapter
2. [Alexa Voice Service Client](https://github.com/ewenchou/alexa-client)
3. [Text-to-Speech](https://github.com/ewenchou/simple-tts)
4. Custom Alexa Skills:
    * [News](https://github.com/ewenchou/alexa-news-headlines)
    * [Stocks](https://github.com/ewenchou/alexa-stock-quotes)
    * [Traffic](https://github.com/ewenchou/alexa-traffic-time)
5. [Bluetooth Proximity Detection](https://github.com/ewenchou/bluetooth-proximity)

## Alexa Agent

To make it easier to send commands to Alexa, I wrote a simple agent class using Python with two main interface methods.

{% highlight python %}
    def say(self, input):
        """Alexa will say the text.
        
        @param: input: Text you want Alexa to say.
        @type: input: str or list of str
        """
        # do some stuff

    def ask(self, input):
        """Ask Alexa to do something(s).
        
        @param: input: Text command you want to send to Alexa.
        @type: input: str or list of str
        """
        # do some stuff
{% endhighlight %}

The agent will take care of TTS, sending the request to AVS, and playing back the response. The full code is available in my Github repository: [alexa-agent](https://github.com/ewenchou/alexa-agent)

## Alexa API

Next, I needed a way to connect triggers (e.g. bluetooth proximity detection) to Alexa commands. A flexible way for doing this is through a simple web service API that can be invoked via HTTP requests.

This can be done very quickly using [Flask](http://flask.pocoo.org/):

{% highlight python %}
#!/usr/bin/env python

from flask import Flask, jsonify
from alexa_agent import AlexaAgent


app = Flask(__name__)


@app.route('/morning-report', methods=['GET'])
def morning_report():
    agent = AlexaAgent()
    agent.wakeup()
    agent.ask([
        "What is today's date",
        "What time is it",
        "How's the weather"
    ])
    return jsonify({'code': 200, 'message': 'Morning report delivered!'})


if __name__ == '__main__':
    app.run(port=8888, debug=True)
{% endhighlight %}

The above code, along with sample WSGI and Apache configuration files is available in my Github repository: [alexa-agent-flask](https://github.com/ewenchou/alexa-agent-flask)

_Note: If you're using custom Alexa skills, make sure to modify the list of commands with the appropriate intent phrases._

## Bluetooth Scanner

The last piece is the bluetooth proximity detection trigger. I recently updated my [bluetooth-proximity](https://github.com/ewenchou/bluetooth-proximity) repository with a [simple scanner](https://github.com/ewenchou/bluetooth-proximity/blob/master/examples/bluetooth_scanner.py) script that can be modified for this purpose. 

Simply replace the `dummy_callback()` function with a function that will call the Flask API endpoint for the morning report. For example:

{% highlight python %}
import requests

def get_morning_report():
    requests.get('http://127.0.0.1/alexa-api/morning-report')
{% endhighlight %}

You can test if the bluetooth scanner is working by running the script manually (i.e. `python bluetooth_scanner.py`). Once it's working, you can set it up as a simple `systemd` service using the steps below.

### Systemd Service 

1. Write a `systemd` service file named `bluetooth_scanner.service`

        [Unit]
        Description=Bluetooth Scanner

        [Service]
        Type=simple
        ExecStart=/usr/bin/bluetooth_scanner.py

        [Install]
        WantedBy=multi-user.target
        
2. Copy the `bluetooth_scanner.py` script to `/usr/bin/`
3. Copy the `bluetooth_scanner.service` file to `/lib/systemd/system/`
4. Reload the `systemd` daemon using the command `systemctl daemon-reload`
5. Start the bluetooth scanner service using the command: `systemctl start bluetooth_scanner.service`
6. Enable the service to auto-start with the system using the command: `systemctl enable bluetooth_scanner.service`

That's it! You should now have a bluetooth triggered morning report. I hope this series of blog posts has been helpful. Thanks for reading.