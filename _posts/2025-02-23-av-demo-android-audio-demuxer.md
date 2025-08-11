---
title: Android AVDemo（4）：音频解封装，代码开源并提供解析
description: 介绍 Android 音频解封装的流程和原理，并提供 Demo 源码和解析。
author: Keyframe
date: 2025-02-23 10:38:08 +0800
categories: [音视频源码示例]
tags: [音视频源码示例, 音视频, Android, 音频, 音频封装]
pin: false
math: true
mermaid: true
---

>想要学习和提升音视频技术的朋友，快来加入我们的<a href="https://t.zsxq.com/jRprT" target="_blank" rel="noopener noreferrer">【音视频技术社群】</a>，加入后你就能：
>
>- 1）下载 30+ 个开箱即用的「音视频及渲染 Demo 源代码」
>- 2）下载包含 500+ 知识条目的完整版「音视频知识图谱」
>- 3）下载包含 200+ 题目的完整版「音视频面试题集锦」
>- 4）技术和职业发展咨询 100% 得到回答
>- 5）获得简历优化建议和大厂内推
>  
>现在加入，送你一张 20 元优惠券：<a href="https://t.zsxq.com/jRprT" target="_blank" rel="noopener noreferrer">点击领取优惠券</a>
>
>![知识星球新人优惠券](assets/img/keyframe-zsxq-coupon.png){: w="300" }
>_微信扫码也可领取优惠券_
{: .prompt-tip }

