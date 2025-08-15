---
title: Android AVDemo（9）：视频封装，代码开源并提供解析
description: 介绍 Android 视频封装流程和原理，并提供 Demo 源码和解析。
author: Keyframe
date: 2025-02-23 11:48:08 +0800
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

这里是 Android 第九篇：**Android 视频封装 Demo**。这个 Demo 里包含以下内容：

- 1）实现一个视频采集模块；
- 2）实现一个视频编码模块，支持 H.264/H.265；
- 3）实现一个视频封装模块；
- 4）串联视频采集、编码、封装模块，将采集到的视频数据输入给编码模块进行编码，再将编码后的数据输入给 MP4 封装模块封装和存储；
- 5）详尽的代码注释，帮你理解代码逻辑和原理。

在本文中，我们将详解一下 Demo 的具体实现和源码。读完本文内容相信就能帮你掌握相关知识。

>>不过，如果你的需求是：1）直接获得全部工程源码；2）想进一步咨询音视频技术问题；3）咨询音视频职业发展问题。可以根据自己的需要考虑是否加入『关键帧的音视频开发圈』。
>>
>>![长按识别二维码→加入我们](assets/img/keyframe-zsxq.png)
>>_长按识别二维码→加入我们_



## 1、视频采集模块

在这个 Demo 中，视频采集模块 `KFVideoCapture` 的实现与[《Android 视频采集 Demo》](https://mp.weixin.qq.com/s/rS077Uu4qJnDJMDD7vto5Q)中一样，这里就不再重复介绍了，其接口如下：


`KFIVideoCapture.java`


```java
public interface KFIVideoCapture {
    ///< 视频采集初始化。
    public void setup(Context context, KFVideoCaptureConfig config, KFVideoCaptureListener listener, EGLContext eglShareContext);
    ///< 释放采集实例。
    public void release();

    ///< 开始采集。
    public void startRunning();
    ///< 关闭采集。
    public void stopRunning();
    ///< 是否正在采集。
    public boolean isRunning();
    ///< 获取 OpenGL 上下文。
    public EGLContext getEGLContext();
    ///< 切换摄像头。
    public void switchCamera();
}
```


## 2、视频编码模块


同样的，视频编码模块 `KFByteBufferCodec`、`KFVideoSurfaceEncoder` 的实现与[《Android 视频编码 Demo》](https://mp.weixin.qq.com/s/URVrvHrDaj-RsJnJiNT6oA)中一样，这里就不再重复介绍了，其接口如下：

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

    ///< 初始化 Codec，第一个参数需告知使用编码还是解码。
    public void setup(boolean isEncoder,MediaFormat mediaFormat, KFMediaCodecListener listener, EGLContext eglShareContext);
    ///< 释放 Codec。
    public void release();

    ///< 获取输出格式描述。
    public MediaFormat getOutputMediaFormat();
    ///< 获取输入格式描述。
    public MediaFormat getInputMediaFormat();
    ///< 处理每一帧数据，编码前与编码后都可以，支持编解码 2 种模式。
    public int processFrame(KFFrame frame);
    ///< 清空 Codec 缓冲区。
    public void flush();
}
```

## 3、视频封装模块

视频编码模块即 `KFMP4Muxer`，复用了[《Android 音频封装 Demo》](https://mp.weixin.qq.com/s/8PLWVp3soM7A5jJIywDmqQ)中介绍的 muxer，这里就不再重复介绍了，其接口如下：

`KFMP4Muxer.java`

```java
public class KFMP4Muxer {
    public KFMP4Muxer(KFMuxerConfig config, KFMuxerListener listener); ///< 根据配置与回调初始化。
    public void start(); ///< 开始。
    public void stop(); ///< 停止。
    public void setVideoMediaFormat(MediaFormat mediaFormat); ///< 设置视频数据格式描述。
    public void setAudioMediaFormat(MediaFormat mediaFormat); ///< 设置音频数据格式描述。
    public void writeSampleData(boolean isVideo, ByteBuffer buffer, MediaCodec.BufferInfo bufferInfo); ///< 写入音视频数据(编码后数据)。
    public void release(); ///< 释放。
}
```

## 4、采集视频数据进行 H.264/H.265 编码以及 MP4 封装和存储

我们还是在一个 `MainActivity` 中来实现采集视频数据进行 H.264/H.265 编码以及 MP4 封装和存储的逻辑。

`MainActivity.java`

```java
public class MainActivity extends AppCompatActivity {
    private KFIVideoCapture mCapture; ///< 视频采集。
    private KFVideoCaptureConfig mCaptureConfig; ///< 视频采集配置。
    private KFRenderView mRenderView; ///< 渲染视图。
    private KFGLContext mGLContext; ///< OpenGL 上下文。

    private KFVideoEncoderConfig mEncoderConfig; ///< 编码配置。
    private KFMediaCodecInterface mEncoder; ///< 编码。
    private KFMP4Muxer mMuxer; ///< 封装器。
    private KFMuxerConfig mMuxerConfig; ///< 封装配置。

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

        ///< 创建 GL 上下文。
        mGLContext = new KFGLContext(null);
        ///< 创建渲染视图。
        mRenderView = new KFRenderView(this,mGLContext.getContext());

        WindowManager windowManager = (WindowManager)this.getSystemService(this.WINDOW_SERVICE);
        Rect outRect = new Rect();
        windowManager.getDefaultDisplay().getRectSize(outRect);
        FrameLayout.LayoutParams params = new FrameLayout.LayoutParams(outRect.width(), outRect.height());
        addContentView(mRenderView,params);

        FrameLayout.LayoutParams startParams = new FrameLayout.LayoutParams(200, 120);
        startParams.gravity = Gravity.CENTER_HORIZONTAL;
        Button startButton = new Button(this);
        startButton.setTextColor(Color.BLUE);
        startButton.setText("开始");
        startButton.setVisibility(View.VISIBLE);
        startButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                if (mEncoder == null) {
                    mEncoder = new KFVideoSurfaceEncoder();
                    MediaFormat mediaFormat = KFAVTools.createVideoFormat(mEncoderConfig.isHEVC,mEncoderConfig.size, MediaCodecInfo.CodecCapabilities.COLOR_FormatSurface,mEncoderConfig.bitrate,mEncoderConfig.fps,mEncoderConfig.gop / mEncoderConfig.fps,mEncoderConfig.profile,mEncoderConfig.profileLevel);
                    mEncoder.setup(true,mediaFormat,mVideoEncoderListener,mGLContext.getContext());
                    mMuxer = new KFMP4Muxer(mMuxerConfig,mMuxerListener);
                    mMuxer.start();
                    ((Button)view).setText("停止");
                } else {
                    mEncoder.release();
                    mEncoder = null;
                    mMuxer.stop();
                    mMuxer.release();
                    mMuxer = null;
                    ((Button)view).setText("开始");
                }
            }
        });
        addContentView(startButton, startParams);

        ///< 创建采集配置。
        mCaptureConfig = new KFVideoCaptureConfig();
        mCaptureConfig.cameraFacing = LENS_FACING_FRONT;
        mCaptureConfig.resolution = new Size(720,1280);
        mCaptureConfig.fps = 30;
        ///< 使用 Camera1 摄像头还是 Camera2 摄像头。
        boolean useCamera2 = false;
        if (useCamera2) {
            mCapture = new KFVideoCaptureV2();
        } else {
            mCapture = new KFVideoCaptureV1();
        }
        mCapture.setup(this,mCaptureConfig,mVideoCaptureListener,mGLContext.getContext());
        mCapture.startRunning();

        ///< 创建编码配置 & 封装配置。
        mEncoderConfig = new KFVideoEncoderConfig();
        mMuxerConfig = new KFMuxerConfig(Environment.getExternalStorageDirectory().getPath() + "/test.mp4");
    }

    private KFVideoCaptureListener mVideoCaptureListener = new KFVideoCaptureListener() {
        @Override
        public void cameraOnOpened(){}

        @Override
        public void cameraOnClosed() {
        }

        @Override
        public void cameraOnError(int error,String errorMsg) {

        }

        @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
        @Override
        public void onFrameAvailable(KFFrame frame) {
            ///< 采集数据回调进入渲染与编码。
            mRenderView.render((KFTextureFrame) frame);
            if (mEncoder != null) {
                mEncoder.processFrame(frame);
            }
        }
    };

    private KFMediaCodecListener mVideoEncoderListener = new KFMediaCodecListener() {
        @Override
        public void onError(int error, String errorMsg) {

        }

        @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
        @Override
        public void dataOnAvailable(KFFrame frame) {
            ///< 编码回调数据进入封装器。
            if (mMuxer != null) {
                if ((((KFBufferFrame)frame).bufferInfo.flags & BUFFER_FLAG_CODEC_CONFIG) != 0) {
                    mMuxer.setVideoMediaFormat(mEncoder.getOutputMediaFormat());
                } else {
                    mMuxer.writeSampleData(true,((KFBufferFrame)frame).buffer,((KFBufferFrame)frame).bufferInfo);
                }
            }
        }
    };

    private KFMuxerListener mMuxerListener = new KFMuxerListener() {
        @Override
        ///< 封装器出错回调。
        public void muxerOnError(int error, String errorMsg) {
            Log.e("KFMuxer","error:" + error + "msg:" +errorMsg);
        }
    };
}
```

上面是 `MainActivity` 的实现，其中主要包含这几个部分：


- 1）创建 OpenGL 上下文。
	- 创建上下文 `mGLContext`，这样好处是采集与预览可以共享，提高扩展性。 
- 2）创建采集实例。
	- 这里需要注意的是，我们通过开关 `useCamera2` 选择 `Camera` 或 `Camera2`。
	- 参数配置 `mCaptureConfig`，可自定义摄像头方向、帧率、分辨率。
- 3）采集数据回调中获取纹理数据输入给渲染模块与编码模块。
	- 在 `KFVideoCaptureListener` 的 `onFrameAvailable` 回调中实现。
- 4）在编码模块的数据回调中获取编码后的 H.264/H.265 数据，并将数据交给封装器 `KFMP4Muxer` 进行封装。
	- 在 `KFMediaCodecListener` 的 `dataOnAvailable` 回调中实现。




## 5、用工具播放 MP4 文件


完成 Demo 后，可以将 `sdcard` 文件夹下面的 `test.mp4` 文件拷贝到电脑上，使用 `ffplay` 播放来验证一下效果是否符合预期：

```c
$ ffplay -i test.mp4
```

关于播放 MP4 文件的工具，可以参考[《FFmpeg 工具》第 2 节 ffplay 命令行工具](https://mp.weixin.qq.com/s/Rl7fxOP-YH37mQEvGxhfUA)和[《可视化音视频分析工具》第 3.5 节 VLC 播放器](https://mp.weixin.qq.com/s/jCYih3qgEIUctuWxn0aTGQ)。


我们还可以用[《可视化音视频分析工具》第 3.1 节 MP4Box.js
](https://mp.weixin.qq.com/s/jCYih3qgEIUctuWxn0aTGQ) 等工具来查看它的格式：


![Demo 生成的 MP4 文件结构](assets/resource/av-demo/av-demo-ios-video-muxer-1.png)
_Demo 生成的 MP4 文件结构_






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

