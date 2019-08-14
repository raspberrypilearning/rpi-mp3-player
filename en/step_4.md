##Add a SenseHat display

You're going to add a SenseHat to display some information.

--- task ---
Download the sense_hat_icons.py file and place it in your project directory.
--- /task ---

--- task ---
Add the following to your imports to access this set of icons

```python
from sense_hat import SenseHat
from sense_hat_icons import *
```
--- /task ---

--- task ---
Create a `SenseHat()` object

```python
##Initialise the mixer and create objects
mixer.init()
app = App(title="Hallelujah Player", height=140, width = 610, layout="grid")
sense = SenseHat()
```
--- /task ---

--- task ---
Now add calls to show the icons within the control functions.

```python
def play_track():
    """
    Load a track if one not already paused, and play it
    Otherwise, unpause a track
    """
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
        sense.set_pixels(play_icon)
    else:
        paused = False
        mixer.music.unpause()
		
def stop_track():
    """Set the stopped variable to True and stop the track"""
    global stopped
    stopped = True
    mixer.music.stop()
    sense.set_pixels(stop_icon)

def pause_track():
    """Pause the current track"""
    global paused
    mixer.music.pause()
    paused = True
    sense.set_pixels(pause_icon)
```
--- /task ---

--- task ---
We're going to use some colours for the SenseHat so set these up as constants.

```python
RED = (255,0,0)
WHITE = (255,255,255)
GREEN = (0,255,255)
```
--- /task ---
--- task ---
Create a function to show the volume on the SenseHAT. You can get the current volume using `mixer.music.get_volume()`. This will be a value between 0 and 1. Multiplying by 8 will get the number of pixels that need colouring.

In the far right column of pixels, they can all be set to turn off, then the correct number of pixels can be illuminated in ted.
```python
def show_volume_display():
    volume = int(mixer.music.get_volume() * 8)
    column = 7
    for row in range(8):
        sense.set_pixel(column, row, b)
    for row in range(7, 7-volume, -1):
        sense.set_pixel(column, row, RED)
```
--- /task ---

--- task ---
This function needs to be called each time the volume changes and after the icons are changed.

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
        sense.set_pixels(play_icon)
        show_volume_display()
    else:
        paused = False
        mixer.music.unpause()

        
def stop_track():
    global stopped
    stopped = True
    mixer.music.stop()
    sense.set_pixels(stop_icon)
    show_volume_display()
	

def pause_track():
    """Pause the current track"""
    global paused
    mixer.music.pause()
    paused = True
    sense.set_pixels(pause_icon)
    show_volume_display()
	
	
def change_volume(volume):
    """Change the volume according to the slider"""
    volume = int(volume) / 100
    mixer.music.set_volume(volume)
    show_volume_display()
```
--- /task ---

--- task ---
The same trick can be used to display the progress of the track. 

```python
def show_progress_display(percentage_progress):
    abs_pos = int(percentage_progress / 100 * 7)
    row = 7
    for column in range(7):
        sense.set_pixel(column, row, BLACK)
    for column in range(abs_pos):
        sense.set_pixel(column, row, GREEN)
```

and this function can be called in the `check_progress()` function, so it runs every 3 seconds.

```python
def check_progress():
    global seconds_played
    if mixer.music.get_busy() == 1 and paused == False:
        seconds_played += 3
    percentage_progress = 100 / lengths[tracks[position]] * seconds_played 
    progress.value = percentage_progress
    show_progress_display(percentage_progress)
    if mixer.music.get_busy() == 0 and stopped == False:
        next_track()
        playing.value = tracks[position]
```
--- /task ---

The last thing that would be nice to have, is the track name scrolling across the SenseHAT whenever the track changes.

The function to scroll text is blocking. This means that no other code will run while the track name is being displayed.

To overcome this we need to run a display function in parallel with the rest of the code. This can be done using threading

--- task ---
Import the threading module.

```python
##Import the required modules
from pygame import mixer
from guizero import App, PushButton, Slider, Text
from mutagen.mp3 import MP3
from os import listdir
from random import shuffle
from sense_hat import SenseHat
from sense_hat_icons import *
from threading import Thread
```
--- /task ---

--- task ---
Now write the function to display the track name. It will display each time the `position` variable alters.

```python
def show_track_name():
    new_position = position
    sense.set_pixels(play_icon)
    while True:
        if new_position != position:
            sense.show_message(tracks[position])
            show_volume_display()
            new_position = position
```
--- /task ---

--- task ---
Now start the function in it's own thread.

```python
show_track = Thread(target=show_track_name)
show_track.start()
```
--- /task ---
