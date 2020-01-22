# Android 音频架构基础 - 使用MediaPlayer播放MP3数据

## 录制流程

类似[MediaRecord](https://www.guohezuzi.cn/article/android-audio-media-record) ，状态图如下

<center>
<img src="https://www.guohezuzi.cn/public/img/blog/mediaplayer_state_diagram.png" width="80%" />    
</center>

## 具体代码

```java
mMediaPlayer = new MediaPlayer();
mMediaPlayer.setDataSource(getExternalFilesDir(Environment.DIRECTORY_MUSIC) + MEDIA_RECORD_FILE_NAME + timestamp + ".mp3");
mMediaPlayer.prepare();
mMediaPlayer.setLooping(true);
mMediaPlayer.start();
//...
mMediaPlayer.pause();
```

idle -> inited -> prepared -> start -> pause -> stop ->release

## 底层逻辑

mediaplay(framework) -> mediaPlayerService(native) -> Nuplayer、OpenMax (native) 

-> (hal) -> (kernel)
