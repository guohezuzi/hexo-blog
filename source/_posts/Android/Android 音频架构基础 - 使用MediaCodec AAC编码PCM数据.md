# Android 音频架构基础 - 使用MediaCodec AAC编码PCM数据

## 工作原理

1. input： 从输入源(文件流、网络流...)读取pcm文件，通过一个默认长度为4的**循环队列**获取音频数据，交给Codec编码

2. ouput：从Codec中获取编码好的数据，通过一个循环队列将编码好的数据写入到对应Clinet(文件、网络传书)

3. codec:：对数据进行编码。可通过一些参数配置：`MediaFormat.KEY_BIT_RATE  `、`MediaFormat.KEY_AAC_PROFILE` 、` MediaFormat.KEY_MAX_INPUT_SIZE`...

<center>
<img src="https://www.guohezuzi.cn/public/img/blog/mediacodec_buffers.png" width="80%" />    
</center>

## 代码实现

### 构建MedicCodec

一些注意点

1. 必须设置码率(BIT_RATE)，大小可参考[音频相关的基本概念](https://www.guohezuzi.cn/article/android-audio-basic-concepts#%E6%AF%94%E7%89%B9%E7%8E%87bit_rate)

2. 默认INPUT_SIZE为4096byte，如有修改需要进行配置，否则会丢失数据。

3. 

```java
//Config encode codec
MediaFormat encodeFormat = MediaFormat.createAudioFormat(MIMETYPE_AUDIO_AAC, SAMPLE_RATE_IN_HZ, 2);
encodeFormat.setInteger(MediaFormat.KEY_BIT_RATE, BIT_RATE);
encodeFormat.setInteger(MediaFormat.KEY_AAC_PROFILE, MediaCodecInfo.CodecProfileLevel.AACObjectLC);
//setting input buff size
encodeFormat.setInteger(MediaFormat.KEY_MAX_INPUT_SIZE, MIN_BUFFER_SIZE);
mediaCodec = MediaCodec.createEncoderByType(MIMETYPE_AUDIO_AAC);
```

### 同步方法

### 异步方法
