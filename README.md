# Obtaining a License

# Integrated SDK

## Step 1: Add Maven repository

Configure the Lexo Maven Service in the repositories of the build.gradle file in the project root
directory.

```text   
allprojects {
    repositories {
        maven { url = uri("https://maven.lexo.video/repository/maven-releases/") }
    }
}
```

## Step 2: Add SDK dependencies

Add the LexoPlayer SDK dependency to the dependencies in the build.gradle file in the module
directory.

```text
dependencies {
    // Replace x.x.x with the latest version number
    implementation("video.lexo:player:x.x.x")
}
```

## Step 3: Declare permissions

Declare the permissions required by the SDK in AndroidManifest.xml.

```text
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```

WRITE_EXTERNAL_STORAGE and READ_EXTERNAL_STORAGE are non-essential permissions and can be set
according to your actual needs:  
If you do not need to play audio and video resources on external storage, nor store downloaded
videos on external storage, you do not need to apply for this permission.
If you need to declare the WRITE_EXTERNAL_STORAGE permission, refer
to <a href="https://developer.android.com/training/data-storage/use-cases">Android Storage Use Cases
and Best Practices</a>

# Quick Start

## Step 1: Initialize the SDK

The initialization operation is lightweight. It is recommended Application#onCreate to perform
initialization in to ensure the initialization order.

```kotlin
val cacheDirPath = this.cacheDir.absolutePath.plus("/lexo_video_cache")
val cacheConfig = CacheConfig.Builder().setCacheDirPath(cacheDirPath)
    .setMaxCacheSize(300 * 1024 * 1024)
    .setCacheKeyGenerator(object : CacheConfig.CacheKeyGenerator {
        override fun generate(url: String): String {
            /**
             * Generates a cache key for local video content.
             * Current implementation uses the original URL as cache key directly.
             * For production use, consider implementing:
             * 1. URL normalization (remove query parameters/fragments)
             * 2. Hashing (MD5/SHA-1) for consistent key length
             */
            return url.split("?").first()
        }
    }).build()

/**
 *  It is recommended that you enable logging to facilitate debugging and troubleshooting.
 *  Please be sure to turn off logging in the online version to reduce performance overhead.
 */
val logConfig = LogConfig().apply {
    this.enableAllLog()
}
val licenseListener = object : LicenseListener {

    override fun onLoadSuccess(licenseId: String) {
        Log.i(TAG, "onLoadSuccess licenseId:$licenseId")
    }

    override fun onLoadFailed(message: String) {
        Log.e(TAG, "onLoadFailed message:$message")
    }

    override fun onUpdateSuccess(licenseId: String) {
        Log.i(TAG, "onUpdateSuccess licenseId:$licenseId")
    }

    override fun onUpdateFailed(licenseId: String, message: String) {
        Log.e(TAG, "onUpdateFailed licenseId:$licenseId message:$message")
    }
}
val lexoConfig = LexoConfig.Builder()
    .setAppId("YOUR_APP_ID")
    .setLicenseUrl("file:///android_asset/license/lexo.lic")
    .setLicenseListener(licenseListener)
    .setLogConfig(logConfig)
    .setCacheConfig(cacheConfig)
    .build()
LexoEnv.setup(this, lexoConfig)
```

The detailed parameter description is shown in the following table.

| parameter       | type            | Is this field required? | illustrate                                                                                     |    
|:----------------|:----------------|:------------------------|:-----------------------------------------------------------------------------------------------|
| AppId           | String          | Required                | Application ID.                                                                                |
| LicenseUrl      | String          | Required                | License file path. Copy the license file to the app's assets folder and set LicenseUrl.        |
| LicenseListener | LicenseListener | Optional                | Get the status of License acquisition and renewal.                                             |
| CacheConfig     | CacheConfig     | Optional                | Video cache related configuration.                                                             |
| LogConfig       | LogConfig       | Optional                | Log related configuration.                                                                     |
| DebugMode       | Boolean         | Optional                | Default is false. Whether to enable debug mode. Do not set this to true in the online version. |

## Step 2: Create the player

To ensure the playback effect, it is recommended to create a new LexoPlayer instance each time you
play, rather than reusing the LexoPlayer instance.

```kotlin
val player = LexoPlayer()
```

## Step 3: Set up the display view

LexoPlayer supports associating the following two views:

### TextureView

Declare in the layout file

