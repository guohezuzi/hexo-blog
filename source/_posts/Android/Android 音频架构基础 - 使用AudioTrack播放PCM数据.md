# Android 音频架构基础 - 使用AudioTrack播放PCM数据

## 介绍

SDK提供的更底层的API，适合低延迟的播放，一般需要配合编码器一起使用。

## 使用

1.初始化audioTrack，参数:播放类型、位宽、音频帧大小、流类型（音频流或静态buff）、当前音频session id。

2.从PCM文件流中读取数据,调用play方法播放。

```java
AudioTrack audioTrack = new AudioTrack(new AudioAttributes.Builder().setLegacyStreamType(AudioManager.STREAM_MUSIC).build(),
        new AudioFormat.Builder().setEncoding(AudioFormat.ENCODING_PCM_16BIT).build(), MIN_BUFFER_SIZE,
        AudioTrack.MODE_STREAM, AudioManager.AUDIO_SESSION_ID_GENERATE);
Log.d(TAG, "play from audio track");
try {
    FileInputStream fileInputStream = new FileInputStream(getExternalFilesDir(Environment.DIRECTORY_MUSIC) + AUDIO_RECORD_FILE_NAME + timestamp + ".pcm");
    byte[] buff = new byte[MIN_BUFFER_SIZE];
    while (fileInputStream.read(buff) != -1) {
        audioTrack.write(buff, 0, buff.length);
        audioTrack.play();
    }
} catch (Exception e) {
    e.printStackTrace();
}
```

## 底层逻辑(framework层)

### new AudioTrack()

