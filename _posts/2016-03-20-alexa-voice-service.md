---
layout: post
title: Alexa Voice Service
categories: blog
tags: alexa github
---
  > TL;DR: Amazon's Alexa is cool. I wrote some [code](https://github.com/ewenchou/alexa-client) to talk to it.

## Updated: March 27, 2016

Amazon has released a [very good demo and guide](https://github.com/amzn/alexa-avs-raspberry-pi) for turning a Raspberry Pi into an Alexa Voice Service device on Github. In particular, the  [README](https://github.com/amzn/alexa-avs-raspberry-pi/blob/master/README.md) file has very detailed sections on how to [setup your Raspberry Pi](https://github.com/amzn/alexa-avs-raspberry-pi/blob/master/README.md#0---setting-up-the-raspberry-pi) and how to [get started](https://github.com/amzn/alexa-avs-raspberry-pi/blob/master/README.md#3---getting-started-with-alexa-voice-service) with Alexa Voice Service, which I did not cover in detail in the post below. I encourage everyone interested in AVS to read through their demo repository (I did!).

## Preamble

So my [last post](/blog/2016/02/08/i-have-pi-now-what/) was well over a month ago *(oops)*. There were a lot of late nights and weekend hacking sessions (as my beautiful wife can attest to, sorry babe :D), some frustration, a lot of trial-and-error, and many iterations of code. Long story short... I can't really remember everything that I did *(oops again)*.

Fortunately, I kept a bunch of code repositories that'll help me distill my experiences into what I hope will be some good posts. I'll also be cleaning up those repos making them available on my [Github](https://github.com/ewenchou)

## Alexa

When Amazon announced the [Echo](http://www.amazon.com/gp/product/B00X4WHP5E/ref=sv_devicesubnav_0) back in 2014. I remember dismissing it as another [gimmicky attempt](https://en.wikipedia.org/wiki/Fire_Phone) by Amazon to sell you more stuff.

At the time, it could only do a limited number of things:

* You can talk to it (a la Siri, Cortana, and Google Now) and ask for simple things like the weather or Wikipedia references
* You can tell it to play music or tell you a joke
* And of course, you can ask it to buy things from Amazon

But over the course of 2015, my opinion of it Alexa, the service; not necessarily the Echo device itself) changed from dismissive to impressed.

More people got their hands on an Echo and the reviews were unanimously positive. Unlike some of the other voice assistants out there, Alexa seems to work really well. So well in fact, that most people refer to "her" as Alexa and the "Echo" name has pretty much been forgotten by all except the techies. I mean, how many people refer to their iPhones as Siri or their PC as Cortana?

Alexa's potential also became more apparent as Amazon opened up the service to developers. In particular, it quickly earned its place as the center of the smart home as an increasing number of third party integrations became available.

As mentioned in [my last post](/blog/2016/02/08/i-have-pi-now-what/), all of this was floating in the back of my mind while I was trying to figure out what to build with my Raspberry Pi.

## Getting Started with Alexa

So my first step was to sign up for the [developer preview](https://developer.amazon.com/public/solutions/alexa/alexa-voice-service). It was fairly straightforward, especially since I already had an Amazon account. [IIRC](http://www.urbandictionary.com/define.php?term=iirc), I just needed to agree to the Terms of Service. Amazon's [Getting Started Guide](https://developer.amazon.com/appsandservices/solutions/alexa/alexa-voice-service/getting-started-with-the-alexa-voice-service) is pretty good. In particular, the excerpt below was what I needed.

__Register with the Alexa Voice Service__

The following steps describe how to register your device or application with the Alexa Voice Service:

1. Get a free Amazon developer account if you do not already have one.
2. Sign into the Alexa developer portal.
3. Select Get Started in the Alexa Voice Service button.
4. __In the Register a Product Type menu, select Device or Application.__
5. __Enter the Device or Application Type Information.__
6. __Select or create a Security Profile to allow Amazon to identify and authenticate your device or application.__
7. __Enter Device or Application Details.__
8. To enable Amazon Music on your device, complete the questionnaire.
9. Select Submit to complete the registration process.

The steps in __bold__ are the ones needed to get the ID values needed for interacting with Alexa Voice Service.

Next, I started writing a standalone Python client to interact with Alexa Voice Service (AVS) based on the code from [AlexaPi](https://github.com/sammachin/AlexaPi).

In particular, I extracted the code for interacting with AVS and abstracted it into a few methods:

### get_token()

This is more-or-less unchanged from the `gettoken()` method in AlexaPi. It's responsible for getting the client token needed to send requests to AVS.

{% highlight python %}
def get_token(self, refresh=False):
    """Returns AVS access token.

    If first call, will send a request to AVS to obtain the token
    and save it for future use.

    Args:
        refresh (bool): If set to True, will send a request to AVS
                        to refresh the token even if one's saved.

    Returns:
        AVS access token (str)
    """
    # Return saved token if one exists.
    if self._token and not refresh:
        return self._token
    # Prepare request payload
    payload = {
        "client_id" : settings.CLIENT_ID,
        "client_secret" : settings.CLIENT_SECRET,
        "refresh_token" : settings.REFRESH_TOKEN,
        "grant_type" : "refresh_token"
    }
    url = "https://api.amazon.com/auth/o2/token"
    res = requests.post(url, data=payload)
    res_json = json.loads(res.text)
    self._token = res_json['access_token']
    return self._token
{% endhighlight %}

### get_request_params()

This is a utility method that returns a tuple of parameters needed for every AVS request. In particular, it will setup the `Authorization` HTTP header and the request payload parameters that are always needed.

{% highlight python %}
def get_request_params(self):
    """Returns AVS request parameters

    Returns a tuple of parameters needed for an AVS request.

    Returns:
        Tuple (url, headers, request_data) where,

           url (str): Request URL
           headers (dict): Request headers
           request_data (dict): Predefined request payload parameters
    """
    url = "https://access-alexa-na.amazon.com/v1"
    url += "/avs/speechrecognizer/recognize"
    headers = {'Authorization' : 'Bearer %s' % self.get_token()}
    request_data = {
        "messageHeader": {
            "deviceContext": [
                {
                    "name": "playbackState",
                    "namespace": "AudioPlayer",
                    "payload": {
                        "streamId": "",
                        "offsetInMilliseconds": "0",
                        "playerActivity": "IDLE"
                    }
                }
            ]
        },
        "messageBody": {
            "profile": "alexa-close-talk",
            "locale": "en-us",
            "format": "audio/L16; rate=16000; channels=1"
        }
    }
    return url, headers, request_data
{% endhighlight %}

### save_response_audio()

This is another utility method for extracting and saving the audio file returned in the AVS response.

{% highlight python %}
def save_response_audio(self, res, save_to=None):
    """Saves the audio from AVS response to a file

    Parses the AVS response object and saves the audio to a file.

    Args:
        res (requests.Response): Response object from request.
        save_to (str): Filename including path for saving the
                       audio. If `None` a random filename will
                       be used and saved in the `TEMP_DIR`.

    Returns:
        Path (str) to where the audio file is saved.
    """
    if not save_to:
        save_to = "{}/{}.mp3".format(settings.TEMP_DIR, uuid.uuid4())
    with open(save_to, 'wb') as f:
        if res.status_code == requests.codes.ok:
            for v in res.headers['content-type'].split(";"):
                if re.match('.*boundary.*', v):
                    boundary =  v.split("=")[1]
            response_data = res.content.split(boundary)
            for d in response_data:
                if (len(d) >= 1024):
                    audio = d.split('\r\n\r\n')[1].rstrip('--')
            f.write(audio)
            return save_to
        # Raise exception for the HTTP status code
        print "AVS returned error: {}: {}".format(
            res.status_code, res.content)
        res.raise_for_status()
{% endhighlight %}

### ask()

The primary method used to interact with AVS. It takes an audio file as input, sends it to AVS and saves the response audio so it can be played back.

{% highlight python %}
def ask(self, audio_file, save_to=None):
    """
    Send a command to Alexa

    Sends a single command to AVS.

    Args:
        audio_file (str): File path to the command audio file.
        save_to (str): File path to save the audio response (mp3).

    Returns:
        File path for the response audio file (str).
    """
    with open(audio_file) as in_f:
        url, headers, request_data = self.get_request_params()
        files = [
            (
                'file',
                (
                    'request', json.dumps(request_data),
                    'application/json; charset=UTF-8',
                )
            ),
            ('file', ('audio', in_f, 'audio/L16; rate=16000; channels=1'))
        ]
        res = requests.post(url, headers=headers, files=files)
        return self.save_response_audio(res, save_to)
{% endhighlight %}

### ask_multiple()

In my testing, I found that the latency for each AVS request is around 3-5 seconds. This is OK if you're interacting with Alexa like you would with an Amazon Echo (i.e. speaking each request one after the other). However, if you're interacting programatically (more on that in the next post), where you want to send multiple requests at a time, that latency quickly adds up.

So I added code using the `requests_futures` package that will allow sending mulitple requests concurrently. This way I would get the responses for __all__ the requests in about 3-5 seconds.

{% highlight python %}
def ask_multiple(self, input_list):
    """Sends multiple requests to AVS concurrently.

    Args:
        input_list (list): A list of input audio filenames to send
                           to AVS. The list elements can also be a
                           tuple, (in_filename, out_filename) to
                           specify where to save the response audio.
                           Otherwise the responses will be saved to
                           `TEMP_DIR`.

    Returns:
        List of paths where the responses were saved.
    """
    session = FuturesSession(max_workers=len(input_list))
    # Keep a list of file handlers to close. The input file handlers
    # need to be kept open while requests_futures is sending the
    # requests concurrently in the background.
    files_to_close = []
    # List of saved files to return
    saved_filenames = []
    # List of future tuples, (future, output_filename)
    futures = []

    try:
        for inp in input_list:
            # Check if input is a tuple
            if isinstance(inp, tuple):
                name_in = inp[0]
                name_out = inp[1]
            else:
                name_in = inp
                name_out = None

            # Open the input file
            in_f = open(name_in)
            files_to_close.append(in_f)

            # Setup request parameters
            url, headers, request_data = self.get_request_params()
            files = [
                (
                    'file',
                    (
                        'request', json.dumps(request_data),
                        'application/json; charset=UTF-8',
                    )
                ),
                (
                    'file',
                    ('audio', in_f, 'audio/L16; rate=16000; channels=1')
                )
            ]

            # Use request_futures session to send the request
            future = session.post(url, headers=headers, files=files)
            futures.append((future, name_out))

        # Get the response from each future and save the audio
        for future, name_out in futures:
            res = future.result()
            save_to = self.save_response_audio(res, name_out)
            saved_filenames.append(save_to)
        return saved_filenames
    finally:
        # Close all file handlers
        for f in files_to_close:
            f.close()
{% endhighlight %}

And there you have it, a simple Python client to interact with Alexa Voice Service. You can find all the code on my [Github](https://github.com/ewenchou/alexa-client).

Thanks for reading!
