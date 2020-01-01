# Android 音频架构基础 - 使用AudioRecord录制PCM数据

## 获取权限

获取设备的读写及录制权限

oncreate中:

```java
if (ContextCompat.checkSelfPermission(AudioActivity.this, Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED
                || ContextCompat.checkSelfPermission(AudioActivity.this, Manifest.permission.RECORD_AUDIO) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(AudioActivity.this, new String[] {Manifest.permission.WRITE_EXTERNAL_STORAGE, Manifest.permission.RECORD_AUDIO}, AUDIO_REQUEST_CODE);
}
```

overrider `onRequestPermissonsResult`方法

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


