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
* First thing we have to do is initialize an instance of pygame in order to be able to play the audio snippets
```python
import pygame as pg

print("Initializing PyGame Mixer")
pg.mixer.init()

```
* To make sure I hear the alarm even if the Pi3’s volume has been lowered or muted, I set the volume using pyalsaaudio a wrapper that allows us to access ALSA Audio using python:  
	> Advanced Linux Sound Architecture (ALSA) provides audio and MIDI functionality to the Linux operating system.  
* We define a mixer instance and set the control to PCM
* Using the setvolume() method we set the Pi3’s volume to the desired value
```python
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
```python
# Set jarvis_dir to your actual path such as /pi/.../JarvisAudio
jarvis_dir = "JarvisAudio"
jarvis_temperature_dir = "{}/temperature".format(jarvis_dir)
jarvis_weekdays_dir = "{}/weekdays".format(jarvis_dir)
jarvis_numbers_dir = "{}/numbers".format(jarvis_dir)
jarvis_dates_dir = "{}/dates".format(jarvis_dir)
jarvis_wake_up_song = "{}/song_{}.ogg".format(jarvis_dir, random.randint(1,3))
song = pg.mixer.Sound(jarvis_wake_up_song)
```
### Functions
#### `currentTime()`
```python
def currentTime():
  now = datetime.datetime.now()
  hour = now.hour
  if hour > 0 and hour < 12:
    period = "time_am"
    hour = str(hour)
  elif hour == 12:
    period = "time_pm"
    hour = str(hour)
  elif hour > 12 and hour < 24:
   period = "time_pm"
   hour = str(hour - 12)
  else:
    period = "time_am"
    hour = "12"

  minute = now.minute
  if minute == 0:
    minute = "oclock"
  elif minute < 10:
    minute = "0" + str(minute)
  else:
    minute = str(minute)

  return hour, minute, period
```
#### `getForecast()`
```python
def getForecast(city_name):
  # store as ENV variable
  api_key = OPEN_WEATHER_API_KEY

  # Opean Weather API request url 
  api_url = "http://api.openweathermap.org/data/2.5/weather?q={}&appid={}".format(city_name,api_key)
  
  # Request response
  response = requests.get(api_url)

  #JSON response 
  json_response = response.json()

  if json_response["cod"] == 200:
    kelvin_temperature = json_response["main"]["temp"]
    celcius_temperature = round(kelvin_temperature - 273.15)
    fahrenheit_temperature = round((kelvin_temperature - 273.15) * 9/5 + 32)

    weather_id = json_response["weather"][0]["id"]
    weather_id_starting_digit = int(str(weather_id)[0])

    # 2xx, 3xx, 5xx, 6xx, 7xx, 8xx
    # https://openweathermap.org/weather-conditions
    if weather_id_starting_digit == 2:
      condition ="Thunderstorm"
    elif weather_id_starting_digit == 3:
      condition ="Light Rain"
    elif weather_id_starting_digit == 5:
      condition ="Rain"
    elif weather_id_starting_digit == 6:
      condition ="Snow"
    elif weather_id_starting_digit == 7:
      condition ="Fog/Mist/Haze"
    elif weather_id_starting_digit == 8:
      if weather_id == 800:
        condition ="Clear Sky"
      else:
        condition ="Cloudy"

    return celcius_temperature, fahrenheit_temperature, condition
  else:
    return False
```

#### `play_forecast(city)`
```python
def play_forecast(city):
  c_temp, f_temp, condition = getForecast(city)
  play_sound(sound=sounds["temperature_introduction"], count=1, wait=2)
  play_number(number=c_temp)
  play_sound(sound=sounds["degrees"], count=1, wait=1)
  play_sound(sound=sounds["celcius"], count=1, wait=1)

  if f_temp > 60 and int(str(f_temp)[1]) != 0:
    firstNumber = int(str(f_temp)[0] + "0")
    secondNumber = int(str(f_temp)[1]) 
    play_number(number=firstNumber)
    play_number(number=secondNumber)
  else:
    play_number(number=f_temp)
  play_sound(sound=sounds["degrees"], count=1, wait=1)
  play_sound(sound=sounds["fahrenheit"], count=1, wait=1)
```

#### `play_number(number)`
```python
def play_number(number):
  number_path = "{}/caged_num_{}.ogg".format(jarvis_numbers_dir,number)
  number = pg.mixer.Sound(number_path)
  number.play()
  time.sleep(1)
```

#### `play_sound(sound, count=1, wait=3)`
```python
def play_sound(sound="", count=1, wait=3):
  while count:
    audio = pg.mixer.Sound(sound)
    audio.play()
    time.sleep(wait)
    count -= 1
```

#### `play_date()`
```python
def play_date():
  now = datetime.datetime.now()
  weekday_str = now.strftime("%A").lower()
  weekday = "{}/caged_day_{}.ogg".format(jarvis_weekdays_dir,weekday_str)

  date_str = now.strftime("%d").lower()
  date = "{}/caged_date_{}.ogg".format(jarvis_dates_dir,date_str)

  play_sound(sound=weekday, count=1, wait=.5)
  play_sound(sound=sounds["date_introduction"], count=1, wait=.5)
  play_sound(sound=date, count=1, wait=1)

```

#### `play_time()`
```python

def play_time():
  hour, minute, period = currentTime()
  play_number(number=hour)
  play_number(number=minute)
  play_sound(sound=sounds[period], count=1, wait=1)
```


#### `play_song(song)`
```python

def play_song(song):
  song.set_volume(0.5)
  song.play()

  currentVolume = song.get_volume()

  while currentVolume < 1:
    currentVolume += .05
    song.set_volume(currentVolume)
    time.sleep(.5)

  time.sleep(60)

  currentVolume = song.get_volume()

  while currentVolume > .05:
    currentVolume -= .05
    song.set_volume(currentVolume)
    time.sleep(.5)
```