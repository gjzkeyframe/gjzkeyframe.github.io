---
title: Android AVDemo（8）：视频编码，代码开源并提供解析
description: 介绍 Android 视频编码流程和原理，并提供 Demo 源码和解析。
author: Keyframe
date: 2025-02-23 11:38:08 +0800
categories: [音视频源码示例]
tags: [音视频源码示例, 音视频, Android, 视频, 编解码]
pin: false
math: true
mermaid: true
---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg)
>_微信扫码关注我们_
>
>您还可以加入知识星球 `关键帧的音视频开发圈` 来一起交流工作中的**技术难题、职场经验**：
>
>![知识星球](assets/img/keyframe-zsxq.png)
>_微信扫码加入星球_
{: .prompt-tip }


iOS/Android 客户端开发同学如果想要开始学习音视频开发，最丝滑的方式是对[音视频基础概念知识](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2140155659944787969#wechat_redirect)有一定了解后，再借助 iOS/Android 平台的音视频能力上手去实践音视频的`采集 → 编码 → 封装 → 解封装 → 解码 → 渲染`过程，并借助[音视频实用工具](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2216997905264082945#wechat_redirect)来分析和理解对应的音视频数据。

在[音视频工程示例](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2273301900659851268#wechat_redirect)这个栏目，我们将通过拆解`采集 → 编码 → 封装 → 解封装 → 解码 → 渲染`流程并实现 Demo 来向大家介绍如何在 iOS/Android 平台上手音视频开发。

这里是 Android 第八篇：**Android 视频编码 Demo**。这个 Demo 里包含以下内容：

- 1）实现一个视频采集模块；
- 2）实现两个视频编码模块 `ByteBuffer`、`Surface`，支持 H.264/H.265；
- 3）串联视频采集和编码模块，将采集到的视频数据输入给编码模块进行编码，并存储为文件；
- 4）详尽的代码注释，帮你理解代码逻辑和原理。


在本文中，我们将详解一下 Demo 的具体实现和源码。读完本文内容相信就能帮你掌握相关知识。

>>不过，如果你的需求是：1）直接获得全部工程源码；2）想进一步咨询音视频技术问题；3）咨询音视频职业发展问题。可以根据自己的需要考虑是否加入『关键帧的音视频开发圈』。
>>
>>![长按识别二维码→加入我们](assets/img/keyframe-zsxq.png)
>>_长按识别二维码→加入我们_


想要了解视频编码，可以看看这几篇：

- [《视频编码（1）：H.264（AVC）》](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)
- [《视频编码（2）：H.265（HEVC）》](https://mp.weixin.qq.com/s/CAgnEIaQQ-6lT1KjESRApA)
- [《视频编码（3）：H.266（VVC）》](https://mp.weixin.qq.com/s/w7xIji29JKYf8cI2BxHZyw)


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


## 2、视频 ByteBuffer 编码模块

在实现视频编码模块之前，我们先实现一个视频编码配置类 `KFVideoEncoderConfig`：

`KFVideoEncoderConfig.java`

```java
public class KFVideoEncoderConfig {
    public Size size = new Size(720,1280);
    public int bitrate = 4 * 1024 * 1024;
    public int fps = 30;
    public int gop = 30 * 4;
    public boolean isHEVC = false;
    public int profile = MediaCodecInfo.CodecProfileLevel.AVCProfileBaseline;
    public int profileLevel = MediaCodecInfo.CodecProfileLevel.AVCLevel1;

    public KFVideoEncoderConfig() {

    }
}
```

这里可以配置各种编码参数，但不同机型支持能力不同，通过此配置生成编码格式描述 `MediaFormat`。


接下来，我们来实现一个视频编码模块 `KFByteBufferCodec `，编码模块 `KFByteBufferCodec` 的实现与 [《Android 音频编码 Demo》](https://mp.weixin.qq.com/s?__biz=MjM5MTkxOTQyMQ==&mid=2257485614&idx=1&sn=636683b05eacc4f4728fb2849a445ded&chksm=a5d4e37c92a36a6a8a4d3d1991cebc5fde3775086b4da22d78924d7f965e78fccbf6605c0be5&token=991245230&lang=zh_CN#rd) 中一样，这里就不再重复介绍了，其接口如下

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

上面是 `KFByteBufferCodec` 接口的设计，与音频编码对比区别如下：

- 1）音频编码使用了继承类 `KFAudioByteBufferEncoder`，视频编码则直接使用类 `KFByteBufferCodec`。
	- 音频编码使用了继承类 `KFByteBufferCodec`，目的是切割合适大小的数据 `2048` 送入编码器,因为 AAC 数据编码每帧大小为 `1024 * 2（位深 16 Bit）`。
	- 视频编码使用了类 `KFByteBufferCodec`。
- 2）外层使用构造方法时配置参数修改：
	- `setup` 接口 `mInputMediaFormat` 需要设置视频编码的格式描述。

更具体细节见上述代码及其注释。


## 3、视频 Surface 编码模块

接下来，我们来实现一个视频编码模块 `KFVideoSurfaceEncoder`，在这里输入采集后的数据，输出编码后的数据，同样也需要实现接口 `KFMediaCodecInterface`，参考模块 `KFByteBufferCodec`。

`KFVideoSurfaceEncoder.java`

```java
public class KFVideoSurfaceEncoder implements KFMediaCodecInterface {
    private static final String TAG = "KFVideoSurfaceEncoder";
    private KFMediaCodecListener mListener = null; ///< 回调。
    private KFGLContext mEGLContext = null; ///< GL 上下文。
    private KFGLFilter mFilter = null; ///< 渲染到 Surface 特效。
    private MediaCodec mEncoder = null; ///< 编码器。
    private Surface mSurface = null; ///< 渲染 Surface 缓存。

    private HandlerThread mEncoderThread = null; ///< 编码线程。
    private Handler mEncoderHandler = null;
    private Handler mMainHandler = new Handler(Looper.getMainLooper()); ///< 主线程。
    private MediaCodec.BufferInfo mBufferInfo = new MediaCodec.BufferInfo();
    private long mLastInputPts = 0;
    private MediaFormat mOutputFormat = null; ///< 输出格式描述。
    private MediaFormat mInputFormat = null; ///< 输入格式描述。

    public KFVideoSurfaceEncoder() {

    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    @Override
    public void setup(boolean isEncoder,MediaFormat mediaFormat, KFMediaCodecListener listener, EGLContext eglShareContext) {
        mInputFormat = mediaFormat;
        mListener = listener;

        mEncoderThread = new HandlerThread("KFSurfaceEncoderThread");
        mEncoderThread.start();
        mEncoderHandler = new Handler((mEncoderThread.getLooper()));

        mEncoderHandler.post(()->{
            if (mInputFormat == null) {
                _callBackError(KFMediaCodecInterfaceErrorParams,"mInputFormat == null");
                return;
            }

            ///< 初始化编码器。
            boolean setupSuccess = _setupEnocder();
            if (setupSuccess) {
                mEGLContext = new KFGLContext(eglShareContext,mSurface);
                mEGLContext.bind();
                ///< 初始化特效，用于纹理渲染到编码器 Surface 上。
                _setupFilter();
                mEGLContext.unbind();
            }
        });
    }

    @Override
    public MediaFormat getOutputMediaFormat() {
        return mOutputFormat;
    }

    @Override
    public MediaFormat getInputMediaFormat() {
        return mInputFormat;
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    @Override
    public void release() {
        mEncoderHandler.post(()->{
            ///< 释放编码器。
            if (mEncoder != null) {
                try {
                    mEncoder.stop();
                    mEncoder.release();
                } catch (Exception e) {
                    Log.e(TAG, "release: " + e.toString());
                }
                mEncoder = null;
            }

            ///< 释放 GL 特效上下文。
            if (mEGLContext != null) {
                mEGLContext.bind();
                if (mFilter != null) {
                    mFilter.release();
                    mFilter = null;
                }
                mEGLContext.unbind();

                mEGLContext.release();
                mEGLContext = null;
            }

            ///< 释放 Surface 缓存。
            if (mSurface != null) {
                mSurface.release();
                mSurface = null;
            }

            mEncoderThread.quit();
        });
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    @Override
    public int processFrame(KFFrame inputFrame) {
        if (inputFrame == null || mEncoderHandler == null) {
            return KFMediaCodeProcessParams;
        }
        KFTextureFrame frame = (KFTextureFrame)inputFrame;

        mEncoderHandler.post(()-> {
            if (mEncoder != null && mEGLContext != null) {
                if (frame.isEnd) {
                    ///< 最后一帧标记。
                    mEncoder.signalEndOfInputStream();
                } else {
                    ///< 最近一帧时间戳。
                    mLastInputPts = frame.usTime();
                    mEGLContext.bind();
                    ///< 渲染纹理到编码器 Surface 设置视口。
                    GLES20.glViewport(0, 0, frame.textureSize.getWidth(), frame.textureSize.getHeight());
                    mFilter.render(frame);
                    ///< 设置时间戳。
                    mEGLContext.setPresentationTime(frame.usTime() * 1000);
                    mEGLContext.swapBuffers();
                    mEGLContext.unbind();

                    ///< 获取编码后的数据，尽量拿出最多的数据出来，回调给外层。
                    long outputDts = -1;
                    while (outputDts < mLastInputPts){
                        int bufferIndex = 0;
                        try {
                            bufferIndex = mEncoder.dequeueOutputBuffer(mBufferInfo, 10 * 1000);
                        } catch (Exception e) {
                            Log.e(TAG, "Unexpected MediaCodec exception in dequeueOutputBufferIndex, " + e);
                            _callBackError(KFMediaCodecInterfaceErrorDequeueOutputBuffer,e.getMessage());
                            return;
                        }

                        if (bufferIndex >= 0) {
                            ByteBuffer byteBuffer = mEncoder.getOutputBuffer(bufferIndex);
                            if (byteBuffer != null) {
                                outputDts = mBufferInfo.presentationTimeUs;
                                if (mListener != null) {
                                    KFBufferFrame encodeFrame = new KFBufferFrame();
                                    encodeFrame.buffer = byteBuffer;
                                    encodeFrame.bufferInfo = mBufferInfo;
                                    mListener.dataOnAvailable(encodeFrame);
                                }
                            } else {
                                break;
                            }

                            try {
                                mEncoder.releaseOutputBuffer(bufferIndex, false);
                            } catch (Exception e) {
                                Log.e(TAG, e.toString());
                                return;
                            }
                        } else {
                            if (bufferIndex == MediaCodec.INFO_OUTPUT_FORMAT_CHANGED) {
                                mOutputFormat = mEncoder.getOutputFormat();
                            }
                            break;
                        }
                    }
                }
            }
        });

        return KFMediaCodeProcessSuccess;
    }

    @Override
    public void flush() {
        mEncoderHandler.post(()-> {
            ///< 刷新缓冲区。
            if (mEncoder != null) {
                try {
                    mEncoder.flush();
                } catch (Exception e) {
                    Log.e(TAG, "flush error!" + e);
                }
            }
        });
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    private boolean _setupEnocder() {
        ///< 初始化编码器。
        try {
            String mimeType = mInputFormat.getString(MediaFormat.KEY_MIME);
            mEncoder = MediaCodec.createEncoderByType(mimeType);
            mEncoder.configure(mInputFormat, null, null, MediaCodec.CONFIGURE_FLAG_ENCODE);
        } catch (IOException e) {
            Log.e(TAG, "createEncoderByType" + e);
            _callBackError(KFMediaCodecInterfaceErrorCreate,e.getMessage());
            return false;
        }

        ///< 创建 Surface。
        mSurface = mEncoder.createInputSurface();

        ///< 开启编码器。
        try {
            mEncoder.start();
        } catch (Exception e) {
            Log.e(TAG, "start" +  e );
            _callBackError(KFMediaCodecInterfaceErrorStart,e.getMessage());
            return false;
        }

        return true;
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    private void _setupFilter() {
        ///< 创建渲染模块，渲染到编码器 Surface。
        if (mFilter == null) {
            mFilter = new KFGLFilter(true, KFGLBase.defaultVertexShader,KFGLBase.defaultFragmentShader);
        }
    }

    private void _callBackError(int error, String errorMsg){
        ///< 出错回调。
        if (mListener != null) {
            mMainHandler.post(()->{
                mListener.onError(error,TAG + errorMsg);
            });
        }
    }
}
```


上面是 `KFVideoSurfaceEncoder` 的实现，与视频编码 `KFByteBufferCodec` 对比区别如下：

- 1）数据源输入不同。
	- `KFByteBufferCodec` 输入为 YUV 数据 `KFBufferFrame`。
	- `KFVideoSurfaceEncoder` 输入为纹理数据 `KFTextureFrame`。
- 2）编码流水线不同。
	- `KFByteBufferCodec` 输入 YUV 数据进行编码。
	- `KFVideoSurfaceEncoder` 输入为纹理数据，执行 OpenGL 渲染，将纹理渲染到编码器缓存 `mSurface`。使用 `mFilter.render` 进行渲染，同时设置时间戳 `setPresentationTime`，交换前后台缓冲区 `swapBuffers` ，将纹理数据刷新到了 `mSurface`。最后取出编码后数据，需要注意 `releaseOutputBuffer` 方法第 2 个参数 `render` 设置为 true。
- 3）使用场景不同。
	- `KFVideoSurfaceEncoder ` 适用于输入数据为纹理的情况，例如采集后添加特效。
	- `KFByteBufferCodec` 适用于非纹理数据，例如游戏直播、录屏直播、图片转视频等输入数据为 ByteBuffer，此时没必要再做数据转换。


更具体细节见上述代码及其注释。




## 4、采集视频数据进行 H.264/H.265 编码和存储

我们在一个 `MainActivity` 中来实现视频采集及编码逻辑，因为 Android 编码的默认输出 AnnexB 码流格式，所以这里不需要转换。


`MainActivity.java`

```java
public class MainActivity extends AppCompatActivity {

    private KFIVideoCapture mCapture; ///< 采集器。
    private KFVideoCaptureConfig mCaptureConfig; ///< 采集配置。
    private KFRenderView mRenderView; ///< 渲染视图。
    private KFGLContext mGLContext; ///< OpenGL 上下文。

    private KFVideoEncoderConfig mEncoderConfig; ///< 编码配置。
    private KFMediaCodecInterface mEncoder; ///< 编码。
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
                ///< 创建编码器。
                if (mEncoder == null) {
                    mEncoder = new KFVideoSurfaceEncoder();
                    MediaFormat mediaFormat = KFAVTools.createVideoFormat(mEncoderConfig.isHEVC,mEncoderConfig.size, MediaCodecInfo.CodecCapabilities.COLOR_FormatSurface,mEncoderConfig.bitrate,mEncoderConfig.fps,mEncoderConfig.gop / mEncoderConfig.fps,mEncoderConfig.profile,mEncoderConfig.profileLevel);
                    mEncoder.setup(true,mediaFormat,mVideoEncoderListener,mGLContext.getContext());
                    ((Button)view).setText("停止");
                } else {
                    mEncoder.release();
                    mEncoder = null;
                    ((Button)view).setText("开始");
                }
            }
        });
        addContentView(startButton, startParams);

        ///< 创建采集器。
        mCaptureConfig = new KFVideoCaptureConfig();
        mCaptureConfig.cameraFacing = LENS_FACING_FRONT;
        mCaptureConfig.resolution = new Size(720,1280);
        mCaptureConfig.fps = 30;
        boolean useCamera2 = false;
        if (useCamera2) {
            mCapture = new KFVideoCaptureV2();
        } else {
            mCapture = new KFVideoCaptureV1();
        }
        mCapture.setup(this,mCaptureConfig,mVideoCaptureListener,mGLContext.getContext());
        mCapture.startRunning();

        mEncoderConfig = new KFVideoEncoderConfig();
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
            ///< 采集数据回调，进入编码器。
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

        @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
        @Override
        public void dataOnAvailable(KFFrame frame) {
            ///< 编码数据回调写入本地文件。
            if (mStream == null) {
                try {
                    mStream = new FileOutputStream(Environment.getExternalStorageDirectory().getPath() + "/test.h264");
                } catch (FileNotFoundException e) {
                    e.printStackTrace();
                }
            }
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

上面是 `MainActivity` 的实现，主要分为以下几个部分：

- 1）创建 OpenGL 上下文。
	- 创建上下文 `mGLContext`，这样好处是采集与预览可以共享，提高扩展性。 
- 2）创建采集实例。
	- 这里需要注意的是，我们通过开关 `useCamera2` 选择 `Camera` 或 `Camera2`。
	- 参数配置 `mCaptureConfig`，可自定义摄像头方向、帧率、分辨率。
- 3）采集数据回调 `onFrameAvailable`，将数据输入给渲染模块与编码模块。
- 4）编码数据回调 `KFMediaCodecListener` 的 `dataOnAvailable` 中，将编码数据存储为 H.264/H.265 文件。


## 5、用工具播放 H.264/H.265 文件


完成视频采集和编码后，可以将 `sdcard` 文件夹下面的 `test.h264` 或 `test.h265` 文件拷贝到电脑上，使用 `ffplay` 播放来验证一下效果是否符合预期：

```c
$ ffplay -i test.h264
$ ffplay -i test.h265
```

关于播放 H.264/H.265 文件的工具，可以参考[《FFmpeg 工具》第 2 节 ffplay 命令行工具](https://mp.weixin.qq.com/s/Rl7fxOP-YH37mQEvGxhfUA)和[《可视化音视频分析工具》第 2.1 节 StreamEye](https://mp.weixin.qq.com/s/jCYih3qgEIUctuWxn0aTGQ)。






