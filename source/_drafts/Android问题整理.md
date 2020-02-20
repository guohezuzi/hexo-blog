# Android问题整理

1.[如何让一个按钮垂直居中](https://stackoverflow.com/questions/20185340/how-to-align-linearlayout-to-vertical-center)

layout_center...属性

2.视频播放相关API

VideoView类

3.SparseArray vs HashMap

4.drawable xml文件的意义

常用于背景和图片

5.AudioFocus机制

使用: audioManager.requestAudioFocus(new AudioFocusReques.Builder(**AudioManager.AudioManager.AUDIOFOCUS_GAIN**).build())

- AUDIOFOCUS_NONE 应用程序不会获取音频焦点，不会终止外部程序的音频播放

- AUDIOFOCUS_GAIN  应用程序获取音频焦点(时间未知)，终止外部程序的音频播放

- AUDIOFOCUS_GAIN_TRANSIENT 应用程序获取音频焦点(临时) 

- AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK 应用程序获取音频焦点(临时)，外部程序的音频音量降低

- AUDIOFOCUS_GAIN_TRANSIENT_EXCLUSIVE 应用程序获取音频焦点(临时)，屏蔽所有外部程序的音频音量

6.VideoPager(页面允许左右滑动的布局管理器) 轮播使用

- OnPageChangeListener

- setAdapter 对应页面适配器，FragmentPagerAdapter、FragmentStatePagerAdapter或自己实现

- setPageTransformer 设置页面滑动动画

- ViewPager.setCurrentItem(参数一,false)不展现动画，实现循环的效果

7.手势滑动相关

- cOnTouchListener

- MotionEvent 手指该接触屏幕后所产生的一系列事件,ACTION_DOWN(刚接触)、ACTION_MOVE、ACTION_UP(松开一瞬间)

- VelocityTracker 速度追踪

- GestureDetector 手势检测

- Scroller 弹性滑动对象

8.应用程序架构

- mvc (model-view-presenter) @Deprecated

- mvvm model-view-viewmodel viewmodel类似vue中的date()、props

9.adb 常用命令

1.logcat

2.dumpsys 

3.settings(list get put) 系统变量编辑

10.& 0xff作用

[保证二进制数据的一致性](https://www.cnblogs.com/think-in-java/p/5527389.html)

11.audio native代码位置

- framework/base/core/jni

- framework/base/media/jni

- framework/base/services/core/jni

- framework/av/services

12.屏幕常亮

`getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);`

`getWindow().clearFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);`

13.通过代码new View

14.ViewPager ViewView 源码

15.dex->odex->oat        AAPT 资源打包工具

16.Android.mk --extra-packages作用

//--extra-packages是为资源文件设置别名：意思是通过该应用包名+R，com.android.test1.R和com.android.test2.R都可以访问到资源
`additionalParameters '--extra-packages', 'com.android.test1','--extra-packages','com.android.test2'`

Android.mk

`LOCAL_AAPT_FLAGS += --extra-packages org.chromium.content`
`LOCAL_AAPT_FLAGS += --extra-packages org.chromium.ui`

R资源生成别名：如果是com.android.test.R，会同时生成org.chromium.content.R和org.chromium.ui.R

17.锁屏Activity逻辑

- 短按

- 长按

18.AndroidManger flag含义

19.TODO

- mediaplayer源码解读

- viewpager源码解读

- viewpager动画

20.一些音频相关的术语

Application Processor(即AP)（应用处理器）AP一般采用[ARM芯片](https://www.baidu.com/s?wd=ARM%E8%8A%AF%E7%89%87&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1d9mWwhmy7Wm1--ujP9mH0d0ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6KdThsqpZwYTjCEQLGCpyw9Uz4Bmy-bIi4WUvYETgN-TLwGUv3EPjDknjT4n1mz)的CPU。运行在Application Processor(AP)的软件包称为AP包,包括[操作系统](https://www.baidu.com/s?wd=%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F&tn=44039180_cpr&fenlei=mv6quAkxTZn0IZRqIHckPjm4nH00T1d9mWwhmy7Wm1--ujP9mH0d0ZwV5Hcvrjm3rH6sPfKWUMw85HfYnjn4nH6sgvPsT6KdThsqpZwYTjCEQLGCpyw9Uz4Bmy-bIi4WUvYETgN-TLwGUv3EPjDknjT4n1mz)、用户界面和应用程序等;

Baseband Processor(即BP)（基带处理器）

ADC(analog-to-digital converter)

DAC(digital-to-analog converter)

SmartPA(smart personal assistant)

21.ffmpeg命令格式

```bash
$ ffmpeg \
[全局参数] \
[输入文件参数] \
-i [输入文件] \
[输出文件参数] \
[输出文件]
```

22.BpBinder(客户端)和BBinder(服务端)

23.Android按照进程的重要性，划分5级：

1. 前台进程 (Foreground process)

2. 可见进程 (Visible process)

3. 服务进程 (Service process)

4. 后台进程 (Background process)

5. 空进程 (Empty process)

24.Service

bindService()、startService()启动，如果需要bind通信使用bindService()，在其中传入的ServiceConnection中获取bind对象通信

IntentService 异步service，简单的说一个IntentService,内部就创建了一个线程，通过Android提供的 Handler Message Looper,这些消息处理的类 构成了一个消息处理的模型。所以IntentService 的onHandleIntent 这个方法其实是在IntentService 中开辟的一个子线程中处理的。

AIDL通信 跨进程通信

25.sva代码位置

3.0 vendor/qcom/proprietary/mm-audio/audio-listen

1.0 vendor/qcom/proprietary/commonsys/qrdplus/sva/audio-listen/sva
