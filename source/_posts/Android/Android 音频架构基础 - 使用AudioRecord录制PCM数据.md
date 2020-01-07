# Android 音频架构基础 - 使用AudioRecord录制PCM数据

## 获取权限

1.获取设备的读写及录制权限，在Activity的onCreate方法中使用如下代码

```java
if (ContextCompat.checkSelfPermission(AudioActivity.this, Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED
                || ContextCompat.checkSelfPermission(AudioActivity.this, Manifest.permission.RECORD_AUDIO) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(AudioActivity.this, new String[] {Manifest.permission.WRITE_EXTERNAL_STORAGE, Manifest.permission.RECORD_AUDIO}, AUDIO_REQUEST_CODE);
}
```

2.处理授权失败的逻辑，我们这里直接退出当前Activity，overrider `onRequestPermissonsResult`方法

```java
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    switch (requestCode) {
        case AUDIO_REQUEST_CODE: {
            int grantTime = 0;
            for (int result : grantResults) {
                if (result == PackageManager.PERMISSION_GRANTED) {
                    grantTime++;
                }
            }
            if (grantTime == grantResults.length) {
                audioRecord = new AudioRecord(MediaRecorder.AudioSource.MIC,
                        SAMPLE_RATE_IN_HZ, CHANNEL_IN_STEREO, ENCODING_PCM_16_BIT,
                        MIN_BUFFER_SIZE);
                //initial
                mMediaRecorder = new MediaRecorder();
                mMediaPlayer = new MediaPlayer();
            } else {
                Toast.makeText(this, "该页面必须授权才能访问！", Toast.LENGTH_SHORT).show();
                finish();
            }
        }
    }
}
```

## 进行PCM数据的录制

1.构建AudioRecord对象

`new AudioRecord(MediaRecorder.AudioSource.MIC,
 SAMPLE_RATE_IN_HZ, CHANNEL_IN_STEREO, ENCODING_PCM_16_BIT,
 MIN_BUFFER_SIZE);`

 参数：录制源、采样率、通道数、位宽、音频帧大小。最小的音频帧通过`AudioRecord.getMinBufferSize`获取。

2.进行录制

通过调用`startRecording`进入录制状态后，通过调用`read`方法从流中读取数据。

在这里，我们新起一个线程不断从流中读取数据，同是通过一个变量控制是否停止。

```java
audioRecord.startRecording();
loopRecord = true;
new Thread(new Runnable() {
      @Override
      public void run() {
      try {
            byte[] buffer = new byte[MIN_BUFFER_SIZE];
            while (loopRecord) {
               int res = audioRecord.read(buffer, 0, MIN_BUFFER_SIZE);
               mDataOutputStream.write(buffer, 0, buffer.length);
               mDataSize += buffer.length;
               Log.i(TAG, "Capture " + res + " bytes!");
            }
      } catch (Exception e) {
           Log.e(TAG, Objects.requireNonNull(e.getMessage()));
      }
      }
}).start();
```

3.停止录制，并将PCM数据通过WAV封装。

默认大多数音乐软件是不支持PCM数据的直接播放的，我们可以通过wav封装进行简单处理。

```java
loopRecord = false;
audioRecord.stop();
Log.d(TAG, "stop record from audio record");
try {
    mDataOutputStream.flush();
    Log.d(TAG, "data size: " + mDataSize);
    mDataOutputStream.close();
    PcmToWavUtil pcmToWavUtil = new PcmToWavUtil(SAMPLE_RATE_IN_HZ, CHANNEL_IN_STEREO, ENCODING_PCM_16_BIT);
    pcmToWavUtil.pcmToWav(captureFilePath, getExternalFilesDir(Environment.DIRECTORY_MUSIC) + AUDIO_RECORD_FILE_NAME + timestamp + ".wav");
} catch (Exception e) {
    Log.e(TAG, Objects.requireNonNull(e.getMessage()));
}
```

wav封装的逻辑：在pcm数据前加入44byte的wav header，对数据的一些说明，参考：[WaveFormat](http://soundfile.sapp.org/doc/WaveFormat/)
