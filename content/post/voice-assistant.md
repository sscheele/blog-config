+++
date = "2017-06-29T12:04:27-04:00"
title = "Voice Assistant"

+++

# Problem: Capitalism. As usual.

It's always been a fun idea to try and create a voice assistant of your own that you can program to do those specialized tasks that Cortana, Google Assistant, and Alexa can't. The problem is that the people who make voice assistants are in it for the money or something. I've managed to get around this and create a voice assistant for myself - it's a little buggy, but a nice concept.

# Solution: Use Google's software.

#### Note: what follows is mostly an explanation of how I built my assistant. If you're just interested in the assistant itself with its default functionality, go get it at https://github.com/sscheele/super-assist

If you're looking to really build an assistant yourself, you've probably heard of CMU Sphinx. It's arguably the best free solution out there for voice recognition. Unfortunately, it's still pretty bad. I decided to go a different route: I installed Google's voice recognition software on a Raspberry Pi (see [this link](https://developers.google.com/assistant/sdk/prototype/getting-started-pi-python/) for instructions directly from Google). Now, that software provides Google's Assistant, which was just a bonus for me - "my" assistant can modify calendar events, set alarms, and tell jokes without my intervention. But it can't (as of this writing) play music. Let's add that feature.

## Parse the text

Google is nice enough to provide us with output formatted like this once it has performed speech recognition:

```
{'text': 'hello, world'}
```

Unfortunately, they don't want us getting our hands on that text. I tried piping it to another program, writing it to a file and reading from that same file, and a few other tricks, Google covered all the traditional bases pretty well. So the first thing my program does is have the Pi SSH into itself. The assistant writes out to SSH with no problems, and I can pipe SSH into whatever I want.

## Do fun things with it

I wrote a framework easy enough for anyone to program for - all you have to do is write a method in Python that accepts a dictionary containing arguments and a channel, then write one line of code adding it to the framework. My first task was writing a task that could play YouTube videos. I won't bore you with the code to search YouTube or pull down the audio - it's all on the above GitHub repository, anyway. The relevant code is:

```
def search_yt(args, chan):
    """ Return the first youtube result for a link """
    print("Started", args['query'])
    url = "https://www.youtube.com/results?search_query=" + quote_plus(args['query'])
    # (removed for brevity - search for and play the video)
    while True:
        tmp = chan.read()["command"]
        if tmp == "pause" or tmp == "play":
            # (removed for brevity - toggle pause/play)
        elif tmp == "PKILL":
            return


YT_TASK = Task("youtube",
               [Expression(compile(r"search for (.+) on youtube"), ('query',)),
                Expression(compile(r"search youtube for (.+)"), ('query',)),
                Expression(compile(r"play (.+)"), ('query',)),
                ],
               [Expression(compile("pause"), ("command",)),
                Expression(compile("play"), ("command",))
                ],
               search_yt)
```

The important information here is that `search_yt` takes a dictionary of arguments, whose names are specified in the `Task` constructor, and a "channel" (concept borrowed from Golang). All this code is running in a separate thread, insofar as that's a thing in Python (which it's not). `chan.read()` blocks until something is written to the channel. The `Task` constructor should be fairly clear - the task can be started by three different phrases, and commands can be written to the channel by two phrases. Note that all letters should be lowercase. There's also a `PKILL` command (keyword borrowed from Linux) - if a channel receives this, your thread should terminate. 

Using this simple model, it's my hope that anyone will be able to build modules for this framework to make their own unique assistants - the sky is the limit!