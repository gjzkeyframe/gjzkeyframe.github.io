---
title: Android AVDemo（5）：音频解码，代码开源并提供解析
description: 介绍 Android 音频解码的流程和原理，并提供 Demo 源码和解析。
author: Keyframe
date: 2025-02-23 10:28:08 +0800
categories: [音视频源码示例]
tags: [音视频源码示例, 音视频, Android, 音频, 音频编解码]
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

这里是 Android 第五篇：**Android 音频解码 Demo**。这个 Demo 里包含以下内容：

- 1）实现一个音频解封装模块；
- 2）实现一个音频解码模块；
- 3）实现对 MP4 文件中音频部分的解封装和解码逻辑，并将解封装、解码后的数据存储为 PCM 文件；
- 4）详尽的代码注释，帮你理解代码逻辑和原理。


>>如果你想获得全部源码和参与音视频技术讨论，可以通过下面二维码加入『关键帧的音视频开发圈』，当然也可以跳过直接看后续的内容。
>>
>>![长按识别二维码→加入我们](assets/img/keyframe-zsxq.png)
>>_长按识别二维码→加入我们_


## 1、音频解封装模块



在这个 Demo 中，解封装模块 `KFMP4Demuxer` 的实现与 [《Android 音频解封装 Demo》](https://mp.weixin.qq.com/s?__biz=MjM5MTkxOTQyMQ==&mid=2257485659&idx=1&sn=1acc6ec2b1240f4b2e389668a14e34f7&chksm=a5d4e30992a36a1f51f4043a3badb5693d8df54c0a9694ed22663f9089795c58b9bb5b72625b&token=991245230&lang=zh_CN#rd) 中一样，这里就不再重复介绍了，其接口如下：

`KFMP4Demuxer.java`

```java
public class KFMP4Demuxer {
    public KFMP4Demuxer(KFDemuxerConfig config, KFDemuxerListener listener); ///< 构造方法 配置 & 回调。
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
    public int audioProfile(); ///< 音频profile。
    public int videoProfile(); ///< 视频profile。
    public MediaFormat audioMediaFormat(); ///< 音频格式描述。
    public MediaFormat videoMediaFormat(); ///< 视频格式描述。
    public ByteBuffer readAudioSampleData(MediaCodec.BufferInfo bufferInfo); ///< 读取音频帧。
    public ByteBuffer readVideoSampleData(MediaCodec.BufferInfo bufferInfo); ///< 读取视频帧。
}
```


## 2、音频解码模块

接下来，我们来实现一个音频解码模块 `KFByteBufferCodec`，在这里输入解封装后的编码数据，输出解码后的数据。解码模块 `KFByteBufferCodec` 的实现与 [《Android 音频编码 Demo》](https://mp.weixin.qq.com/s?__biz=MjM5MTkxOTQyMQ==&mid=2257485614&idx=1&sn=636683b05eacc4f4728fb2849a445ded&chksm=a5d4e37c92a36a6a8a4d3d1991cebc5fde3775086b4da22d78924d7f965e78fccbf6605c0be5&token=991245230&lang=zh_CN#rd) 中一样，这里就不再重复介绍了，其接口如下

`KFMediaCodecInterface.java`


```java
public interface KFMediaCodecInterface {
    public static final int KFMediaCodecInterfaceErrorCreate = -2000;
    public static final int KFMediaCodecInterfaceErrorConfigure = -2001;
    public static final int KFMediaCodecInterfaceErrorStart = -2002;
    public static final int KFMediaCodecInterfaceErrorDequeueOutputBuffer = -2003;
    public static final int KFMediaCodecInterfaceErrorParams = -2004;

    public static int KFMediaCodeProcessParams = -1;
    public static int KFMediaCodeProcessAgainLater = -2;
    public static int KFMediaCodeProcessSuccess = 0;

    ///< 初始化Codec,第一个参数需告知使用编码还是解码。
    public void setup(boolean isEncoder,MediaFormat mediaFormat, KFMediaCodecListener listener, EGLContext eglShareContext);
    ///< 释放Codec。
    public void release();

    ///< 获取输出格式描述。
    public MediaFormat getOutputMediaFormat();
    ///< 获取输入格式描述。
    public MediaFormat getInputMediaFormat();
    ///< 处理每一帧数据，编码前与编码后都可以，支持编解码2种模式。
    public int processFrame(KFFrame frame);
    ///< 清空 Codec 缓冲区。
    public void flush();
}
```

上面是 `KFByteBufferCodec` 接口的设计，与音频编码对比区别如下：

- 1）音频编码使用了继承类 `KFAudioByteBufferEncoder`，解码则直接使用类 `KFByteBufferCodec`。
	- 音频编码使用了继承类 `KFByteBufferCodec`，目的是切割合适大小的数据 `2048` 送入编码器,因为 AAC 数据编码每帧大小为 `1024 * 2（位深 16 Bit）`。
	- 音频解码使用了类 `KFByteBufferCodec`，音频解决封装后的数据通常都是一帧数据 `2048` 以及它的倍数。
- 2）外层使用构造方法时配置参数修改：
	- `setup` 接口 `mIsEncoder` 设置为 `false` 代表解码，`mInputMediaFormat` 需要设置解码的格式描述。

更具体细节见上述代码及其注释。




## 3、解封装和解码 MP4 文件中的音频部分存储为 PCM 文件


我们在一个 `MainActivity` 中来实现音频解封装及解码逻辑，并将解码后的数据存储为 PCM 文件。

`MainActivity.java`

```java
public class MainActivity extends AppCompatActivity {
    private KFDemuxer mDemuxer; ///< 音频解封装
    private KFDemuxerConfig mDemuxerConfig; ///< 音频解封装配置
    private KFMediaCodecInterface mDecoder; ///< 音频解码
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
        mDemuxerConfig.demuxerType = KFGLBase.KFMediaType.KFMediaAudio;
        if (mStream == null) {
            try {
                mStream = new FileOutputStream(Environment.getExternalStorageDirectory().getPath() + "/test.pcm");
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
                ///< 创建解封装器 与 解码器。
                if (mDemuxer == null) {
                    mDemuxer = new KFDemuxer(mDemuxerConfig,mDemuxerListener);
                    mDecoder = new KFByteBufferCodec();
                    mDecoder.setup(false,mDemuxer.audioMediaFormat(),mDecoderListener,null);

                    MediaCodec.BufferInfo bufferInfo = new MediaCodec.BufferInfo();
                    ByteBuffer nextBuffer = mDemuxer.readAudioSampleData(bufferInfo);
                    ///< 循环读取音频帧进入解码器。
                    while (nextBuffer != null) {
                        mDecoder.processFrame(new KFBufferFrame(nextBuffer,bufferInfo));
                        nextBuffer = mDemuxer.readAudioSampleData(bufferInfo);
                    }
                    mDecoder.flush();
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

    private KFMediaCodecListener mDecoderListener = new KFMediaCodecListener() {
        @Override
        ///< 解码出错回调。
        public void onError(int error, String errorMsg) {

        }

        @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
        @Override
        ///< 解码数据回调 存储本地。
        public void dataOnAvailable(KFFrame frame) {
            KFBufferFrame bufferFrame = (KFBufferFrame)frame;
            try {
                byte[] dst = new byte[bufferFrame.bufferInfo.size];
                bufferFrame.buffer.get(dst);
                mStream.write(dst);
            }  catch (IOException e) {
                e.printStackTrace();
            }
        }
    };
}
```

上面是 `MainActivity` 的实现，其中主要包含这几个部分：

- 1）通过启动音频解封装来驱动整个解封装和解码流程。
	- 在 `onClick` 中实现开始动作，并且循环读取数据输入给解码器。
	- 解码器实例初始化第一个参数为 `false`，代表解码。
	- 解码器输入音频格式描述从解封装器获取 `audioMediaFormat`。
- 2）在解码模块 `KFByteBufferCodec` 的数据回调中获取解码后的 PCM 数据存储为文件。
	- 在 `KFMediaCodecListener` 的 `dataOnAvailable` 回调中实现。



## 4、用工具播放 PCM 文件


完成音频解码后，可以将 `sdcard` 文件夹下面的 `test.pcm` 文件拷贝到电脑上，使用 `ffplay` 播放来验证一下音频采集是效果是否符合预期：

```c
$ ffplay -ar 44100 -channels 2 -f s16le -i test.pcm
```

注意这里的参数要对齐在工程中输入视频源的`采样率`、`声道数`、`采样位深`。比如我们的 Demo 中输入视频源的声道数是 2，所以上面的声道数需要设置为 2 才能播放正常的声音。


关于播放 PCM 文件的工具，可以参考[《FFmpeg 工具》第 2 节 ffplay 命令行工具](https://mp.weixin.qq.com/s/Rl7fxOP-YH37mQEvGxhfUA)和[《可视化音视频分析工具》第 1.1 节 Adobe Audition](https://mp.weixin.qq.com/s/jCYih3qgEIUctuWxn0aTGQ)。























---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
>
>你还可以加入我们的微信群和更多同行朋友来交流和讨论：
>
>![关键帧的音视频开发群](assets/img/av-wechat-group.jpg){: w="600" }
>_微信扫码进群_
{: .prompt-tip }

