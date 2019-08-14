## Controlling MP3 files with Python

--- task ---
Move MP3 files into the `/home/pi/Music` directory
--- /task ---

--- task ---
Import and initialise modules for MP3

```python
##Import the required modules
from pygame import mixer
from mutagen.mp3 import MP3

##Initialise the mixer and create objects
mixer.init()
```
--- /task ---

--- task ---
Create a constant for the `Music` directory path

```python
##Import the required modules
from pygame import mixer
from mutagen.mp3 import MP3

##Initialise the mixer and create objects
mixer.init()

##Create some constants
PATH = "/home/pi/Music/"
```
--- /task ---

--- task ---
Get all the tracks from the `Music` directory into a list. Create a global variable to store which track is playing and set it to an initial value of `0`

```python
##Import the required modules
from pygame import mixer
from mutagen.mp3 import MP3
from os import listdir

##Initialise the mixer and create objects
mixer.init()

##Create some constants
PATH = "/home/pi/Music/"

##Get the list of tracks from the Music dir
tracks = sorted([file for file in listdir(PATH) if file.endswith(".mp3")])

##Globals
position = 0
```
--- /task ---

--- task ---
Create a function to play an MP3 file

```python
##Import the required modules
from pygame import mixer
from mutagen.mp3 import MP3
from os import listdir

##Initialise the mixer and create objects
mixer.init()

##Create some constants
PATH = "/home/pi/Music/"

##Get the list of tracks from the Music dir
tracks = sorted([file for file in listdir(PATH) if file.endswith(".mp3")])

##Globals
position = 0

#### MAIN MUSIC CONTROL FUNCTIONS ####
def play_track():
    track = PATH + tracks[position]
    mixer.music.load(track)
    mixer.music.play()
```
--- /task ---

--- task ---
Test track plays ny running script and typing `play_track()` in the shell or adding it to the end of the file.
--- /task ---

--- task ---
Create a function to stop the track

```python
def stop_track():
    mixer.music.stop()
```
--- /task ---

--- task ---
Function to pause a track

```python
def pause_track():
    mixer.music.pause()
```
--- /task ---

--- task ---
Need to unpause the track. Easiest to use the `play_track()` function again. Check to see if the player is busy (a track has been loaded and paused), if not load the track, otherwise unpause the track.

```python
def play_track():
    if mixer.music.get_busy() == 0:
        track = PATH + tracks[position]
        mixer.music.load(track)
        mixer.music.play()
    else:
        mixer.music.unpause()
```
--- /task ---

--- task ---
Can test all functions in the shell to make sure you can start, stop, pause and unpause track `0`
--- /task ---

--- task ---
Next and previous track functions. For previous track, check the position isn't 0 and if it is set the position to the last track in the list. For the next track, check position isn't last in list and if it is set it to `0`. Otherwise increment/decrement the position global, stop the track and call the `play_track()` function

```python
def previous_track():
    """Decrements the posiion variable and plays the next track"""
    global position
    if position == 0:
        position = len(tracks) - 1
    else:
        position -= 1
    mixer.music.stop()
    play_track()

    
def next_track():
    """Increments the position variable and plays the next track"""
    global position
    if position == len(tracks) - 1:
        position = 0
    else:
        position += 1
    mixer.music.stop()
    play_track()
```
--- /task ---

--- task ---
Shuffle the tracks by shuffling the list of tracks. Need a new import for this.

```python
def shuffle_tracks():
    """Randomises the tracks list and starts playing from 0"""
    global position
    position = 0
    stop_track()
    shuffle(tracks)
    play_track()
```
--- /task ---

--- task ---
Change the volume of the track. Make the function take a value from 0-100. Turn it into a value between 0 and 1
```python
def change_volume(volume):
    volume = int(volume) / 100
    mixer.music.set_volume(volume)
```
--- /task ---

--- task ---
Test all functions in the shell
--- /task ---
