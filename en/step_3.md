## Adding a Graphical User Interface.

--- task ---
Add in `guizero` module. (needs installing)

```python
##Import the required modules
from pygame import mixer
from mutagen.mp3 import MP3
from os import listdir
from random import shuffle
from guizero import App, PushButton, Slider, Text
```
--- /task ---

--- task ---
Grab the graphics and place in an `icons` directory - shortlink to be created.
--- /task ---

--- task ---
Fairly self explanatory what the guizero buttons do.
```python
#guizero UI
play = PushButton(app, command=play_track, image="icons/play.png", grid=[0,0])
stop = PushButton(app, command=stop_track, image="icons/stop.png", grid=[1,0])
pause = PushButton(app, command=pause_track, image="icons/pause.png", grid=[2,0])
previous = PushButton(app, command=previous_track, image="icons/previous.png", grid=[3,0])

advance = PushButton(app, command=next_track, image="icons/next.png", grid=[6,0])
random = PushButton(app, command=shuffle_tracks, image="icons/shuffle.png", grid=[7,0])

#run the app (keep this at bottom of file)
app.display()
```
--- /task ---

--- task ---
Add a volume slider
```python
volume = Slider(app, command=change_volume, horizontal=False, start="100", end="0", height=128, grid=[8,0,8,3])
```
--- /task ---

--- task ---
Add text to show what's playing
```
playing = Text(app, text=tracks[position], grid=[0,2,7,2])
```
--- /task ---

--- task ---
Add progress slider
```python
progress = Slider(app, width="400", grid=[0,1,7,1])
```
--- /task ---

Now to control the progress slider.

--- task ---
Create a new `global` for the number of seconds played and if the track is paused

```python
##Globals
position = 0
paused = False
seconds_played = 0
```
--- /task ---

--- task ---
Create a new function to check advance seconds played, when track not stoppped or paused. (Note pygame will time keep how long a track has been playing, but pauseing doesn't stop the time.) Add this beneath the `jog_forward()` function.

If the music is loaded and not paused, then advance the timer by 3.

```python
def check_progress():
    global seconds_played
    if mixer.music.get_busy() == 1 and paused == False:
        seconds_played += 3
```
--- /task ---

--- task ---
Now switch pause to `True` when paused, and `False` when playing, in earlier functions
Also switch `seconds_played` to `0` when a track starts playing

```python
def play_track():
    global seconds_played
    global paused
    if mixer.music.get_busy() == 0:
        seconds_played = 0
        paused = False
        track = PATH + tracks[position]
        playing.value = tracks[position]
        mixer.music.load(track)
        mixer.music.play()
    else:
        paused = False
        mixer.music.unpause()
		
		
def pause_track():
    """Pause the current track"""
    global paused
    mixer.music.pause()
    paused = True
```
--- /task ---

--- task ---
You can run your `check_progress()` function every three seconds.

```python
#update the text for which track is playing each second
playing.repeat(3000, check_progress)

#run the app
app.display()
```
--- /task ---

--- task ---
Now that we know how many seconds a track has been played, we need the total length of the track to monitor it's progress. We can use `mutagen` for this, to build a dictionary of track lenghts.

```python
##Import the required modules
from mutagen.mp3 import MP3


##Find the length of each track and store in dictionary
lengths = {track: MP3(PATH + track).info.length for track in tracks}
```
--- /task ---

--- task ---
Now in the `check_progress()` function, calculate the percentage progress and use the slider to display it.

```python
def check_progress():
    global seconds_played
    if mixer.music.get_busy() == 1 and paused == False:
        seconds_played += 3
    percentage_progress = 100 / lengths[tracks[position]] * seconds_played 
    progress.value = percentage_progress
```
--- /task ---

--- task ---
An advantage of knowing the time the track is at, is that we can use it to have jog buttons

```python
def jog_back():
    """Go back five seconds"""
    global seconds_played
    seconds_played -= 5
    mixer.music.set_pos(seconds_played)


def jog_forward():
    """Go forward five seconds"""
    global seconds_played
    seconds_played += 5
    mixer.music.set_pos(seconds_played)
	

back = PushButton(app, command=jog_back, image="icons/back.png", grid=[4,0])
forward = PushButton(app, command=jog_forward, image="icons/forward.png", grid=[5,0])
```
--- /task ---

--- task ---
At the moment, when a track comes to an end, the MP3 player needs prompting to play the next track. This can be fixed, but we'll need another global variable.

```python
##Globals
position = 0
paused = False
seconds_played = 0
stopped = True
```
--- /task ---

--- task ---
When a track is stopped, we want to set `stopped` to `True`, and when we play a track it needs to be `False`

```python
def play_track():
    global stopped
    global seconds_played
    global paused
    sense.set_pixels(play_icon)
    if mixer.music.get_busy() == 0:
        seconds_played = 0
        stopped = False
        paused = False
        track = PATH + tracks[position]
        playing.value = tracks[position]
        mixer.music.load(track)
        mixer.music.play()
    else:
        paused = False
        mixer.music.unpause()
		
		
def stop_track():
    global stopped
    stopped = True
    mixer.music.stop()
```
--- /task ---

--- task ---
Now the `check_progress()` function (that is called every 3 seconds), can check if a track has stopped, because it ended, or because the user manually stopped the track. If the former, it can progress to the next track, and update the name of the track on the GUI.

```python
def check_progress():
    global seconds_played
    if mixer.music.get_busy() == 1 and paused == False:
        seconds_played += 3
    percentage_progress = 100 / lengths[tracks[position]] * seconds_played 
    progress.value = percentage_progress
    if mixer.music.get_busy() == 0 and stopped == False:
        next_track()
        playing.value = tracks[position]
```
--- /task ---