```xml

<TextureView android:id="@+id/textureView" 
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

Call setTextureView to complete the association between the player and TextureView.

```kotlin
val textureView = findViewById(R.id.textureView)
player.setTextureView(textureView)
```

### SurfaceView

Declare in the layout file

```xml

<SurfaceView android:id="@+id/surfaceView" 
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

Call setSurfaceView to complete the association between the player and SurfaceView.

```kotlin
val surfaceView = findViewById(R.id.surfaceView)
player.setSurfaceView(surfaceView)
```

## Step 4: Set the playback source

LexoPlayer supports playing local videos and network streaming videos. The SDK provides a variety of
MediaSource for different purposes.

### UrlMediaSource

```kotlin
val urlMediaSource = UrlMediaSource.Builder()
    .setUrl("https://www.example.com/lexo_test.mp4")
    .build()
player.setMediaSource(urlMediaSource)
player.prepare()
player.play()
```

```kotlin
val urlMediaSource = UrlMediaSource.Builder()
    .setUrl("file:///data/user/0/video.lexo.player.demo/video/lexo_test.mp4")
    .build()
player.setMediaSource(urlMediaSource)
player.prepare()
player.play()
```

### EncryptedMediaSource(Only supports hls resources)

```kotlin
val encryptedMediaSource = EncryptedMediaSource.Builder()
    .setCryptKey("CRYPT_KEY")
    .setCryptIV("CRYPT_IV")
    .setMediaSource(
        UrlMediaSource.Builder().setUrl("https://www.example.com/lexo_test_encrypted.m3u8").build()
    )
    .build()
player.setMediaSource(encryptedMediaSource)
player.prepare()
player.play()
```

### MultiResolutionMediaSource(Only supports hls resources)

```kotlin
val highResolutionSource = UrlMediaSource.Builder()
    .setUrl("https://www.example.com/lexo_test_480p.m3u8")
    .setResolution(Resolution.High)
    .build()
val superHighResolutionSource = UrlMediaSource.Builder()
    .setUrl("https://www.example.com/lexo_test_720p.m3u8")
    .setResolution(Resolution.SuperHigh)
    .build()
val extremelyHighResolutionSource = UrlMediaSource.Builder()
    .setUrl("https://www.example.com/lexo_test_1080p.m3u8")
    .setResolution(Resolution.ExtremelyHigh)
    .build()

val multiResolutionMediaSource = MultiResolutionMediaSource.Builder()
    .setMediaSources(
        listOf(
            highResolutionSource,
            superHighResolutionSource,
            extremelyHighResolutionSource
        )
    )
    .build()
player.setMediaSource(multiResolutionMediaSource)
player.prepare()
player.play()
```

## Step 5: Release the player

If the video playback ends or the user leaves the video playback page, the playback must be stopped
in time and the LexoPlayer instance must be released.
Calling release method can release the hardware decoder usage, memory usage, and network usage of
LexoPlayer, which can effectively help users save power.

```kotlin
player.release()
```

# Basic functions

## Prepare Resources

Move the player out of idle state and the player will start loading media and acquire resources
needed for playback.

```kotlin
player.prepare()
```

## Start Playing

Resume playback immediately after the player is in the ready state.

```kotlin
player.play()
```

## Pause playback

Pause playback and call play again to resume from the paused state to the playing state.

```kotlin
player.pause()
player.play()
```

## Stop

Calling this method will cause the playback state to transition to idle state and the player will
release the loaded media and resources required for playback. The player instance can still be used
by calling prepare again, and release must still be called on the player if it's no longer required.

```kotlin
player.stop()
```

## Release

Releases the player. This method must be called when the player is no longer required. After
release, you can set player to null to prevent it from being called again.

```kotlin
player.release()
player = null
```

## Seek to the specified position to play

Seeks to a position specified in milliseconds in current content.

```kotlin
player.seekTo(2000, object : SeekCompleteListener {
    override fun onComplete(success: Boolean) {
        //This method will be called back after the Seek operation is completed
    }
})
```

## Start playing from the specified time

Before calling setMediaSource, use the setStartPositionMs method to specify the start time of
playback, which can be used to implement functions such as starting playback from a specified time
or skipping the beginning of a video.

```kotlin
player.setStartPositionMs(10 * 1000L)
```

## Get video width and height

Call getVideoSize to get the width and height of video if the player has prepared the resources.

```kotlin
val videoSize = player.getVideoSize()
val width = videoSize.width
val height = videoSize.height
```

## Get the current playback progress

Call getCurrentPosition to get the current playback position, in milliseconds.