iOS/Android 客户端开发同学如果想要开始学习音视频开发，最丝滑的方式是对[音视频基础概念知识](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2140155659944787969#wechat_redirect)有一定了解后，再借助 iOS/Android 平台的音视频能力上手去实践音视频的`采集 → 编码 → 封装 → 解封装 → 解码 → 渲染`过程，并借助[音视频实用工具](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2216997905264082945#wechat_redirect)来分析和理解对应的音视频数据。

在[音视频工程示例](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2273301900659851268#wechat_redirect)这个栏目，我们将通过拆解`采集 → 编码 → 封装 → 解封装 → 解码 → 渲染`流程并实现 Demo 来向大家介绍如何在 iOS/Android 平台上手音视频开发。

这里是 Android 第四篇：**Android 音频解封装 Demo**。这个 Demo 里包含以下内容：


- 1）实现一个音频解封装模块；
- 2）实现对 MP4 文件中音频部分的解封装逻辑并将解封装后的编码数据存储为 AAC 文件；
- 3）详尽的代码注释，帮你理解代码逻辑和原理。


>>如果你想获得全部源码和参与音视频技术讨论，可以通过下面二维码加入『关键帧的音视频开发圈』，当然也可以跳过直接看后续的内容。
>>
>>![长按识别二维码→加入我们](assets/img/keyframe-zsxq.png)
>>_长按识别二维码→加入我们_


## 1、音频解封装模块


首先，实现一个 `KFDemuxerConfig` 类用于定义音频解封装参数的配置。这里包括了：视频路径、解封装类型这几个参数。这样设计是因为这个配置类不仅会用于音频解封装，后续的视频解封装也会使用。


`KFDemuxerConfig.java`

```java
public class KFDemuxerConfig {
    ///< 输入路径。
    public String path;
    ///< 音视频解封装类型（仅音频、仅视频、音视频）。
    public KFMediaBase.KFMediaType demuxerType = KFMediaBase.KFMediaType.KFMediaAV;
}
```

其中用到的 `KFMediaType` 是定义在 `KFMediaBase` 中的一个枚举：

`KFMediaBase.java`

```java
public class KFMediaBase {
    public enum KFMediaType{
        KFMediaUnkown(0),
        KFMediaAudio (1 << 0),
        KFMediaVideo  (1 << 1),
        KFMediaAV ((1 << 0) | (1 << 1));
        private int index;
        KFMediaType(int index) {
            this.index = index;
        }

        public int value() {
            return index;
        }
    }
}
```


接下来，我们实现一个 `KFMP4Demuxer` 类来实现 MP4 的解封装。它能从符合 MP4 标准的文件中解封装出音频编码数据。


`KFMP4Demuxer.java`


```java
public class KFMP4Demuxer {
    public static final int KFDemuxerErrorAudioSetDataSource = -2300;
    public static final int KFDemuxerErrorVideoSetDataSource = -2301;
    public static final int KFDemuxerErrorAudioReadData = -2302;
    public static final int KFDemuxerErrorVideoReadData = -2303;

    private static final String TAG = "KFDemuxer";
    private KFDemuxerConfig mConfig = null; ///< 解封装配置
    private KFDemuxerListener mListener = null; ///< 回调
    private MediaExtractor mAudioMediaExtractor = null; ///< 音频解封装器
    private MediaFormat mAudioMediaFormat = null; ///< 音频格式描述
    private MediaExtractor mVideoMediaExtractor = null; ///< 视频解封装器
    private MediaFormat mVideoMediaFormat = null; ///< 视频格式描述
    private MediaMetadataRetriever mRetriever = null; ///< 视频信息获取实例
    private Handler mMainHandler = new Handler(Looper.getMainLooper()); ///< 主线程

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    public KFMP4Demuxer(KFDemuxerConfig config, KFDemuxerListener listener) {
        mConfig = config;
        mListener = listener;
        if (mRetriever == null) {
            mRetriever = new MediaMetadataRetriever();
            mRetriever.setDataSource(mConfig.path);
        }

        ///< 初始化音频解封装器。
        if (hasAudio() && (config.demuxerType.value() & KFMediaBase.KFMediaType.KFMediaAudio.value()) != 0) {
            _setupAudioMediaExtractor();
        }

        ///< 初始化视频解封装器。
        if (hasVideo() && (config.demuxerType.value() & KFMediaBase.KFMediaType.KFMediaVideo.value()) != 0) {
            _setupVideoMediaExtractor();
        }
    }

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    public void release() {
        ///< 释放音视频解封装器、视频信息获取实例。
        if (mAudioMediaExtractor != null) {
            mAudioMediaExtractor.release();
            mAudioMediaExtractor = null;
        }

        if (mVideoMediaExtractor != null) {
            mVideoMediaExtractor.release();
            mVideoMediaExtractor = null;
        }

        if (mRetriever != null) {
            mRetriever.release();
            mRetriever = null;
        }
    }

    public boolean hasVideo() {
        ///< 是否包含视频。
        if (mRetriever == null) {
            return false;
        }
        String value = mRetriever.extractMetadata(MediaMetadataRetriever.METADATA_KEY_HAS_VIDEO);
        return value != null && value.equals("yes");
    }

    public boolean hasAudio() {
        ///< 是否包含音频。
        if (mRetriever == null) {
            return false;
        }
        String value = mRetriever.extractMetadata(MediaMetadataRetriever.METADATA_KEY_HAS_AUDIO);
        return value != null && value.equals("yes");
    }

    public int duration() {
        ///< 文件时长。
        if (mRetriever == null) {
            return 0;
        }
        return Integer.parseInt(mRetriever.extractMetadata(MediaMetadataRetriever.METADATA_KEY_DURATION));
    }

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    public int rotation() {
        ///< 视频旋转。
        if (mVideoMediaFormat == null) {
            return 0;
        }
        return mVideoMediaFormat.getInteger(MediaFormat.KEY_ROTATION);
    }

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    public boolean isHEVC() {
        ///< 是否为 H.265。
        if (mVideoMediaFormat == null) {
            return false;
        }
        String mime = mVideoMediaFormat.getString(MediaFormat.KEY_MIME);
        return mime.contains("hevc") || mime.contains("dolby-vision");
    }

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    public int width() {
        ///< 视频宽度。
        if (mVideoMediaFormat == null) {
            return 0;
        }
        return mVideoMediaFormat.getInteger(MediaFormat.KEY_WIDTH);
    }

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    public int height() {
        ///< 视频高度。
        if (mVideoMediaFormat == null) {
            return 0;
        }
        return mVideoMediaFormat.getInteger(MediaFormat.KEY_HEIGHT);
    }

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    public int samplerate() {
        ///< 音频采样率。
        if (mAudioMediaFormat == null) {
            return 0;
        }
        return mAudioMediaFormat.getInteger(MediaFormat.KEY_SAMPLE_RATE);
    }

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    public int channel() {
        ///< 音频声道数。
        if (mAudioMediaFormat == null) {
            return 0;
        }
        return mAudioMediaFormat.getInteger(MediaFormat.KEY_CHANNEL_COUNT);
    }

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    public int audioProfile() {
        ///< AAC、HEAAC 等。
        if (mAudioMediaFormat == null) {
            return 0;
        }
        return mAudioMediaFormat.getInteger(MediaFormat.KEY_PROFILE);
    }

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    public int videoProfile() {
        ///< 视频画质级别 BaseLine Main High 等。
        if (mVideoMediaFormat == null) {
            return 0;
        }
        return mVideoMediaFormat.getInteger(MediaFormat.KEY_PROFILE);
    }

    public MediaFormat audioMediaFormat() {
        return mAudioMediaFormat;
    }

    public MediaFormat videoMediaFormat() {
        return mVideoMediaFormat;
    }

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    public ByteBuffer readAudioSampleData(MediaCodec.BufferInfo bufferInfo) {
        ///< 音频数据读取。
        if (mAudioMediaExtractor == null) {
            return null;
        }

        ByteBuffer buffer = ByteBuffer.allocateDirect(500 * 1024);
        try {
            bufferInfo.size = mAudioMediaExtractor.readSampleData(buffer, 0);
        } catch (Exception e) {
            Log.e(TAG, "readSampleData" + e);
            return null;
        }

        if (bufferInfo.size > 0) {
            bufferInfo.flags = mAudioMediaExtractor.getSampleFlags() == MediaExtractor.SAMPLE_FLAG_SYNC ? MediaCodec.BUFFER_FLAG_KEY_FRAME : 0;
            bufferInfo.presentationTimeUs = mAudioMediaExtractor.getSampleTime();
            mAudioMediaExtractor.advance();
            return buffer;
        } else {
            bufferInfo.flags = MediaCodec.BUFFER_FLAG_END_OF_STREAM;
            return null;
        }
    }

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    public ByteBuffer readVideoSampleData(MediaCodec.BufferInfo bufferInfo) {
        ///< 视频数据读取
        if (mVideoMediaExtractor == null) {
            return null;
        }

        ByteBuffer buffer = ByteBuffer.allocateDirect(1000 * 1024);
        try {
            bufferInfo.size = mVideoMediaExtractor.readSampleData(buffer, 0);
        } catch (Exception e) {
            Log.e(TAG, "readVideoData" + e);
            return null;
        }

        if (bufferInfo.size > 0) {
            bufferInfo.flags = mVideoMediaExtractor.getSampleFlags() == MediaExtractor.SAMPLE_FLAG_SYNC ? MediaCodec.BUFFER_FLAG_KEY_FRAME : 0;
            bufferInfo.presentationTimeUs = mVideoMediaExtractor.getSampleTime();
            mVideoMediaExtractor.advance();
            return buffer;
        } else {
            bufferInfo.flags = MediaCodec.BUFFER_FLAG_END_OF_STREAM;
            return null;
        }
    }

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    private void _setupAudioMediaExtractor() {
        ///< 初始化音频解封装器。
        if (mAudioMediaExtractor == null) {
            mAudioMediaExtractor = new MediaExtractor();
            try {
                mAudioMediaExtractor.setDataSource(mConfig.path);
            } catch (Exception e) {
                Log.e(TAG, "setDataSource" + e);
                _callBackError(KFDemuxerErrorAudioSetDataSource,e.getMessage());
                return;
            }

            ///< 查找音频轨道与格式描述。
            int numberTracks = mAudioMediaExtractor.getTrackCount();
            for(int index = 0; index < numberTracks; index ++) {
                MediaFormat format = mAudioMediaExtractor.getTrackFormat(index);
                String mime = format.getString(MediaFormat.KEY_MIME);
                if (mime.startsWith("audio/")) {
                    mAudioMediaFormat = format;
                    mAudioMediaExtractor.selectTrack(index);
                    mAudioMediaExtractor.seekTo(0,MediaExtractor.SEEK_TO_PREVIOUS_SYNC);
                }
            }
        }
    }

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    private void _setupVideoMediaExtractor() {
        ///< 初始化视频解封装器。
        if (mVideoMediaExtractor == null) {
            mVideoMediaExtractor = new MediaExtractor();
            try {
                mVideoMediaExtractor.setDataSource(mConfig.path);
            } catch (Exception e) {
                Log.e(TAG, "setDataSource" + e);
                _callBackError(KFDemuxerErrorVideoSetDataSource,e.getMessage());
                return;
            }

            ///< 查找视频轨道与格式描述。
            int numberTracks = mVideoMediaExtractor.getTrackCount();
            for(int index = 0; index < numberTracks; index++) {
                MediaFormat format = mVideoMediaExtractor.getTrackFormat(index);
                String mime = format.getString(MediaFormat.KEY_MIME);
                if (mime.startsWith("video/")) {
                    mVideoMediaFormat = format;
                    mVideoMediaExtractor.selectTrack(index);
                    mVideoMediaExtractor.seekTo(0,MediaExtractor.SEEK_TO_PREVIOUS_SYNC);
                }
            }
        }
    }

    private void _callBackError(int error, String errorMsg) {
        if (mListener != null) {
            mMainHandler.post(()->{
                mListener.demuxerOnError(error,TAG + errorMsg);
            });
        }
    }
}
```


上面是 `KFMP4Demuxer` 的实现，从代码上可以看到主要有这几个部分：

- 1）构造方法创建解封装器实例及获取视频信息实例。
	- 在 `_setupAudioMediaExtractor` 方法中初始化音频解封装器实例以及设置数据源 `setDataSource`，查找音频轨道下标与格式描述。
	- 在 `_setupVideoMediaExtractor` 方法中初始化视频解封装器实例以及设置数据源 `setDataSource`，查找视频轨道下标与格式描述。
	- 初始化获取视频信息实例，`mRetriever` 初始化视频获取信息实例以及设置数据源 `setDataSource`。
- 2）从音视频输入源读取数据。
	- 音频读取方法 `readAudioSampleData`，读取完一帧移动下一帧 `advance`。
	- 视频读取方法 `readVideoSampleData`，读取完一帧移动下一帧 `advance`。
- 3）清理解封装实例、获取视频信息实例，`release`。


更具体细节见上述代码及其注释。



## 2、解封装 MP4 文件中的音频部分存储为 AAC 文件

我们还是在一个 `MainActivity` 中来实现对一个 MP4 文件解封装、获取其中的音频编码数据并存储为 AAC 文件。

`MainActivity.java`

```java
public class MainActivity extends AppCompatActivity {
    private KFMP4Demuxer mDemuxer; ///< 解封装实例
    private KFDemuxerConfig mDemuxerConfig; ///< 解封装配置
    private FileOutputStream mStream = null;

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        if (ActivityCompat.checkSelfPermission(this, Manifest.permission.RECORD_AUDIO) != PackageManager.PERMISSION_GRANTED || ActivityCompat.checkSelfPermission(this, Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED ||
                ActivityCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED ||
                ActivityCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions((Activity) this,
                    new String[] {Manifest.permission.CAMERA,Manifest.permission.RECORD_AUDIO, Manifest.permission.READ_EXTERNAL_STORAGE, Manifest.permission.WRITE_EXTERNAL_STORAGE},
                    1);
        }

        mDemuxerConfig = new KFDemuxerConfig();
        mDemuxerConfig.path = Environment.getExternalStorageDirectory().getPath() + "/2.mp4";
        mDemuxerConfig.demuxerType = KFMediaBase.KFMediaType.KFMediaAudio;
        if (mStream == null) {
            try {
                mStream = new FileOutputStream(Environment.getExternalStorageDirectory().getPath() + "/test.aac");
            } catch (FileNotFoundException e) {
                e.printStackTrace();
            }
        }

        FrameLayout.LayoutParams startParams = new FrameLayout.LayoutParams(200, 120);
        startParams.gravity = Gravity.CENTER_HORIZONTAL;
        Button startButton = new Button(this);
        startButton.setTextColor(Color.BLUE);
        startButton.setText("开始");
        startButton.setVisibility(View.VISIBLE);
        startButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                ///< 创建解封装实例。
                if (mDemuxer == null) {
                    mDemuxer = new KFMP4Demuxer(mDemuxerConfig,mDemuxerListener);

                    ///< 读取音频数据。
                    MediaCodec.BufferInfo bufferInfo = new MediaCodec.BufferInfo();
                    ByteBuffer nextBuffer = mDemuxer.readAudioSampleData(bufferInfo);
                    while (nextBuffer != null) {
                        try {
                            ///< 添加 ADTS。
                            ByteBuffer adtsBuffer = KFAVTools.getADTS(bufferInfo.size,mDemuxer.audioProfile(),mDemuxer.samplerate(),mDemuxer.channel());
                            byte[] adtsBytes = new byte[adtsBuffer.capacity()];
                            adtsBuffer.get(adtsBytes);
                            mStream.write(adtsBytes);

                            byte[] dst = new byte[bufferInfo.size];
                            nextBuffer.get(dst);
                            mStream.write(dst);
                        }  catch (IOException e) {
                            e.printStackTrace();
                        }
                        nextBuffer = mDemuxer.readAudioSampleData(bufferInfo);
                    }
                    Log.i("KFDemuxer","complete");
                }
            }
        });
        addContentView(startButton, startParams);
    }

    private KFDemuxerListener mDemuxerListener = new KFDemuxerListener() {
        ///< 解封装错误回调。
        @Override
        public void demuxerOnError(int error, String errorMsg) {
            Log.i("KFDemuxer","error" + error + "msg" + errorMsg);
        }
    };
}
```

上面是 `MainActivity` 的实现，其中主要包含这几个部分：

- 1）设置好待解封装的资源。
	- 在 `mDemuxerConfig` 中实现，我们这里是一个 MP4 文件。
- 2）创建解封装器。
	- `new KFMP4Demuxer(mDemuxerConfig,mDemuxerListener)`。
- 3）读取解封装后的音频编码数据并存储为 AAC 文件。
	- 循环读取 `readAudioSampleData` AAC 裸数据。
	- 需要注意的是，我们从解封装器读取的音频 AAC 编码数据在存储为 AAC 文件时需要添加 ADTS 头。生成一个 AAC packet 对应的 ADTS 头数据在 `KFAVTools` 类的工具方法 `static ByteBuffer getADTS(int size, int profile, int sampleRate, int channel)` 中实现。这个在前面的音频编码的 Demo 中已经介绍过了。



## 3、用工具播放 AAC 文件

完成音频采集和编码后，可以将 `sdcard` 文件夹下面的 `test.aac` 文件拷贝到电脑上，使用 `ffplay` 播放来验证一下音频采集是效果是否符合预期：

```c
$ ffplay -i test.aac
```


关于播放 AAC 文件的工具，可以参考[《FFmpeg 工具》第 2 节 ffplay 命令行工具](https://mp.weixin.qq.com/s/Rl7fxOP-YH37mQEvGxhfUA)和[《可视化音视频分析工具》第 1.1 节 Adobe Audition](https://mp.weixin.qq.com/s/jCYih3qgEIUctuWxn0aTGQ)。










---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
>
>你还可以加入我们的微信群：
>
>![关键帧的音视频开发群](assets/img/av-wechat-group.jpg){: w="600" }
>_微信扫码进群_
{: .prompt-tip }

