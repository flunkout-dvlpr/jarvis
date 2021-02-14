# Jarvis (Iron Man Assistant) Alram
Any Iron Man/Marvel fan is familiar with JARVIS(Just A Rather Very Intelligent System…lol clever), Tony Stark’s 
> natural-language user interface computer system, named after  [Edwin Jarvis](https://ironman.fandom.com/wiki/Edwin_Jarvis_(film)) , the butler who worked for  [Howard Stark](https://ironman.fandom.com/wiki/Howard_Stark_(MCU)) .  [Source](https://ironman.fandom.com/wiki/J.A.R.V.I.S.)

Here is my favorite scenes and the initial inspiration for this project, enjoy!

[Wake Up Daddy’s Home](https://youtu.be/TC77zYMoKJE)

[Alarm Clock](https://www.youtube.com/watch?v=SD9tCCp1F8s)

While building something to the same capacity as what’s shown in the movie would be a huge undertaking(although I’m sure some one is working on it or has done it), I figured I would settle for a JARVIS alarm at the very least, which is exactly what I did a couple months ago! Since then I’ve been adjusting and tweaking its functions and I think its ready to be shared!

Let me walk you through my *thought process* and how it came together!
1. First up was figuring out how the get the iconic JARVIS voice, with out it there is no JARVIS. Initially I considered snipping audio from the movie scenes and after doing a quick YouTube search I found plenty of clips. Unfortunately all of them had background noice or music and really wouldn’t do it justice but then I came across this [JARVIS App](https://www.youtube.com/watch?v=6i5hho2aD-E)! It looks like a couple of years back as part of Iron Man 3  promotional campaign, Marvel released a version of JARVIS through an app.  
	> Know what it’s like to be Tony Stark and have your own personal JARVIS, now available for free download on iOS devices! [Source](https://www.youtube.com/watch?v=6i5hho2aD-E)
	>
	To my luck the app is no longer available in the App Store but I wasn’t ready to give up just yet. I did a google search for ‘jarvis audio files’ and lead me to a [reddit post](https://www.reddit.com/r/Marvel/comments/71k7nk/iron_man_3_jarvis_app_sound_files/) of some one who had found a copy of the app, extracted all  the audio files, and uploaded them to google drive! BINGO, the zip file had a total of 380 audio clips, all kinds of useful snippets including numbers, time, weather and more!

2. Next up was choosing the hardware where the JARVIS alarm would ‘live in’.  I wanted this alarm to be ‘bullet proof’ and maintenance free, I’ve used plenty alarms over the years, physical alarms,  iPhone alarms, but they are all too easy to turn off and that’s if I don’t forget to set it up the night before! So I decided to go with a good ole reliable Raspberry Pi3, while it may be a bit over kill for a simple alarm, I already have a Pi3  hooked up to speakers and configured running other projects including:
	1. [Homebridge](https://homebridge.io/)  a node js server to link non-HomeKit devices to Apple’s HomeKit setup
	2. [Shairport Sync](https://github.com/mikebrady/shairport-sync) a service that essentially mimics AirPlay enabling audio streaming from any iOS/macOS device
	3. [Spotify Remote](https://github.com/flunkout-dvlpr/spotify-remote) a service I created to make a physical Spotify remote using an LED screen and arcade buttons

  Stay tuned, posts on these topics coming soon!

3. The last piece of the puzzle was the code that would play the JARVIS audio clips in the precise order and time. For this I decided to go with a pretty basic python3 script. Below I will walk through the code I wrote and demo the final result!

### Python Packages
* [time](https://docs.python.org/3/library/time.html): Used to pause/sleep in between audio clips
* [datetime](https://docs.python.org/3/library/datetime.html): Used to determine current date/time
* [pyalsaaudio](https://larsimmisch.github.io/pyalsaaudio/pyalsaaudio.html): Used to adjust the volume on the Raspberry Pi3
* [pygame](https://www.pygame.org/docs/ref/mixer.html): Used to play the audio clips
* [requests](https://requests.readthedocs.io/en/master/): Used to retrieve weather data from Open Weather Map API

I’ve included a requirements.txt file you can run using the command below to install all necessary packages except for time and datetime which come preinstalled with python3
COMMAND

### Set Up
* First thing we have to do is initialize an instance of pygame
```
import pygame as pg

print("Initializing PyGame Mixer")
pg.mixer.init()

```
* To make sure I hear the alarm even if the Pi3’s volume has been lowered or muted, I set the volume using pyalsaaudio a wrapper that allows us to access ALSA Audio using python:  
> 	Advanced Linux Sound Architecture (ALSA) provides audio and MIDI functionality to the Linux operating system.  
* We define a mixer instance and set the control to PCM, which will allow us to control the the volume coming out of the Pi3’s audio jack
* Using setvolume() method we set the Pi3’s volume to the desired value
```
import alsaaudio
import pygame as pg

print("Initializing PyGame Mixer")
pg.mixer.init()

print("Adjusting Volume...")
mixer = alsaaudio.Mixer(control="PCM")
print(mixer.getvolume())
mixer.setvolume(90)
print(mixer.getvolume())
```
* I sorted the JARVIS audio files into a main folder containing sub-folders based on use case (dates, weekdays, numbers, etc) and declared sub-folder paths as formatted strings, making it easy to edit the base path. 
```
# Set jarvis_dir to your actual path such as /pi/.../JarvisAudio
jarvis_dir = "JarvisAudio"
jarvis_temperature_dir = "{}/temperature".format(jarvis_dir)
jarvis_weekdays_dir = "{}/weekdays".format(jarvis_dir)
jarvis_numbers_dir = "{}/numbers".format(jarvis_dir)
jarvis_dates_dir = "{}/dates".format(jarvis_dir)
jarvis_wake_up_song = "{}/song_{}.ogg".format(jarvis_dir, random.randint(1,3))
song = pg.mixer.Sound(jarvis_wake_up_song)
``