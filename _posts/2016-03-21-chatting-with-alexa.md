---
layout: post
title: Chatting with Alexa
categories: blog
tags: festival alexa tts
---
In a [previous post](/blog/2016/02/08/i-have-pi-now-what/) I mentioned my idea of using different triggers for Alexa. In particular, I didn't want to setup a hardware "push-to-talk" button. Now that I had [code](/blog/2016/03/20/alexa-voice-service/) to interact with Alexa Voice Service (AVS), I needed a way to generate the audio commands for the requests (instead of recording my voice for each command).

So I ended up looking for a text-to-speech (TTS) tool that could run on Linux. After a bit of searching on the Internet, I found that [Festival](http://festvox.org/festival/) seemed to be the best option for TTS on Linux.

I am using Ubuntu for my development and testing, and it was very straightforward to install.

    sudo apt-get install festival


As is the usual case, the [Arch Linux wiki](https://wiki.archlinux.org/index.php/Festival) also had a great entry for Festival that offers more details.

What I needed was the `text2wave` command that comes with the `festival` package when it is installed. With this command, you can give a text file as input and it can output the speech as a WAV file.

    text2wave -o <output_wav_file> <input_text_file>

I wrote a simple Python wrapper to execute the command:

{% highlight python %}
def tts(text, save_to=None):
    """Converts text to speech (WAV) file.

    Args:
        text (str): Text to convert
        save_to (str): File path for saving the WAV file. If not
                       provided will save to a `/tmp/simple-tts/`

    Returns:
        Path (str) where the WAV file is saved.
    """
    os.system('mkdir -p {}'.format(TEMP_DIR))
    temp_file = TEMP_DIR + '/{}.txt'.format(uuid.uuid4())
    if not save_to:
        save_to = TEMP_DIR + '/{}.wav'.format(uuid.uuid4())
    with open(temp_file, 'w') as f:
        f.write(text)
    os.system('text2wave -o {out_fn} {in_fn}'.format(
        out_fn=save_to, in_fn=temp_file))
    return save_to
{% endhighlight %}

After some testing, I found that Alexa had some trouble understanding the default voice that is used by Festival. After some more Google-Fu, I was able to find some [information](http://ubuntuforums.org/showthread.php?t=677277) about installing additional voice packs for Festival. I tested several of the voice packs and found that the [CMU Arctic clb (US English female voice)](http://www.speech.cs.cmu.edu/cmu_arctic/packed/cmu_us_clb_arctic-0.95-release.tar.bz2) gave the best results.

Here's the [TTS code](https://github.com/ewenchou/simple-tts), and a  [simple example](https://github.com/ewenchou/alexa-tts-demo) that shows how to tie it together with [alexa-client](https://github.com/ewenchou/alexa-client).

<iframe width="560" height="315" src="https://www.youtube.com/embed/9JhDP1F_ho0" frameborder="0" allowfullscreen></iframe>