[Source code](https://cs.android.com/android/platform/superproject/+/master:frameworks/base/media/java/android/media/AudioTrack.java;bpv=1;bpt=1;l=598?q=audiotrack)

#### 分析

1.对各个参数的校验及处理

2.对一些成员变量的初始化

其中这里我们深入看下`mAttributes`的初始化，引入一些Playerback mode的点

```java
// Check if we should enable deep buffer mode
if (shouldEnablePowerSaving(mAttributes, format, bufferSizeInBytes, mode)) {
    mAttributes = new AudioAttributes.Builder(mAttributes)
        .replaceFlags((mAttributes.getAllFlags()
                | AudioAttributes.FLAG_DEEP_BUFFER)
                & ~AudioAttributes.FLAG_LOW_LATENCY)
        .build();
}
```

1 Deep buffer Playback:音频文件是在**AP侧解码成PCM文件**，然后再送到ADSP中处理，音效处理在AP侧或者ADSP中进行。
Playback mode in which PCM data is sent to the aDSP, postprocessed, and rendered to output sound device; audio effects can also be applied in the ARM or aDSP.
FLAG： AUDIO_OUTPUT_FLAG_DEEP_BUFFER

2 Low Latency Playback : 和Deep buffer Playback方式类似，但是它所分配的buffer更小些，并且在ADSP侧只做很少或者基本不做处理， 主要是播放一些对延迟要求较高的音频，
Playback mode similar to deep buffer that uses a smaller buffer size and minimal or no
postprocessing in the aDSP so that the PCM stream is rendered to the output sound
device
Use cases – Touchtone, gaming audio
FLAG：AUDIO_OUTPUT_FLAG_FAST

3 Offload Playback: 音频解码部分的工作是在ADSP中完成，**AP侧只负责把音频数据送到ADSP中**，送出去后，AP侧会进行休眠，ADSP中会分配一块较大的buffer去处理此数据，在ADSP中进行解码，音效的处理工作，在ADSP解码器处理完数据之前，它会唤醒AP侧去送下一包数据。
Playback mode in which a large-sized buffer is sent to the aDSP and the APSS goes to sleep; the aDSP decodes, applies postprocessing effects, and outputs the PCM data to the physical sound device. Before the aDSP decoder input runs out of data, it interrupts the APSS to wake up and send the next set of buffers.
Examples – MP3, AAC, FLAC 24 bit, 24 bit wav playback
FLAG：AUDIO_OUTPUT_FLAG_DIRECT，AUDIO_OUTPUT_FLAG_COMPRESS_OFFLOAD，AUDIO_OUTPUT_FLAG_NON_BLOCKING
参考: [Android Audio Playback Mode](https://blog.csdn.net/ch853199769/article/details/79916166)

存疑：offload playback vs deep buffer playback

AP侧不处理有什么影响呢？这两个应用场景是什么样的呢？一般我们手机播放音乐playback mode是什么呢？

deep-buffer的好处app自己解码，app可以保证在所有手机上都能播，app跨平台也好维护。
offload的好处是通过mediaplayer来播，android系统会查看底层是否支持硬解，如果支持就走offload，由于adsp解码会省电些，但是app控制不了，比如某个手机播放不了某些歌曲。

手机播放音乐playback mode两种都有，主流app一般是deep-buffer。

3.native setup

4.更新audioTrack状态

```java
if (mDataLoadMode == MODE_STATIC) {
    mState = STATE_NO_STATIC_DATA;
} else {
    mState = STATE_INITIALIZED;
}
```

5.在service注册一个player

```java
protected void baseRegisterPlayer() {
    if (!USE_AUDIOFLINGER_MUTING_FOR_OP) {
        IBinder b = ServiceManager.getService(Context.APP_OPS_SERVICE);
        mAppOps = IAppOpsService.Stub.asInterface(b);
        // initialize mHasAppOpsPlayAudio
        updateAppOpsPlayAudio();
        // register a callback to monitor whether the OP_PLAY_AUDIO is still allowed
        mAppOpsCallback = new IAppOpsCallbackWrapper(this);
        try {
            mAppOps.startWatchingMode(AppOpsManager.OP_PLAY_AUDIO,
                    ActivityThread.currentPackageName(), mAppOpsCallback);
        } catch (RemoteException e) {
            Log.e(TAG, "Error registering appOps callback", e);
            mHasAppOpsPlayAudio = false;
        }
    }
    try {
        mPlayerIId = getService().trackPlayer(
                new PlayerIdCard(mImplType, mAttributes, new IPlayerWrapper(this)));
    } catch (RemoteException e) {
        Log.e(TAG, "Error talking to audio service, player will not be tracked", e);
    }
}
```

### write()

[Source code](https://cs.android.com/android/platform/superproject/+/master:frameworks/base/media/java/android/media/AudioTrack.java;bpv=1;bpt=1;l=2491)

1.对各个参数的校验及处理

这里说明一下第四个参数当写入的队列满了是否阻塞，我们平常调用的不指定是否阻塞，默认就是阻塞的。同时这里对stop状态下做了一个处理

```java
if (!blockUntilOffloadDrain(writeMode)) {
        return 0;
}

private boolean blockUntilOffloadDrain(int writeMode) {
    synchronized (mPlayStateLock) {
        while (mPlayState == PLAYSTATE_STOPPING || mPlayState == PLAYSTATE_PAUSED_STOPPING) {
            if (writeMode == WRITE_NON_BLOCKING) {
                return false;
            }
            try {
                mPlayStateLock.wait();
            } catch (InterruptedException e) {
            }
        }
        return true;
    }
}
```

如果是stoping状态，blocking write -> wait 、not blocking -> reture 0

2.调用native write

### play()

[Source code](https://cs.android.com/android/platform/superproject/+/master:frameworks/base/media/java/android/media/AudioTrack.java;bpv=1;bpt=1;l=2286)

对audioTrack状态判断后，调用native play

## 后记

作为Java程序员native层代码未做深入分析，后续补上，这里先梳理一下大致逻辑。

native层主要是AudioFlinger对多个AudioTrack的管理，不同的进程有不同的AudioTracker通过AudioFlinger管理，通过MixerThread将不同AudioTracker的数据混音后再输出
