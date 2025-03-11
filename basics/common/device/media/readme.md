# Use audio

​    	There are currently two mainstream methods for HTML5 audio playback, one is Audio tag playback, and the other is WebAudio binary playback.

​    	Audio is a dom element with a UI interface. On the mobile side, Audio is downloaded and played at the same time. It is suitable for files with relatively large sound files. However, Audio will have gesture restrictions on the mobile side. The gesture-requirement-for-media-playback attribute indicates that It can only be played if there is a user's gesture operation.

​    	WebAudio is a new form of sound playback that can load multiple sounds for synthesis. It decodes binary files into a format supported by the browser for playback. And using this interface, you can even realize the animation effect of the audio spectrum, giving the sound a synthesis function.

​    	Music and sound effects are commonly used basic elements in games. The LayaAir engine encapsulates WebAudio and Audio. On browsers that support WebAudio, WebAudio is used first, and on browsers that do not support WebAudio, Audio is used to maximize compatibility with audio in all browsers. The format support makes it more convenient for developers to directly play audio by calling the laya.media.SoundManager API interface.

### 1. The difference between the application of music and sound effects

​    	Music: refers to the background music used in the game. Use the playMusic method in the laya.media.SoundManager audio management class to play. Since it is background music, the playMusic method can only play one audio file at the same time.

​    	Sound effect: The playSound method in the laya.media.SoundManager audio management class is used, allowing multiple audio files to be played at the same time.

### 2. Audio compatibility preparation

​    	Since the compatibility of various browsers for audio playback issues is different, we must make preliminary compatibility preparations before starting the application.

(1) Use the "Format Factory" audio file conversion tool. Select 44100Hz, 96kbps for conversion.

(2) Keep audio files as small as possible, not only due to bandwidth limitations, but also the efficiency of browser audio decoding.

### 3. Audio volume control

​    	Sound volume control can be set through the setSoundVolume method in the laya.media.SoundManager audio management class:

```typescript
/**
 	* Set sound volume. Depending on the parameters, you can set the volume of the specified sound (background music or sound effects) or the volume of all sound effects (excluding background music).
 	* @param volume volume. The initial value is 1. Volume ranges from 0 (silent) to 1 (maximum volume).
 	* @param url	 (default = null)Sound playback address. Default is null. If it is empty, it means setting the volume of all sound effects (excluding background music). If it is not empty, it means setting the volume of the specified sound (background music or sound effects).
 	*/
	static setSoundVolume(volume: number, url: string = null): void {
    	if (url) {
        	SoundManager._setVolume(url, volume);
    	} else {
        	SoundManager.soundVolume = volume;
        	for (let i = SoundManager._channels.length - 1; i >= 0; i--) {
            	let channel = SoundManager._channels[i];
            	if (channel.url != SoundManager._bgMusic) {
                	channel.volume = volume;
            	}
        	}
    	}
	}
```

By setting the volume parameter, you can effectively control the volume of the sound file corresponding to the URL. The initial value is 1. Volume ranges from 0 (silent) to 1 (maximum volume).

### 4. Device mute control

If you use the device mute key to mute the audio automatically, the device will be muted. UseAudioMusic needs to be set to false.

```typescript
Laya.SoundManager.useAudioMusic = false;
```

### 5. Dealing with loss of focus

If it is not a full-screen game on a mobile phone, or interactive in a browser. It may cause the game to lose focus after switching out of the game, and after losing focus, the audio will stop playing. This is caused by the browser mechanism. Developers have two ways to avoid it.

One is to set autoStopMusic to false in the entry file.

```typescript
……
//Whether to automatically stop background music after losing focus. false will not automatically stop and will continue to play. true means automatic stop
Laya.SoundManager.autoStopMusic = false;
……
```

If this is not set, it will stop when focus is lost. Developers need to control the loss and recovery of focus by listening for the loss of stage focus and the gain of stage focus (on the mini-game platform, it also depends on the platform cut-out rules of each mini-game), and the restoration in the browser The reference code is as follows:

```typescript
……
//Loop play_sound
_sound.play(0);

//Handling of losing stage focus (cutting out of the game)
Laya.stage.on(Laya.Event.BLUR, this, () => {
	_sound.stop();
});
//Processing of obtaining stage focus (switching back to the game)
Laya.stage.on(Laya.Event.FOCUS, this, () => {
	_sound.play(0);
});
……
```