```kotlin
val currentPosition = player.getCurrentPosition()
```

Get the timing progress through callback.

```kotlin
player.setPositionUpdateInterval(1000L)
player.addListener(object : ILexoPlayer.Listener {
    override fun onPlaybackTimeUpdated(currentPlaybackTimeMs: Long) {

    }
})
```

## Get video duration

Call getDuration to get the duration of the current content if the player has prepared the
resources, in
milliseconds.

```kotlin
val duration = player.getDuration()
```

## Get the current playback progress

Call getBufferedPosition to get an estimate of the position in the current content up to which data
is buffered, in
milliseconds.

```kotlin
val bufferedPosition = player.getBufferedPosition()
```

## Loop Playback

Sets whether to repeat the currently playing infinitely during playback.

```kotlin
player.setLooping(true)
val isLooping = player.isLooping()
```

## Playback Speed

Sets the playback speed. Must be greater than zero.

```kotlin
player.setPlaybackSpeed(0.5f)
```

## Adjust the volume

Sets the audio volume, valid values are between 0 (silence) and 1, inclusive.

```kotlin
player.setPlaybackVolume(0.5f)
```

## Screen always on

Call View#setKeepScreenOn(boolean) to set the screen to stay on and View#getKeepScreenOn get whether
it stays on.

```kotlin
textureView.setKeepScreenOn(true)
val isKeepScreenOn = textureView.getKeepScreenOn()
```

## Get the current resolution

Call getCurrentResolution to get the current resolution of the player if the player has prepared the resources.

```kotlin
val currentResolution = player.getCurrentResolution()
```

## Get the clarity list

Call getAvailableResolutions to get all the resolutions currently available for the player if the player has prepared the resources.

```kotlin
val availableResolutions = player.getAvailableResolutions()
```

## Switch clarity

Switches the player to the specified video resolution.

```kotlin
val resolution = Resolution.ExtremelyHigh
player.setResolutionListener(object : ILexoPlayer.ResolutionListener {

    /**
     * Called each time when [ILexoPlayer.getCurrentResolution] changes.
     *
     * @param resolution Current resolution of the player.
     * @param action The action that caused the resolution changes.
     */
    override fun onChanged(resolution: Resolution, action: ResolutionChangeAction) {

    }

    /**
     * Called when resolution switching failed.
     *
     * @param resolution The resolution that failed to switch.
     * @param errMsg Specific error message.
     */
    override fun onSwitchFailed(resolution: Resolution, errMsg: String) {
    }
})
player.switchResolution(resolution, true, 10 * 1000L)
```

## Clarity Enumeration

The Resolution enumeration is shown in the following table.

| key           | video clarity | description                              |    
|:--------------|:--------------|:-----------------------------------------|
| L_Standard    | 240p          | Low Definition (LD) / Very Basic Quality |
| Standard      | 360p          | Standard Definition (SD)                 |
| High          | 480p          | Enhanced Definition (ED) / High SD       |
| H_High        | 540p          | Intermediate Definition / Pre-HD         |
| SuperHigh     | 720p          | HD (High Definition)                     |
| ExtremelyHigh | 1080p         | Full HD                                  |

## Monitor playback status

```kotlin
player.addListener(object : ILexoPlayer.Listener {

    override fun onPlaybackStateChanged(state: PlaybackState) {
        when (state) {
            /**
             * The player is idle, meaning it holds only limited resources. The player must be [LexoPlayer.prepare]
             * before it will play the media.
             */
            PlaybackState.STATE_IDLE -> {

            }
            /**
             * The player is able to immediately play from its current position. The player will be playing if
             * [LexoPlayer.play] is called, and paused otherwise.
             */
            PlaybackState.STATE_PREPARED -> {

            }
            /** Playback is actively playing. */
            PlaybackState.STATE_PLAYING -> {

            }
            /** Playback is paused but ready to play. */
            PlaybackState.STATE_PAUSED -> {
            }
            /** The player has finished playing the media. */
            PlaybackState.STATE_COMPLETED -> {

            }
            /** Playback is stopped due a fatal error and can be retried. */
            PlaybackState.STATE_ERROR -> {

            }
        }
    }

    /**
     * Called when a frame is rendered for the first time since setting the surface, or since the
     * renderer was reset, or since the stream being rendered was changed.
     *
     * @param costTimeMs The time taken (in milliseconds) from the start of playback until the first frame is rendered.
     */
    override fun onFirstFrameRendered(costTimeMs: Long) {}

    /**
     * Called when the playback time is updated.
     *
     * @param currentPlaybackTimeMs The current playback position in milliseconds.
     */
    override fun onPlaybackTimeUpdated(currentPlaybackTimeMs: Long) {}

    /**
     * Called when the player enters a buffering state.
     *
     * @param afterFirstFrame Indicates whether the buffering occurred before or after the first frame is rendered.
     * @param action The action that caused the buffering.
     *
     */
    override fun onBufferStart(afterFirstFrame: Boolean, action: BufferStartAction) {}

    /**
     * Called when the player exits the buffering state and resumes playback.
     */
    override fun onBufferEnd() {}

    /**
     * Called each time when [ILexoPlayer.getVideoSize] changes.
     *
     * @param size The new size of the video.
     */
    override fun onVideoSizeChanged(size: VideoSize) {}

    /**
     * Called when an error occurs. The playback state will transition to [PlaybackState.STATE_IDLE]
     * immediately after this method is called. The player instance can still be used,
     * and [ILexoPlayer.release] must still be called on the player should it no longer be required.
     *
     * @param error The error.
     */
    override fun onError(error: Error) {}
})
```

