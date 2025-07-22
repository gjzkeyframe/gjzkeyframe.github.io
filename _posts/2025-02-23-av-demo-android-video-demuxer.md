---
title: Android AVDemo（10）：视频解封装，代码开源并提供解析
description: 介绍 Android 视频解封装流程和原理，并提供 Demo 源码和解析。
author: Keyframe
date: 2025-02-23 11:28:08 +0800
categories: [音视频源码示例]
tags: [音视频源码示例, 音视频, Android, 视频, 视频封装]
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

这里是 Android 第十篇：**Android 视频解封装 Demo**。这个 Demo 里包含以下内容：

- 1）实现一个视频解封装模块；
- 2）实现对 MP4 文件中视频部分的解封装逻辑并将解封装后的编码数据存储为 H.264/H.265 文件；
- 3）详尽的代码注释，帮你理解代码逻辑和原理。

在本文中，我们将详解一下 Demo 的具体实现和源码。读完本文内容相信就能帮你掌握相关知识。

>>不过，如果你的需求是：1）直接获得全部工程源码；2）想进一步咨询音视频技术问题；3）咨询音视频职业发展问题。可以根据自己的需要考虑是否加入『关键帧的音视频开发圈』。
>>
>>![长按识别二维码→加入我们](assets/img/keyframe-zsxq.png)
>>_长按识别二维码→加入我们_


## 1、视频解封装模块

视频解封装模块即 `KFMP4Demuxer`，复用了[《Android 音频解封装 Demo》](https://mp.weixin.qq.com/s/2tv6J-11FMjq3YCQoJC8eQ)中介绍的 demuxer，这里就不再重复介绍了，其接口如下：

`KFMP4Demuxer.java`

```java
public class KFMP4Demuxer {
    public KFMP4Demuxer(KFDemuxerConfig config, KFDemuxerListener listener); ///< 构造方法：配置 & 回调。
    public void release(); ///< 释放解封装器实例。
    public boolean hasVideo(); ///< 是否包含视频。
    public boolean hasAudio(); ///< 是否包含音频。
    public int duration(); ///< 文件时长。
    public int rotation(); ///< 视频旋转角度。
    public boolean isHEVC(); ///< 是否为 H265。
    public int width(); ///< 视频宽度。
    public int height(); ///< 视频高度。
    public int samplerate(); ///< 音频采样率。
    public int channel(); ///< 音频声道数。
    public int audioProfile(); ///< 音频 profile。
    public int videoProfile(); ///< 视频 profile。
    public MediaFormat audioMediaFormat(); ///< 音频格式描述。
    public MediaFormat videoMediaFormat(); //< 视频格式描述。
    public ByteBuffer readAudioSampleData(MediaCodec.BufferInfo bufferInfo); ///< 读取音频帧。
    public ByteBuffer readVideoSampleData(MediaCodec.BufferInfo bufferInfo); ///< 读取视频帧。
}
```



## 2、解封装 MP4 文件中的视频部分存储为 H.264/H.265 文件

我们还是在一个 `MainActivity` 中来实现对一个 MP4 文件解封装、获取其中的视频编码数据并存储为 H.264/H.265 文件。

`MainActivity.java`

```java
public class MainActivity extends AppCompatActivity {
    private KFMP4Demuxer mDemuxer; ///< 解封装器。
    private KFDemuxerConfig mDemuxerConfig; ///< 解封装配置。
    private FileOutputStream mStream = null;

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ///< 申请采集、存储权限。
        if (ActivityCompat.checkSelfPermission(this, Manifest.permission.RECORD_AUDIO) != PackageManager.PERMISSION_GRANTED || ActivityCompat.checkSelfPermission(this, Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED ||
                ActivityCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED ||
                ActivityCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions((Activity) this,
                    new String[] {Manifest.permission.CAMERA,Manifest.permission.RECORD_AUDIO, Manifest.permission.READ_EXTERNAL_STORAGE, Manifest.permission.WRITE_EXTERNAL_STORAGE},
                    1);
        }

        ///< 解封装配置，控制仅输出视频。
        mDemuxerConfig = new KFDemuxerConfig();
        mDemuxerConfig.path = Environment.getExternalStorageDirectory().getPath() + "/2.mp4";
        mDemuxerConfig.demuxerType = KFMediaBase.KFMediaType.KFMediaVideo;
        if (mStream == null) {
            try {
                mStream = new FileOutputStream(Environment.getExternalStorageDirectory().getPath() + "/test.h264");
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
                ///< 解封装创建。
                if (mDemuxer == null) {
                    mDemuxer = new KFMP4Demuxer(mDemuxerConfig,mDemuxerListener);

                    ///< 根据 HEVC 分别获取 vps、sps、pps 等信息。
                    if (mDemuxer.isHEVC()) {
                        try {
                            ByteBuffer extradata = mDemuxer.videoMediaFormat().getByteBuffer("csd-0");
                            byte[] extradataBytes = new byte[extradata.capacity()];
                            extradata.get(extradataBytes);
                            mStream.write(extradataBytes);
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    } else {
                        try {
                            ByteBuffer sps = mDemuxer.videoMediaFormat().getByteBuffer("csd-0");
                            byte[] spsBytes = new byte[sps.capacity()];
                            sps.get(spsBytes);
                            mStream.write(spsBytes);
                        } catch (IOException e) {
                            e.printStackTrace();
                        }

                        try {
                            ByteBuffer pps = mDemuxer.videoMediaFormat().getByteBuffer("csd-1");
                            byte[] ppsBytes = new byte[pps.capacity()];
                            pps.get(ppsBytes);
                            mStream.write(ppsBytes);
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }

                    ///< 循环读取视频数据。
                    MediaCodec.BufferInfo bufferInfo = new MediaCodec.BufferInfo();
                    ByteBuffer nextBuffer = mDemuxer.readVideoSampleData(bufferInfo);
                    while (nextBuffer != null) {
                        try {
                            byte[] dst = new byte[bufferInfo.size];
                            nextBuffer.get(dst);
                            mStream.write(dst);
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                        nextBuffer = mDemuxer.readVideoSampleData(bufferInfo);
                    }
                    Log.i("KFDemuxer","complete");
                }
            }
        });
        addContentView(startButton, startParams);
    }

    private KFDemuxerListener mDemuxerListener = new KFDemuxerListener() {
        @Override
        ///< 解封装出错回调。
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
- 3）读取解封装后的视频编码数据并存储为 H.264/H.265 文件。
	- 循环读取 `readVideoSampleData` H.264/H.265 裸数据。
	- 需要注意的是，我们从解封装器读取的音频 H.264/H.265 编码数据在存储文件时需要添加 VPS、PPS、SPS 信息。



## 3、用工具播放 H.264/H.265 文件

完成视频解封装后，可以将 `sdcard` 文件夹下面的 `test.h264` 或 `test.h265` 文件拷贝到电脑上，使用 `ffplay` 播放来验证一下视频解封装的效果是否符合预期：

```c
$ ffplay -i test.h264
$ ffplay -i test.h265
```

关于播放 H.264/H.265 文件的工具，可以参考[《FFmpeg 工具》第 2 节 ffplay 命令行工具](https://mp.weixin.qq.com/s/Rl7fxOP-YH37mQEvGxhfUA)和[《可视化音视频分析工具》第 2.1 节 StreamEye](https://mp.weixin.qq.com/s/jCYih3qgEIUctuWxn0aTGQ)。












