---
id: extension-multimedia
title: Multimedia
---

```inform7
Include Vorple Multimedia by Juhana Leinonen.
```

## Including media files

Media files (images and audio) should be declared in the story source so that Inform knows to include them in the release version.

```inform7
Release along with the file "whatever.png".
```

The files should be placed in the Materials directory, as per chapter 23.7. in Writing with Inform.

Note that we shouldn't use the "Figure of ..." or "Sound of ..." directives described in chapter 22 of Writing with Inform. Files declared this way won't be included correctly in the release folder.


## Images

Images can be displayed with the "place an image" phrase:

```inform7
place an image "pic.jpg" with the description "Example image";
```

The description is shown in standard interpreters instead of the picture and read aloud by screen readers, so it should be a compact but accurate description of what the image depicts. Especially if seeing the image is relevant to the story or to solving a puzzle, the description should give the same information to those who can't see the image.

By default the image is displayed left-aligned. The position can be changed by giving it as a parameter:

```inform7
place an image "pic.jpg" with the description "Example image", centered;
```

The possible values are centered, aligned left, aligned right, floating left or floating right. Floating means that the image is set to the left or to the right and the text is wrapped around it, in contrast to left or right aligned where the remaining space is left blank.

Images should be in jpg, png or gif format.

The images can be named by adding `called "something"`:

```inform7
place an image "pic.jpg" called "example" with the description "Example image";
```

Note that in HTML the image is wrapped in a div that receives the image's name as a class. The above code produces the following HTML:

```html
<div class="vorple-image example">
    <img src="pic.jpg" alt="Example image">
</div>
```

Tip: When releasing the project, the file "Cover.jpg" or "Cover.png" is automatically included in the same place where other project files are. Therefore we can always show the cover image with:
	
```inform7
place an image "Cover.jpg" with the description "Cover image";
```

### Images from the Internet

The `place an image` phrase accepts direct web addresses of images as well:
	
```inform7
place an image "http://example.com/image.jpg";
```

Some important caveats:
	
* Things on the Internet tend to disappear over time: they get removed, moved, or the web site just ceases to exists. If you don't have control over the image source, it's better to just download the image and include it with other story resources - within the limits of copyright permissions, of course.
* Don't include an image as a link if you don't have a permission to do so! It's called 'hotlinking' and is generally frowned upon as it causes sometimes expensive traffic to the original web site. Furthermore the owner of the hotlinked web site might change the image to something else, which might be more than awkward to the author. This obviously doesn't include image sharing web sites that are explicitly meant for this kind of use.


### Preloading images

Images can be preloaded either individually or as a list:

```inform7
When play begins:
    preload image "pic.jpg";
    preload images { "pic1.jpg", "pic2.png" }.
```

Preloading images makes them appear immediately when they are later included on the page. Otherwise the images are loaded only when they are first displayed which may take some time with slower connections, resulting in a noticeable delay between when they should be shown and when they have loaded and actually appear.

Images must be preloaded inside a rule, most commonly in a When play begins rule. We can save bandwidth by preloading in later stages when the story is closer to the point when it should display the image, but note that starting to preload at the same time when the image is displayed is too late and preloading images that are shown right when the story begins is not useful.


## Audio

There are two types of audio: sound effects and music. The main difference is that multiple sound effects can be played at the same time, and at the same time as background music is playing. With music only one track can be playing at the same time, and starting to play another music file the previous one will stop.

Audio files should be in either mp3 or ogg format.

```inform7
play sound effect file "bang.mp3";
play music file "horns.mp3";
```

By default the files are played once and then stopped. They can also be set to loop:

```inform7
play music file "background.mp3", looping;
```

Starting to play a music file that's already playing doesn't do anything other than set the looping status. In other words, if the file "mozart.mp3" is already playing and we try to play it again:

```inform7
play music file "mozart.mp3", looping;
```

...then nothing happens, except that when the music ends it starts to loop. Conversely, if we leave out the 'looping' option then the music will not loop when it ends, even if it was originally set to loop.

If we do want the music to always start playing from the beginning even when it's already playing, we can command:

```inform7
play music file "mozart.mp3", always from the start;
```

Starting a new music track while a different track is playing will fade out the old track before starting the new one. The old track fades out during one second and then waits another second to play the next one.

Once playing the sounds can be stopped with the following phrases:

```inform7
stop sound effects;
stop music;
stop all audio;
```