# Advanced Features

## Preloading

Preloading refers to downloading the header data of the video to be played in advance before
starting to play, so as to achieve fast start of playback and improve the playback experience.
Preloading is Premium Feature. Please make sure you have purchased a Premium License.

### Adding a preload task

```kotlin
val url = "https://www.example.com/lexo_test_encrypted.m3u8"
val taskId = md5(url)
val task = PreloadTask.Builder().setId(taskId)
    .setSource(UrlMediaSource.Builder().setUrl(url).build())
    .setPreloadSize(800 * 1024L)
    .build()
LexoPlayer.addPreloadTask(task, object : PreloadListener {

    override fun onStart() {

    }

    override fun onPause() {

    }

    override fun onResume() {

    }

    override fun onCanceled() {

    }

    override fun onSuccess() {

    }

    override fun onFailed(errMsg: String) {

    }
})
```

### Cancel preload task by taskId

```kotlin
val url = "https://www.example.com/lexo_test_encrypted.m3u8"
val taskId = md5(url)
LexoPlayer.cancelPreloadTask(taskId)
```

### Cancel all preload tasks

```kotlin
LexoPlayer.cancelPreloadTasks()
```

### Set the number of concurrent preload tasks

```kotlin
LexoPlayer.setPreloadParallelNum(1)
```

## Add external subtitles

External subtitles refer to subtitle files that are separated from video files. Users can import
them on demand during playback. The SDK supports adding external subtitles in WebVTT, SRT, SSA and
TTML formats. External Subtitles are Premium Feature. Please make sure you have purchased a Premium
License.

### Create a MediaSource with subtitles

```kotlin
val subtitles = mutableListOf<Subtitle>()
subtitles.add(
    Subtitle.Builder().setUrl("https://www.example.com/sub/sub_en.vtt")
        .setMimeType(Subtitle.MIME_TYPE_VTT)
        .setLanguage("en").build()
)
subtitles.add(
    Subtitle.Builder().setUrl("https://www.example.com/sub/sub_ar.vtt")
        .setMimeType(Subtitle.MIME_TYPE_VTT)
        .setLanguage("ar").build()
)
val mediaSource = UrlMediaSource.Builder()
    .setUrl("https://www.example.com/lexo_test.mp4")
    .setSubtitles(subtitles)
    .build()
player.setMediaSource(mediaSource)
player.prepare()
player.play()
```

### Add subtitle callback

Before calling prepare, call addSubtitleListener to set subtitle-related callbacks.

```kotlin
player.addSubtitleListener(object : ILexoPlayer.SubtitleListener {

    /**
     * Called when the subtitle content changes.
     */
    override fun onSubInfo(info: String) {}

    /**
     * Called when the subtitle loaded.
     */
    override fun onSubLoadCompleted(language: String) {}

    /**
     * Called when the subtitle load failed.
     */
    override fun onSubLoadError(language: String, errMsg: String) {}

    /**
     * Called when the subtitle switching completed.
     */
    override fun onSubSwitchCompleted(language: String) {}

    /**
     * Called when the subtitle switching failed.
     */
    override fun onSubSwitchError(language: String, errMsg: String) {}
})
```

### Toggle subtitles

Toggle subtitles in different languages.

```kotlin
player.switchSubtitleLanguage("ar")
```

