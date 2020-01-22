# Android 音频架构基础 - 使用AudioTrack播放PCM数据

## 介绍

SDK提供的更底层的API，适合低延迟的播放，一般需要配合解码器一起使用。

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