(Naturally stopping just the sound effects won't affect music, and vice versa.) Stopping music or all audio also clears the playlist (see the [Playlists](#playlists) chapter.)

Whether audio is currently playing can be tested with these phrases:
	
```inform7
if music is playing: ...
if a sound effect is playing: ...
if any audio is playing: ...
if the audio file called "bigband.mp3" is playing: ...
```

An audio file is considered "playing" even when it's still loading and hasn't actually started to play yet. `If ... is playing` only checks for audio that is initiated by Vorple.

The currently playing music file can be retrieved with `the currently playing music file`:
	
```inform7
if the currently playing music file is "elvis.mp3": ...
```


### Audio restrictions in browsers

Due to restrictions set by most modern browsers, audio will not play unless the user has interacted with the page in some way, be it clicking or tapping on something or typing on the keyboard. This is to prevent ads and autoplaying videos from annoying the user, and mobile devices to preserve device battery.

This becomes an issue if we wish to start playing music immediately when the story starts. A workaround is to start playing only after the player has typed their first command, which works for most browsers and mobile devices.

```inform7
After reading a command when the turn count is 1:
    play music file "intro.mp3".
```

Another option is to make an intro screen (possibly with the Modal Windows extension) that forces the player to click a button or type a command to start the game and start playing the music.

The Safari browser on desktop Macs is extremely protective when it comes to playing sounds: it requires that the user initiates the sound directly (e.g. by pushing a button.) In Safari the above mentioned "tricks" to get the music playing won't work and there is currently no workaround. 


## Playlists

Playlists are collections of music that automatically play one after another. Playlists can be started with the following phrase:
	
```inform7
start the playlist { "one.mp3", "two.mp3", "three.mp3" };
```

The file "one.mp3" starts to play, when it finishes "two.mp3" starts automatically, and finally after that "three.mp3" plays. If we want the playlist to repeat from the start after it has played the last track:

```inform7
start the playlist { "one.mp3", "two.mp3", "three.mp3" }, looping;
```

We can also play the list in a random order:

```inform7
start the playlist { "one.mp3", "two.mp3", "three.mp3" }, shuffled;
```

Both options can be used together, but a looping playlist is shuffled only once. When the playlist finishes and restarts again, it replays in the same order as the first time.

If the first track in the playlist is already playing at the same time when the playlist is started, the track will keep playing normally to the end and then continue from the second track in the playlist. Otherwise any music track that's playing will stop and the playlist will start immediately from the first item.

If a music file is played manually with the "play music file..." phrase while the playlist is running, the playlist will continue from the next track after the manually started music ends. Starting a new playlist while the previous one is playing will replace the old playlist with the new one.

The playlist can be cleared with:
	
```inform7
clear the playlist;
```

Clearing the playlist doesn't immediately stop the currently playing track. Unless instructed otherwise, the track will continue but doesn't start a new track when it ends. Stopping music with `stop the music` or `stop all audio` phrases clears the playlist automatically.

Note that the music system uses the playlist internally for queueing songs, so remember to clear the playlist before playing a new track or starting a new playlist.

In other words, don't do this as it won't work:

```inform7
play the music file called "new.mp3";
clear the playlist;
```

Do this instead:

```inform7
clear the playlist;
play the music file called "new.mp3";
```

## Examples

### Serinette

*Basic example of playing music and sound effects*

The serinette (a type of music box) plays music when it opens and a sound effect when it's wound. We'll also show its picture when it's examined.

The example media files can be downloaded from https://vorple-if.com/resources.zip

<iframe width="780" height="450" src="https://embedded-snippet.borogove.app/?id=sjrxhh"></iframe>


### Port Royal Reggae

*Applying background music to different regions*

We'll spice up the Port Royal example from Writing with Inform with some background music.

When the player moves around the map the music will change according to the region they're in. We'll assign each region a music file, and the music changes only if the new room is in a different region than the one we just came from so that the same sound file won't start over when the player moves inside the same region. 

With small modifications the code could be used with individual rooms or scenes.

The first part is identical with the Port Royal 3 example in the Writing with Inform manual. Vorple-specific code starts from chapter 2 at the end.

The audio files used here can be downloaded from https://vorple-if.com/resources.zip

<iframe width="780" height="450" src="https://embedded-snippet.borogove.app/?id=kfg2tk"></iframe>

We could also set up playlists for each region instead of just one track.

```inform7
A region has a list of text called the background playlist.
The current region is a region that varies.

The background playlist of Inland is {"inland.mp3", "market.mp3"}.
The background playlist of Waterfront is {"waterfront.mp3", "seagulls.mp3", "fishermen.mp3"}.
The background playlist of Military Holdings is {"wind.mp3", "cannons.mp3"}.
The background playlist of Tavern is {"tavern.mp3"}.

Every turn when the map region of the location is not nothing and the map region of the location is not the current region and the background playlist of the map region of the location is not empty (this is the play background audio rule):
    start the playlist background playlist of the map region of the location, looping and shuffled;
    now the current region is the map region of the location.
    
[Every turn rules aren't run on the first turn so we'll run it manually.]
When play begins (this is the start initial background audio rule):
    if the map region of the location is not nothing and the background playlist of the map region of the location is not empty:
        follow the play background audio rule.		
```

(The tracks in the playlists are made up so unfortunately they're not included in the example resource package like the other tracks.)
