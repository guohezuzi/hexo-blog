# Android 音频架构基础 - 音频相关的基本概念

## 自然声音

## 音调(频率)

声音的频率，频率代表音阶的高低，频率越高，波长越短。

## 音量(振幅)

声音的响度，声音能量大小的反映。

## 音色(波形)

声音的波形

## 数字音频

### 声道(CHANNEL)

扬声器个数，单声道(Mono)、双声道(Stereo)... 对应代码中`AudioFormat.CHANNEL_IN_MONO`

#### 采样率(SAMPLE_RATE_IN_HZ)

每秒对应的采样频率，如48kHz，代表每秒采样48000次。根据奈奎斯特理论，采样频率只要不低于音频信号最高频率的两倍，就可以无损失地还原原始的声音。常见采样频率8kHz、11.025kHz、22.05kHz、16kHz、37.8kHz、44.1kHz、48kHz、96kHz、192kHz等。

### 量化精度(位宽 BIT_DEPTH)

采样点(采样率每一次)对应的数据大小，常见有8bit、16bit... 对应 `AudioFormat.ENCODING_PCM_16BIT`

### 音频帧(MIN_BUFFER_SIZE)

采样时间2.5ms - 120ms的音频数据大小，对应代码中`getMinBufferSize`方法获取

### 比特率(BIT_RATE)

多媒体行业在指音频或者视频在单位时间内的数据传输率时通常使用**码流**或**码率**，单位是kbps（千位每秒）。

原始PCM数据的比特率等于：

sample rate × bit depth × channel

在有损压缩中常见的比特率：

- 32 kbit/s—[MW](https://zh.wikipedia.org/wiki/%E4%B8%AD%E6%B3%A2 "中波")（[AM](https://zh.wikipedia.org/wiki/%E8%B0%83%E5%B9%85%E5%B9%BF%E6%92%AD "调幅广播")）质量。一般只用于语音。
- 96 kbit/s—FM质量。一般用于语音或低质量流媒体。
- 128 - 160 kbit/s –中档质量
- 192 kbit/s—中等质量比特率
- 256 kbit/s—常用的高质量比特率之一
- 320 kbit/s—MP3标准支持的最高比特率

## 音频编码

### 无损编码

wav: 对pcm数据加一个文件头，不对数据进行处理。

### 有损编码

mp3: 使用LAME编码，在中高码率情况下表现较好

aac: LC-AAC(适用于中高码率编码) HE-AAC(适用于中低码率编码) HE-AAC v2(适用于低码率编码)。

ogg:适用于语音聊天的音频消息场景

## 参考链接

[《音视频开发进阶指南:基于Android与iOS平台的实践》](https://book.douban.com/subject/30124646/)

[Android音频开发（1）：基础知识](https://blog.51cto.com/ticktick/1748506)

[比特率 Wiki](https://en.wikipedia.org/wiki/Bit_rate)
