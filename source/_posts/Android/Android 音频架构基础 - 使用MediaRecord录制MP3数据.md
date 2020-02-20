# Android 音频架构基础 - 使用MediaRecord录制MP3数据

## 录制流程

MediaRecord的整个录制过程可以用下面这个状态机表示，内部封装了编解码过程。

<center>

<img src="https://www.guohezuzi.cn/public/img/blog/mediarecorder_state_diagram.png" width="60%" />

</center>

init -> setDataSoureConfigued -> prepared -> recording -> release

## 具体代码

```java
Log.d(TAG, "record from media record");
mMediaRecorder.reset();
//initialized
mMediaRecorder.setAudioSource(MediaRecorder.AudioSource.MIC);
//设置输出文件容器封装格式
mMediaRecorder.setOutputFormat(MediaRecorder.OutputFormat.THREE_GPP);
mMediaRecorder.setAudioSamplingRate(SAMPLE_RATE_IN_HZ);
mMediaRecorder.setAudioChannels(2);
String mediaRecordFileName = getExternalFilesDir(Environment.DIRECTORY_MUSIC) + MEDIA_RECORD_FILE_NAME + timestamp + ".mp3";
//DataSourceConfigured
mMediaRecorder.setOutputFile(mediaRecordFileName);
mMediaRecorder.setAudioEncoder(MediaRecorder.AudioEncoder.AAC);
//prepare
try {
    mMediaRecorder.prepare();
} catch (IOException e) {
    e.printStackTrace();
    Log.e(TAG, e.getMessage());
}
//recording
mMediaRecorder.start();
//released
mMediaRecorder.release();
```
