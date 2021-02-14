# Jarvis (Iron Man Assistant) Alram
Any Iron Man/Marvel fan is familiar with JARVIS(Just A Rather Very Intelligent System…lol clever), Tony Stark’s 
> natural-language user interface computer system, named after  [Edwin Jarvis](https://ironman.fandom.com/wiki/Edwin_Jarvis_(film)) , the butler who worked for  [Howard Stark](https://ironman.fandom.com/wiki/Howard_Stark_(MCU)) . 

Here is my favorite scenes and the initial inspiration for this project, enjoy!

[Wake Up Daddy’s Home](https://youtu.be/TC77zYMoKJE)

[Alarm Clock](https://www.youtube.com/watch?v=SD9tCCp1F8s)

While building something to the same capacity as what’s shown in the movie would be a huge undertaking(although I’m sure some one is working on it or has done it), I figured I would settle for a JARVIS alarm at the very least, which is exactly what I did a couple months ago! Since then I’ve been adjusting and tweaking its functions and I think its ready to be shared!

Let me walk you through my *thought process* and how it came together!
1. First up was figuring out how the get the iconic JARVIS voice, with out it there is no JARVIS. Initially I considered snipping audio from the movie scenes and after doing a quick YouTube search I found plenty of clips. Unfortunately all of them had background noice or music and really wouldn’t do it justice but then I came across this [JARVIS App](https://www.youtube.com/watch?v=6i5hho2aD-E)! It looks like a couple of years back as part of Iron Man 3  promotional campaign, Marvel released a version of JARVIS through an app.
	> Know what it’s like to be Tony Stark and have your own personal JARVIS, now available for free download on iOS devices!  
To my luck the app is no longer available in the App Store but I wasn’t ready to give up just yet. I did a google search 	for ‘jarvis audio files’ and lead me to a [reddit post](https://www.reddit.com/r/Marvel/comments/71k7nk/iron_man_3_jarvis_app_sound_files/) of some one who had found a copy of the app, extracted all 	the audio files, and uploaded them to google drive! BINGO, the zip file had a total of 380 audio clips, all kinds of useful snippets including numbers, time, weather and more!

2. Next up was choosing the hardware where the JARVIS alarm would ‘live in’.  I wanted this alarm to be ‘bullet proof’ and maintenance free, I’ve used plenty alarms over the years, physical alarms,  iPhone alarms, but they are all too easy to turn off and that’s if I don’t forget to set it up the night before! So I decided to go with a good ole reliable Raspberry Pi3, while it may be a bit over kill for a simple alarm, I already have a Pi3  hooked up to speakers and configured running other projects including:
	1. [Homebridge](https://homebridge.io/)  a node js server to link non-HomeKit devices to Apple’s HomeKit setup
	2. [Shairport Sync](https://github.com/mikebrady/shairport-sync) a service that essentially mimics AirPlay enabling audio streaming from any iOS/macOS device
	3. [Spotify Remote](https://github.com/flunkout-dvlpr/spotify-remote) a service I created to make a physical Spotify remote using an LED screen and arcade buttons

  Stay tuned, posts on these topics coming soon!

3. The last piece of the puzzle was the code that would play the JARVIS audio clips in the precise order and time. For this I decided to go with a pretty basic python script. Below I will walk through all the functions I crated and demo the final result!