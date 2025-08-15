---
title: Android AVDemo（12）：视频解码，代码开源并提供解析
description: 介绍 Android 视频解码流程和原理，并提供 Demo 源码和解析。
author: Keyframe
date: 2025-02-23 11:18:08 +0800
categories: [音视频源码示例]
tags: [音视频源码示例, 音视频, Android, 视频, 编解码]
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

这里是 Android 第十二篇：**Android 视频解码 Demo**。这个 Demo 里包含以下内容：

- 1）实现一个视频解封装模块；
- 2）实现两个视频解码模块 `ByteBuffer`、`Surface`；
- 3）串联视频解封装和解码模块，将解封装的 H.264/H.265 数据输入给解码模块进行解码，并存储解码后的 YUV 数据与纹理数据渲染；
- 4）详尽的代码注释，帮你理解代码逻辑和原理。



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


## 2、视频 ByteBuffer 解码模块

接下来，我们来实现一个视频解码模块 `KFByteBufferCodec`，解码模块 `KFByteBufferCodec` 的实现与 [《Android 音频编码 Demo》](https://mp.weixin.qq.com/s/yI6XMPbLfNvaJNZEmQkNqQ) 中一样，这里就不再重复介绍了，其接口如下：


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

上面是 `KFByteBufferCodec` 接口的设计，与视频编码对比区别如下：

- 1）外层使用构造方法时配置参数修改：
	- `setup` 接口 `mInputMediaFormat` 需要设置视频解码的格式描述，`isEncoder` 设置为解码 `false`。

## 3、视频 Surface 解码模块

接下来，我们来实现一个视频解码模块 `KFVideoSurfaceDecoder`，在这里输入解封装后的编码数据，输出解码后的数据，同样也需要实现接口 `KFMediaCodecInterface`，参考模块 `KFByteBufferCodec`。


`KFVideoSurfaceDecoder.java`

```java
public class KFVideoSurfaceDecoder implements  KFMediaCodecInterface {
    private static final String TAG = "KFVideoSurfaceDecoder";
    private KFMediaCodecListener mListener = null; ///< 回调。
    private MediaCodec mDecoder = null; ///< 解码器。
    private ByteBuffer[] mInputBuffers; ///< 解码器输入缓存。
    private MediaFormat mInputMediaFormat = null; ///< 输入格式描述。
    private MediaFormat mOutMediaFormat = null; ///< 输出格式描述。
    private KFGLContext mEGLContext = null; ///< OpenGL 上下文。
    private KFSurfaceTexture mSurfaceTexture = null; ///< 纹理缓存。
    private Surface mSurface = null; ///< 纹理缓存，对应 Surface。
    private KFGLFilter mOESConvert2DFilter; ///< 特效。

    private long mLastInputPts = 0; ///< 输入数据最后一帧时间戳。
    private List<KFBufferFrame> mList = new ArrayList<>();
    private ReentrantLock mListLock = new ReentrantLock(true);

    private HandlerThread mDecoderThread = null; ///< 解码线程。
    private Handler mDecoderHandler = null;
    private HandlerThread mRenderThread = null; ///< 渲染线程。
    private Handler mRenderHandler = null;
    private Handler mMainHandler = new Handler(Looper.getMainLooper()); ///< 主线程。

    public KFVideoSurfaceDecoder() {

    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    @Override
    public void setup(boolean isEncoder,MediaFormat mediaFormat,KFMediaCodecListener listener, EGLContext eglShareContext) {
        mInputMediaFormat = mediaFormat;
        mListener = listener;

        ///< 创建解码线程。
        mDecoderThread = new HandlerThread("KFVideoSurfaceDecoderThread");
        mDecoderThread.start();
        mDecoderHandler = new Handler((mDecoderThread.getLooper()));

        ///< 创建渲染线程。
        mRenderThread = new HandlerThread("KFVideoSurfaceRenderThread");
        mRenderThread.start();
        mRenderHandler = new Handler((mRenderThread.getLooper()));

        mDecoderHandler.post(()->{
            if (mInputMediaFormat == null) {
                _callBackError(KFMediaCodecInterfaceErrorParams,"mInputMediaFormat null");
                return;
            }

            ///< 创建 OpenGL 上下文、纹理缓存、纹理缓存 Surface、OES 转 2D 数据。
            mEGLContext = new KFGLContext(eglShareContext);
            mEGLContext.bind();
            mSurfaceTexture = new KFSurfaceTexture(mSurfaceTextureListener);
            mSurfaceTexture.getSurfaceTexture().setDefaultBufferSize(mInputMediaFormat.getInteger(MediaFormat.KEY_WIDTH),mInputMediaFormat.getInteger(MediaFormat.KEY_HEIGHT));
            mSurface = new Surface(mSurfaceTexture.getSurfaceTexture());
            mOESConvert2DFilter = new KFGLFilter(false, KFGLBase.defaultVertexShader,KFGLBase.oesFragmentShader);
            mEGLContext.unbind();

            _setupDecoder();
        });
    }

    @Override
    public MediaFormat getOutputMediaFormat() {
        return mOutMediaFormat;
    }

    @Override
    public MediaFormat getInputMediaFormat() {
        return mInputMediaFormat;
    }

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN_MR1)
    @Override
    public void release() {
        mDecoderHandler.post(()-> {
            ///< 释放解码器、GL 上下文、数据缓存、SurfaceTexture。
            if (mDecoder != null) {
                try {
                    mDecoder.stop();
                    mDecoder.release();
                } catch (Exception e) {
                    Log.e(TAG, "release: " + e.toString());
                }
                mDecoder = null;
            }

            mEGLContext.bind();
            if (mSurfaceTexture != null) {
                mSurfaceTexture.release();
                mSurfaceTexture = null;
            }
            if (mSurface != null) {
                mSurface.release();
                mSurface = null;
            }
            if (mOESConvert2DFilter != null) {
                mOESConvert2DFilter.release();
                mOESConvert2DFilter = null;
            }
            mEGLContext.unbind();

            if (mEGLContext != null) {
                mEGLContext.release();
                mEGLContext = null;
            }

            mListLock.lock();
            mList.clear();
            mListLock.unlock();

            mDecoderThread.quit();
            mRenderThread.quit();
        });
    }

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    @Override
    public void flush() {
        mDecoderHandler.post(()-> {
            ///< 刷新解码器缓冲区。
            if (mDecoder == null) {
                return;
            }

            try {
                mDecoder.flush();
            } catch (Exception e) {
                Log.e(TAG, "flush" + e);
            }

            mListLock.lock();
            mList.clear();
            mListLock.unlock();
        });
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    @Override
    public int processFrame(KFFrame inputFrame) {
        if (inputFrame == null) {
            return KFMediaCodeProcessParams;
        }

        KFBufferFrame frame = (KFBufferFrame)inputFrame;
        if (frame.buffer ==null || frame.bufferInfo == null || frame.bufferInfo.size == 0) {
            return KFMediaCodeProcessParams;
        }

        ///< 外层数据进入缓存。
        _appendFrame(frame);

        mDecoderHandler.post(()-> {
            if (mDecoder == null) {
                return;
            }

            ///< 缓存获取数据，尽量多的输入给解码器。
            mListLock.lock();
            int mListSize = mList.size();
            mListLock.unlock();
            while (mListSize > 0) {
                mListLock.lock();
                KFBufferFrame packet = mList.get(0);
                mListLock.unlock();

                int bufferIndex;
                try {
                    ///< 获取解码器输入缓存下标。
                    bufferIndex = mDecoder.dequeueInputBuffer(10 * 1000);
                } catch (Exception e) {
                    Log.e(TAG, "dequeueInputBuffer" + e);
                    return;
                }

                if (bufferIndex >= 0) {
                    ///< 填充数据。
                    mInputBuffers[bufferIndex].clear();
                    mInputBuffers[bufferIndex].put(packet.buffer);
                    mInputBuffers[bufferIndex].flip();
                    try {
                        ///< 数据塞入解码器。
                        mDecoder.queueInputBuffer(bufferIndex, 0, packet.bufferInfo.size, packet.bufferInfo.presentationTimeUs, packet.bufferInfo.flags);
                    } catch (Exception e) {
                        Log.e(TAG, "queueInputBuffer" + e);
                        return;
                    }

                    mLastInputPts = packet.bufferInfo.presentationTimeUs;
                    mListLock.lock();
                    mList.remove(0);
                    mListSize = mList.size();
                    mListLock.unlock();
                } else {
                    break;
                }
            }

            ///< 从解码器拉取尽量多的数据出来。
            long outputDts = -1;
            MediaCodec.BufferInfo outputBufferInfo = new MediaCodec.BufferInfo();
            while (outputDts < mLastInputPts) {
                int bufferIndex;
                try {
                    ///< 获取解码器输出缓存下标。
                    bufferIndex = mDecoder.dequeueOutputBuffer(outputBufferInfo, 10 * 1000);
                } catch (Exception e) {
                    Log.e(TAG, "dequeueOutputBuffer" + e);
                    return;
                }

                if (bufferIndex >= 0) {
                    ///< 释放缓存，第二个参数必须设置位 true，这样数据刷新到指定 surface。
                    mDecoder.releaseOutputBuffer(bufferIndex,true);
                } else {
                    if (bufferIndex == MediaCodec.INFO_OUTPUT_FORMAT_CHANGED) {
                        mOutMediaFormat = mDecoder.getOutputFormat();
                    }
                    break;
                }
            }
        });

        return KFMediaCodeProcessSuccess;
    }


    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    private void _appendFrame(KFBufferFrame frame) {
        ///< 添加数据到缓存 List。
        KFBufferFrame packet = new KFBufferFrame();

        ByteBuffer newBuffer = ByteBuffer.allocateDirect(frame.bufferInfo.size);
        newBuffer.put(frame.buffer).position(0);
        MediaCodec.BufferInfo newInfo = new MediaCodec.BufferInfo();
        newInfo.size = frame.bufferInfo.size;
        newInfo.flags = frame.bufferInfo.flags;
        newInfo.presentationTimeUs = frame.bufferInfo.presentationTimeUs;
        packet.buffer = newBuffer;
        packet.bufferInfo = newInfo;

        mListLock.lock();
        mList.add(packet);
        mListLock.unlock();
    }

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    private boolean _setupDecoder() {
        ///< 初始化解码器。
        try {
            ///< 根据输入格式描述创建解码器。
            String mimetype = mInputMediaFormat.getString(MediaFormat.KEY_MIME);
            mDecoder = MediaCodec.createDecoderByType(mimetype);
        } catch (Exception e) {
            Log.e(TAG, "createDecoderByType" + e);
            _callBackError(KFMediaCodecInterfaceErrorCreate,e.getMessage());
            return false;
        }

        try {
            ///< 配置位 Surface 解码模式。
            mDecoder.configure(mInputMediaFormat, mSurface, null, 0);
        } catch (Exception e) {
            Log.e(TAG, "configure" + e);
            _callBackError(KFMediaCodecInterfaceErrorConfigure,e.getMessage());
            return false;
        }

        try {
            ///< 启动解码器。
            mDecoder.start();
            ///< 获取解码器输入缓存。
            mInputBuffers = mDecoder.getInputBuffers();
        } catch (Exception e) {
            Log.e(TAG, "start" +  e );
            _callBackError(KFMediaCodecInterfaceErrorStart,e.getMessage());
            return false;
        }

        return true;
    }

    private void _callBackError(int error, String errorMsg) {
        ///< 错误回调。
        if (mListener != null) {
            mMainHandler.post(()->{
                mListener.onError(error,TAG + errorMsg);
            });
        }
    }

    private KFSurfaceTextureListener mSurfaceTextureListener = new KFSurfaceTextureListener() {
        ///< SurfaceTexture 数据回调。
        @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
        @Override
        public void onFrameAvailable(SurfaceTexture surfaceTexture) {
            mRenderHandler.post(() -> {
                mEGLContext.bind();
                mSurfaceTexture.getSurfaceTexture().updateTexImage();
                if (mListener != null) {
                    int width = mInputMediaFormat.getInteger(MediaFormat.KEY_WIDTH);
                    int height = mInputMediaFormat.getInteger(MediaFormat.KEY_HEIGHT);
                    int rotation = (mInputMediaFormat.getInteger(MediaFormat.KEY_ROTATION) + 360) % 360;
                    int rotationWidth = (rotation % 360 == 90 || rotation % 360 == 270) ? height : width;
                    int rotationHeight = (rotation % 360 == 90 || rotation % 360 == 270) ? width : height;
                    KFTextureFrame frame = new KFTextureFrame(mSurfaceTexture.getSurfaceTextureId(),new Size(rotationWidth,rotationHeight),mSurfaceTexture.getSurfaceTexture().getTimestamp() * 1000,true);
                    mSurfaceTexture.getSurfaceTexture().getTransformMatrix(frame.textureMatrix);
                    ///< OES 数据转换 2D。
                    KFFrame convertFrame = mOESConvert2DFilter.render(frame);
                    mListener.dataOnAvailable(convertFrame);
                }
                mEGLContext.unbind();
            });
        }
    };
}
```


上面是 `KFVideoSurfaceDecoder` 的实现，与视频解码 `KFByteBufferCodec` 对比区别如下：

- 1）数据输出不同。
	- `KFByteBufferCodec` 输出为 YUV 数据 `KFBufferFrame`。
	- `KFVideoSurfaceEncoder` 输出为纹理数据 `KFTextureFrame`。
- 2）解码流水线不同。
	- `KFVideoSurfaceEncoder` 输出为纹理数据，将数据解码到纹理缓存 `mSurface`。释放缓存 `releaseOutputBuffer` 触发 `KFSurfaceTextureListener` 的 `onFrameAvailable` 回调，需要注意 `releaseOutputBuffer` 方法第 2 个参数 `render` 设置为 true。然后调用 `mSurfaceTexture` 的 `updateTexImage` 将数据刷新到自定义纹理。
- 3）使用场景不同。
	- `KFVideoSurfaceDecoder` 适用于输出数据为纹理的情况，例如播放器。
	- `KFByteBufferCodec` 适用于输出数据非纹理数据，例如抽帧。


更具体细节见上述代码及其注释。


## 4、解封装和解码(ByteBuffer) MP4 文件中的视频部分存储为 YUV 文件

我们在一个 `MainActivity` 中来实现视频解封装及解码逻辑，并将解码后的数据存储为 YUV 文件。

`MainActivity.java`


```java
public class MainActivity extends AppCompatActivity {
    private KFMP4Demuxer mDemuxer; ///< 解封装器。
    private KFDemuxerConfig mDemuxerConfig; ///< 解封装器配置。
    private KFMediaCodecInterface mDecoder = null; ///< 解码器。
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

        ///< 解封装配置。
        mDemuxerConfig = new KFDemuxerConfig();
        mDemuxerConfig.path = Environment.getExternalStorageDirectory().getPath() + "/2.mp4";
        mDemuxerConfig.demuxerType = KFMediaBase.KFMediaType.KFMediaVideo;
        if (mStream == null) {
            try {
                mStream = new FileOutputStream(Environment.getExternalStorageDirectory().getPath() + "/test.yuv");
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
                ///< 创建解封装。
                if (mDemuxer == null) {
                    mDemuxer = new KFMP4Demuxer(mDemuxerConfig,mDemuxerListener);
                    ///< 创建解码器。
                    mDecoder = new KFByteBufferCodec();
                    mDecoder.setup(false,mDemuxer.videoMediaFormat(),mDecoderListener,null);

                    ///< 循环读取数据输入给解码器。
                    MediaCodec.BufferInfo bufferInfo = new MediaCodec.BufferInfo();
                    ByteBuffer nextBuffer = mDemuxer.readVideoSampleData(bufferInfo);
                    while (nextBuffer != null) {
                        mDecoder.processFrame(new KFBufferFrame(nextBuffer,bufferInfo));
                        nextBuffer = mDemuxer.readVideoSampleData(bufferInfo);
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
        ///< 解封装回调出错。
        public void demuxerOnError(int error, String errorMsg) {
            Log.i("KFDemuxer","error" + error + "msg" + errorMsg);
        }
    };

    private KFMediaCodecListener mDecoderListener = new KFMediaCodecListener() {
        @Override
        ///< 解码回调出粗。
        public void onError(int error, String errorMsg) {

        }

        @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
        @Override
        ///< 解码后数据回调。
        public void dataOnAvailable(KFFrame frame) {
            if (frame == null) {
                return;
            }

            KFBufferFrame bufferFrame = (KFBufferFrame)frame;
            if (bufferFrame.buffer == null) {
                return;
            }

            MediaFormat mediaFormat = mDecoder.getOutputMediaFormat();
            int width = mediaFormat.getInteger("width");
            int height = mediaFormat.getInteger("height");
            int cropLeft = mediaFormat.getInteger("crop-left");
            int cropRight = mediaFormat.getInteger("crop-right");
            int cropTop = mediaFormat.getInteger("crop-top");
            int cropBottom = mediaFormat.getInteger("crop-bottom");
            int colorFormat = mediaFormat.getInteger("color-format"); //COLOR_FormatYUV420SemiPlanar

            ///< YUV 数据存储本地。
            try {
                byte[] dst = new byte[(int) (width*height*1.5)];
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

- 1）通过启动视频解封装来驱动整个解封装和解码流程。
	- 在 `onClick` 中实现开始动作并且循环读取数据塞入解码器。
- 2）在解码模块 `KFByteBufferCodec` 的数据回调中获取解码后的 YUV 数据存储为文件。
	- 在 `KFMediaCodecListener` 的 `dataOnAvailable` 中实现。
	- 这里按照 NV12 的 YUV 格式存储。


## 5、解封装和解码(Surface) MP4 文件中的视频纹理进行渲染

我们在一个 `MainActivity` 中来实现视频解封装及解码逻辑，并将解码后的数据进行渲染。

`MainActivity.java`


```java
public class MainActivity extends AppCompatActivity {
    private KFMP4Demuxer mDemuxer; ///< 解封装器。
    private KFDemuxerConfig mDemuxerConfig; ///< 解封装器配置。
    private KFMediaCodecInterface mDecoder = null; ///< 解码。
    private KFRenderView mRenderView; ///< 渲染。
    private KFGLContext mGLContext; ///< GL 上下文。
    private Timer mTimer;

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
        //< 创建渲染视图。
        mRenderView = new KFRenderView(this,mGLContext.getContext());
        WindowManager windowManager = (WindowManager)this.getSystemService(this.WINDOW_SERVICE);
        Rect outRect = new Rect();
        windowManager.getDefaultDisplay().getRectSize(outRect);
        FrameLayout.LayoutParams params = new FrameLayout.LayoutParams(outRect.width(), outRect.height());
        addContentView(mRenderView,params);

        ///< 创建解封装器配置。
        mDemuxerConfig = new KFDemuxerConfig();
        mDemuxerConfig.path = Environment.getExternalStorageDirectory().getPath() + "/2.mp4";
        mDemuxerConfig.demuxerType = KFMediaBase.KFMediaType.KFMediaVideo;

        FrameLayout.LayoutParams startParams = new FrameLayout.LayoutParams(200, 120);
        startParams.gravity = Gravity.CENTER_HORIZONTAL;
        Button startButton = new Button(this);
        startButton.setTextColor(Color.BLUE);
        startButton.setText("开始");
        startButton.setVisibility(View.VISIBLE);
        startButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                ///< 创建解封装与解码器。
                if (mDemuxer == null) {
                    mDemuxer = new KFMP4Demuxer(mDemuxerConfig,mDemuxerListener);
                    mDecoder = new KFVideoSurfaceDecoder();
                    mDecoder.setup(false, mDemuxer.videoMediaFormat(),mDecoderListener,mGLContext.getContext());
                    ((Button)view).setText("停止");
                } else {
                    mDemuxer.release();
                    mDemuxer = null;
                    mDecoder.release();
                    mDecoder = null;
                    ((Button)view).setText("开始");
                }
            }
        });
        addContentView(startButton, startParams);

        Timer timer = new Timer();
        TimerTask task = new TimerTask() {
            @Override
            public void run() {
                ///< 根据 Timer 回调读取解封装数据给解码器。
                if (mDemuxer != null) {
                    MediaCodec.BufferInfo bufferInfo = new MediaCodec.BufferInfo();
                    ByteBuffer byteBuffer = mDemuxer.readVideoSampleData(bufferInfo);
                    if (byteBuffer != null) {
                        KFBufferFrame frame = new KFBufferFrame();
                        frame.bufferInfo = bufferInfo;
                        frame.buffer = byteBuffer;
                        mDecoder.processFrame(frame);
                    }
                }
            }
        };
        timer.schedule(task,0,33);
    }

    private KFDemuxerListener mDemuxerListener = new KFDemuxerListener() {
        @Override
        ///< 解封装回调出错。
        public void demuxerOnError(int error, String errorMsg) {
            Log.i("KFDemuxer","error" + error + "msg" + errorMsg);
        }
    };

    private KFMediaCodecListener mDecoderListener = new KFMediaCodecListener() {
        @Override
        ///< 解码回调出错。
        public void onError(int error, String errorMsg) {

        }

        ///< 解码回调进行渲染。
        @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
        @Override
        public void dataOnAvailable(KFFrame frame) {
            mRenderView.render((KFTextureFrame) frame);
        }
    };
}
```

上面是 `MainActivity` 的实现，其中主要包含这几个部分：

- 1）通过启动视频解封装来驱动整个解封装和解码流程。
	- 在 `onClick` 中实现开始动作。
- 2）启动 Timer 模块指定间隔进行解码渲染。
	- 启动 Timer 模块 `mTimer`。
	- Timer 中 调用获取视频数据 `readVideoSampleData`，输入到解码器 `processFrame`。 
- 3）在解码模块 `KFVideoSurfaceDecoder` 的数据回调中获取纹理数据进行渲染。
	- 在 `KFMediaCodecListener` 的 `dataOnAvailable` 中进行渲染到 `mRenderView`。


## 6、用工具播放 YUV 文件

完成 Demo 后，可以将 `sdcard` 文件夹下面的 `test.yuv` 文件拷贝到电脑上，使用 `ffplay` 播放来验证一下效果是否符合预期：

```c
$ ffplay -f rawvideo -pix_fmt nv12 -video_size 1280x720 -i test.yuv
```

注意这里的参数要对齐在工程中存储的 YUV 格式，我们 Demo 中的视频尺寸是 `1280x720`，我们是用 `NV12` 格式存储的 YUV。

关于播放 YUV 文件的工具，可以参考[《FFmpeg 工具》第 2 节 ffplay 命令行工具](https://mp.weixin.qq.com/s/Rl7fxOP-YH37mQEvGxhfUA)和[《可视化音视频分析工具》第 1.2 节 YUVToolkit 或 1.3 节 YUVView](https://mp.weixin.qq.com/s/jCYih3qgEIUctuWxn0aTGQ)。












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

