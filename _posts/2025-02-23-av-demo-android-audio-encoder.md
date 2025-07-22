---
title: Android AVDemo（2）：音频编码，代码开源并提供解析
description: 介绍 Android 音频编码的流程和原理，并提供 Demo 源码和解析。
author: Keyframe
date: 2025-02-23 10:08:08 +0800
categories: [音视频源码示例]
tags: [音视频源码示例, 音视频, Android, 音频, 编解码]
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

这里是 Android 第二篇：**Android 音频编码 Demo**。这个 Demo 里包含以下内容：

- 1）实现一个音频采集模块；
- 2）实现一个音频编码模块；
- 3）串联音频采集和编码模块，将采集到的音频数据输入给 AAC 编码模块进行编码和存储；
- 4）详尽的代码注释，帮你理解代码逻辑和原理；


在本文中，我们将详解一下 Demo 的具体实现和源码。读完本文内容相信就能帮你掌握相关知识。

>>不过，如果你的需求是：1）直接获得全部工程源码；2）想进一步咨询音视频技术问题；3）咨询音视频职业发展问题。可以根据自己的需要考虑是否加入『关键帧的音视频开发圈』，这是一个收费的社群服务，我们在这里答疑解惑和分享资料，你可以通过下面的二维码加入。
>>![长按识别二维码→加入我们](assets/img/keyframe-zsxq.png)
>>_长按识别二维码→加入我们_


## 1、音频采集模块

在这个 Demo 中，音频采集模块 `KFAudioCapture` 的实现与 [Android 音频采集 Demo](https://mp.weixin.qq.com/s?__biz=MjM5MTkxOTQyMQ==&mid=2257484867&idx=1&sn=d857104930a86de8ab0bdf2358ca6283&chksm=a5d4e01192a36907ac113fcac5807807cc10d587713e0ff3f3bff116b7e3d15fadee92c51fe4&scene=21&token=336104652&lang=zh_CN#wechat_redirect) 中一样，这里就不再重复介绍了，其接口如下：


`KFAudioCapture.java`

```java
public class KFAudioCapture {
	public KFAudioCapture(KFAudioCaptureConfig config,KFAudioCaptureListener listener);
	public void startRunning(); ///< 开始采集音频数据。
	public void stopRunning(); ///< 停止采集音频数据。
	public void release(); ///< 释放音频采集。
}
```

## 2、音频编码模块

我们定义了接口类 `KFMediaCodecInterface`，后续编解码模块实现这个接口即可。需要关注 `setup` 接口的参数 isEncoder 代表是否使用编码功能，mediaFormat 代表输入数据格式描述。

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
    public void setup(boolean isEncoder, MediaFormat mediaFormat, KFMediaCodecListener listener, EGLContext eglShareContext);
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

接下来，我们来实现一个音频编码模块 `KFByteBufferCodec`，需要实现上面的接口 `KFMediaCodecInterface `，在这里输入采集后的数据，输出编码后的数据。这里命名为 KFByteBufferCodec，主要因为它可以支持音视频编解码多个功能。

`KFByteBufferCodec.java`

```java
public class KFByteBufferCodec implements KFMediaCodecInterface {
    public static final int KFByteBufferCodecErrorParams = -2500;
    public static final int KFByteBufferCodecErrorCreate = -2501;
    public static final int KFByteBufferCodecErrorConfigure = -2502;
    public static final int KFByteBufferCodecErrorStart = -2503;

    private static final int KFByteBufferCodecInputBufferMaxCache = 20 * 1024 * 1024;
    private static final String TAG = "KFByteBufferCodec";
    private KFMediaCodecListener mListener = null; ///< 回调
    private MediaCodec mMediaCodec = null; ///< Codec 实例
    private ByteBuffer[] mInputBuffers; ///<  Codec 输入缓冲区
    private MediaFormat mInputMediaFormat = null; ///< 输入数据格式描述
    private MediaFormat mOutMediaFormat = null; ///< 输出数据格式描述

    private long mLastInputPts = 0; ///< 上一帧时间戳
    private List<KFBufferFrame> mList = new ArrayList<>(); ///< 输入数据缓存
    private int mListCacheSize = 0; ///< 输入数据缓存数量
    private ReentrantLock mListLock = new ReentrantLock(true); ///< 数据缓存锁
    private boolean mIsEncoder = true;

    private HandlerThread mCodecThread = null; ///< Codec 线程
    private Handler mCodecHandler = null;
    private Handler mMainHandler = new Handler(Looper.getMainLooper()); ///< 主线程

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    @Override
    public void setup(boolean isEncoder,MediaFormat mediaFormat, KFMediaCodecListener listener, EGLContext eglShareContext) {
        mListener = listener;
        mInputMediaFormat = mediaFormat;
        mIsEncoder = isEncoder;

        mCodecThread = new HandlerThread("KFByteBufferCodecThread");
        mCodecThread.start();
        mCodecHandler = new Handler((mCodecThread.getLooper()));

        mCodecHandler.post(()->{
            if(mInputMediaFormat == null){
                _callBackError(KFByteBufferCodecErrorParams,"mInputMediaFormat null");
                return;
            }
            ///< 初始化 Codec 实例。
            _setupCodec();
        });
    }

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    public void  release() {
        ///< 释放 Codec 实例、输入缓存。
        mCodecHandler.post(()-> {
            if(mMediaCodec != null){
                try {
                    mMediaCodec.stop();
                    mMediaCodec.release();
                } catch (Exception e) {
                    Log.e(TAG, "release: " + e.toString());
                }
                mMediaCodec = null;
            }

            mListLock.lock();
            mList.clear();
            mListCacheSize = 0;
            mListLock.unlock();

            mCodecThread.quit();
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

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    @Override
    public int processFrame(KFFrame inputFrame) {
        ///< 处理输入帧数据。
        if(inputFrame == null){
            return KFMediaCodeProcessParams;
        }

        KFBufferFrame frame = (KFBufferFrame)inputFrame;
        if(frame.buffer ==null || frame.bufferInfo == null || frame.bufferInfo.size == 0){
            return KFMediaCodeProcessParams;
        }

        ///< 先添加到缓冲区，一旦缓冲区满则返回 KFMediaCodeProcessAgainLater。
        boolean appendSuccess = _appendFrame(frame);
        if(!appendSuccess){
            return KFMediaCodeProcessAgainLater;
        }

        mCodecHandler.post(()-> {
            if(mMediaCodec == null){
                return;
            }

            ///< 子线程处理编解码，从队列取出一组数据，能塞多少就塞多少数据。
            mListLock.lock();
            int mListSize = mList.size();
            mListLock.unlock();
            while (mListSize > 0){
                mListLock.lock();
                KFBufferFrame packet = mList.get(0);
                mListLock.unlock();

                int bufferIndex;
                try {
                    bufferIndex = mMediaCodec.dequeueInputBuffer(10 * 1000);
                } catch (Exception e) {
                    Log.e(TAG, "dequeueInputBuffer" + e);
                    return;
                }

                if (bufferIndex >= 0) {
                    mInputBuffers[bufferIndex].clear();
                    mInputBuffers[bufferIndex].put(packet.buffer);
                    mInputBuffers[bufferIndex].flip();
                    try {
                        mMediaCodec.queueInputBuffer(bufferIndex, 0, packet.bufferInfo.size, packet.bufferInfo.presentationTimeUs, packet.bufferInfo.flags);
                    } catch (Exception e) {
                        Log.e(TAG, "queueInputBuffer" + e);
                        return;
                    }

                    mLastInputPts = packet.bufferInfo.presentationTimeUs;
                    mListLock.lock();
                    mList.remove(0);
                    mListSize = mList.size();
                    mListCacheSize -= packet.bufferInfo.size;
                    mListLock.unlock();
                } else {
                    break;
                }
            }

            ///< 获取 Codec 后的数据，一样的策略，尽量拿出最多的数据出来，回调给外层。
            long outputDts = -1;
            MediaCodec.BufferInfo outputBufferInfo = new MediaCodec.BufferInfo();
            while (outputDts < mLastInputPts) {
                int bufferIndex;
                try {
                    bufferIndex = mMediaCodec.dequeueOutputBuffer(outputBufferInfo, 10 * 1000);
                } catch (Exception e) {
                    Log.e(TAG, "dequeueOutputBuffer" + e);
                    return;
                }

                if (bufferIndex >= 0) {
                    ByteBuffer decodeBuffer = mMediaCodec.getOutputBuffer(bufferIndex);
                    if (mListener != null) {
                        KFBufferFrame bufferFrame = new KFBufferFrame(decodeBuffer,outputBufferInfo);
                        mListener.dataOnAvailable(bufferFrame);
                    }
                    mMediaCodec.releaseOutputBuffer(bufferIndex,true);
                } else {
                    if (bufferIndex == MediaCodec.INFO_OUTPUT_FORMAT_CHANGED) {
                        mOutMediaFormat = mMediaCodec.getOutputFormat();
                    }
                    break;
                }
            }
        });

        return KFMediaCodeProcessSuccess;
    }

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    public void flush() {
        ///< Codec 清空缓冲区，一般用于Seek、结束时时使用。
        mCodecHandler.post(()-> {
            if (mMediaCodec == null) {
                return;
            }

            try {
                mMediaCodec.flush();
            } catch (Exception e) {
                Log.e(TAG, "flush" + e);
            }

            mListLock.lock();
            mList.clear();
            mListCacheSize = 0;
            mListLock.unlock();
        });
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    private boolean _appendFrame(KFBufferFrame frame) {
        ///< 将输入数据添加至缓冲区。
        mListLock.lock();
        int cacheSize = mListCacheSize;
        mListLock.unlock();
        if(cacheSize >= KFByteBufferCodecInputBufferMaxCache){
            return false;
        }

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
        mListCacheSize += packet.bufferInfo.size;
        mListLock.unlock();

        return true;
    }

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    private boolean _setupCodec() {
        ///< 初始化 Codec 模块，支持编码、解码，根据不同 MediaFormat 创建不同 Codec。
        try {
            String mimetype = mInputMediaFormat.getString(MediaFormat.KEY_MIME);
            if (mIsEncoder) {
                mMediaCodec = MediaCodec.createEncoderByType(mimetype);
            } else {
                mMediaCodec = MediaCodec.createDecoderByType(mimetype);
            }
        } catch (Exception e) {
            Log.e(TAG, "createCodecByType" + e + mIsEncoder);
            _callBackError(KFByteBufferCodecErrorCreate,e.getMessage());
            return false;
        }

        try {
            mMediaCodec.configure(mInputMediaFormat, null, null, mIsEncoder ? MediaCodec.CONFIGURE_FLAG_ENCODE : 0);
        } catch (Exception e) {
            Log.e(TAG, "configure" + e);
            _callBackError(KFByteBufferCodecErrorConfigure,e.getMessage());
            return false;
        }

        try {
            mMediaCodec.start();
            mInputBuffers = mMediaCodec.getInputBuffers();
        } catch (Exception e) {
            Log.e(TAG, "start" +  e );
            _callBackError(KFByteBufferCodecErrorStart,e.getMessage());
            return false;
        }

        return true;
    }

    private void _callBackError(int error, String errorMsg) {
        if (mListener != null) {
            mMainHandler.post(()->{
                mListener.onError(error,TAG + errorMsg);
            });
        }
    }
}

```

上面是 `KFByteBufferCodec` 的实现，从代码上可以看到主要有这几个部分：

* 1）创建与开启编码实例，`_setupCodec`，调用 `setup:` 时才会创建编码实例。
	- `mIsEncoder` 为 true 代表使用编码功能，创建编码功能使用 `createEncoderByType`，创建解码使用 `createDecoderByType`，`configure` 配置 Codec 编码使用 `MediaCodec.CONFIGURE_FLAG_ENCODE`，解码则填 0 即可。
	- `start` 在 `_setupCodec` 中执行，开启音频编码。
* 2）停止与清理编码实例，`release` 。
	- `stop` 在 `release` 中执行，关闭音频编码。
* 3）刷新编码缓冲区，`flush`，通常编码结束时将缓冲区数据刷新出来。
* 4）处理音频编码数据，`processFrame`，将编码前数据放入缓冲区，编码后数据抛给外层。
	- 输入缓冲区队列为 `mList`，需要注意缓冲区有上限，一旦超过最大值则返回 `KFMediaCodeProcessAgainLater`，防止因内存问题导致 OOM。
	- 编码线程异步处理数据，从 `mList` 取出数据塞入尽量多的数据给编码器，这样跳出循环条件为塞入编码器失败或者 `mList` 为空。拉取数据是借助标记 `mList` 最后一帧时间戳 `mLastInputPts`，跳出循环条件为输出数据等于此时间戳或拉取数据失败。MediaCodec 采用异步方式处理数据，并且使用了一组输入输出缓存 `mInputBuffers`。通过请求一个空的输入缓存 `dequeueInputBuffer`，向其中填充满数据并将它传递给编解码器处理 `queueInputBuffer`。编解码器处理完这些数据并将处理结果输出至一个空的输出缓存中 `dequeueOutputBuffer`。使用完输出缓存的数据之后 `getOutputBuffer`，将其释放回编解码器 `releaseOutputBuffer`。具体流程如下图所示：
	

![MediaCodec](assets/resource/av-demo/av-demo-android-buffer-codec.png)
_MediaCodec_

我们又定义了类 `KFAudioByteBufferEncoder`，继承自 `KFByteBufferCodec`，重写了 `processFrame` `release` `flush` 三个方法。

`KFAudioByteBufferEncoder.java`

```java
public class KFAudioByteBufferEncoder extends KFByteBufferCodec {
    private int mChannel = 0; ///< 音频声道数
    private int mSampleRate = 0; ///< 音频采样率
    private long mCurrentTimestamp = -1; ///< 标记当前时间戳 (因为数据重新分割，所以时间戳需要手动计算)
    private byte[] mByteArray = new byte[500 * 1024]; ///< 输入音频数据数组
    private int mByteArraySize = 0; ///< 输入音频数据 Size

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    @Override
    public int processFrame(KFFrame inputFrame) {
        ///< 获取音频声道数与采样率。
        if (mChannel == 0) {
            MediaFormat inputMediaFormat = getInputMediaFormat();
            if (inputMediaFormat != null) {
                mChannel = inputMediaFormat.getInteger(MediaFormat.KEY_CHANNEL_COUNT);
                mSampleRate = inputMediaFormat.getInteger(MediaFormat.KEY_SAMPLE_RATE);
            }
        }

        if (mChannel == 0  || mSampleRate == 0 || inputFrame == null) {
            return KFMediaCodeProcessParams;
        }

        KFBufferFrame bufferFrame = (KFBufferFrame)inputFrame;
        if (bufferFrame.bufferInfo == null || bufferFrame.bufferInfo.size == 0) {
            return KFMediaCodeProcessParams;
        }

        ///< 控制音频输入给编码器单次字节数 2048 字节。
        int sendSize = 2048;
        ///< 外层输入如果为 2048 则直接跳过执行。
        if (mByteArraySize == 0 && sendSize == bufferFrame.bufferInfo.size) {
            return super.processFrame(inputFrame);
        } else {
            long currentTimestamp = 0;
            if (mCurrentTimestamp == -1) {
                mCurrentTimestamp = bufferFrame.bufferInfo.presentationTimeUs;
            }

            ///< 将缓存中数据执行送入编码器操作。
            int sendCacheStatus = sendBufferEncoder(sendSize);
            if (sendCacheStatus < 0) {
                return sendCacheStatus;
            }

            ///< 将输入数据送入缓冲区重复执行此操作。
            byte[] inputBytes = new byte[bufferFrame.bufferInfo.size];
            bufferFrame.buffer.get(inputBytes);

            System.arraycopy(inputBytes,0,mByteArray,mByteArraySize,bufferFrame.bufferInfo.size);
            mByteArraySize += bufferFrame.bufferInfo.size;

            return sendBufferEncoder(sendSize);
        }
    }

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    @Override
    public void release() {
        mCurrentTimestamp = -1;
        mByteArraySize = 0;
        super.release();
    }

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
    @Override
    public void flush() {
        mCurrentTimestamp = -1;
        mByteArraySize = 0;
        super.flush();
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    private int sendBufferEncoder(int sendSize) {
        ///< 将当前 Buffer 中数据按每次 2048 送给编码器。
        while (mByteArraySize >= sendSize) {
            MediaCodec.BufferInfo newBufferInfo = new MediaCodec.BufferInfo();
            newBufferInfo.size = sendSize;
            newBufferInfo.presentationTimeUs = mCurrentTimestamp;

            ByteBuffer newBuffer = ByteBuffer.allocateDirect(sendSize);
            newBuffer.put(mByteArray,0,sendSize).position(0);

            KFBufferFrame newFrame = new KFBufferFrame();
            newFrame.buffer = newBuffer;
            newFrame.bufferInfo = newBufferInfo;
            int status = super.processFrame(newFrame);
            if (status < 0) {
                return status;
            } else {
                mByteArraySize -= sendSize;
                if (mByteArraySize > 0) {
                    System.arraycopy(mByteArray, sendSize, mByteArray, 0, mByteArraySize);
                }
            }
            mCurrentTimestamp += sendSize * 1000000 / (16 / 8 * mSampleRate * mChannel);
        }
        return KFMediaCodeProcessSuccess;
    }
}
```

上面是 `KFAudioByteBufferEncoder` 的实现，主要就干了一件事：拆分合适大小（2048 字节）的数据送给编码器。因为 AAC 数据编码每 packet 大小为 `1024 * 2（位深 16 Bit）`。


## 3、采集音频数据进行 AAC 编码和存储

我们在一个 `MainActivity` 中来实现音频采集及编码逻辑，并将编码后的数据加上 [ADTS ](https://wiki.multimedia.cx/index.php?title=ADTS "ADTS 格式") 头信息存储为 AAC 数据。

关于 ADTS，在[《音频编码：PCM 和 AAC 编码》](https://mp.weixin.qq.com/s/tVUgH17HZe6PWj09hYzkbg)中也有介绍，可以去看看了解一下。

`MainActivity.java`

```java
public class MainActivity extends AppCompatActivity {
    private FileOutputStream mStream = null;
    private KFAudioCapture mAudioCapture = null; ///< 音频采集模块
    private KFAudioCaptureConfig mAudioCaptureConfig = null; ///< 音频采集配置
    private KFMediaCodecInterface mEncoder = null; ///< 音频编码
    private MediaFormat mAudioEncoderFormat = null; ///< 音频编码格式描述
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

        mAudioCaptureConfig = new KFAudioCaptureConfig();
        mAudioCapture = new KFAudioCapture(mAudioCaptureConfig,mAudioCaptureListener);
        mAudioCapture.startRunning();

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
                if (mEncoder == null) {
                    mEncoder = new KFAudioByteBufferEncoder();
                    MediaFormat mediaFormat = KFAVTools.createAudioFormat(mAudioCaptureConfig.sampleRate,mAudioCaptureConfig.channel,96*1000);
                    mEncoder.setup(true,mediaFormat,mAudioEncoderListener,null);
                    ((Button)view).setText("停止");
                } else {
                    mEncoder.release();
                    mEncoder = null;
                    ((Button)view).setText("开始");
                }
            }
        });
        addContentView(startButton, startParams);
    }

    private KFAudioCaptureListener mAudioCaptureListener = new KFAudioCaptureListener() {
        @Override
        public void onError(int error, String errorMsg) {
            Log.e("KFAudioCapture","errorCode" + error + "msg"+errorMsg);
        }

        @Override
        public void onFrameAvailable(KFFrame frame) {
            if (mEncoder != null) {
                mEncoder.processFrame(frame);
            }
        }
    };

    private KFMediaCodecListener mAudioEncoderListener = new KFMediaCodecListener() {
        @Override
        public void onError(int error, String errorMsg) {
            Log.i("KFMediaCodecListener","error" + error + "msg" + errorMsg);
        }

        @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
        @Override
        public void dataOnAvailable(KFFrame frame) {
            ///< 音频回调数据
            if (mAudioEncoderFormat == null && mEncoder != null) {
                mAudioEncoderFormat = mEncoder.getOutputMediaFormat();
            }
            KFBufferFrame bufferFrame = (KFBufferFrame)frame;
            try {
                ///< 添加ADTS数据
                ByteBuffer adtsBuffer = KFAVTools.getADTS(bufferFrame.bufferInfo.size,mAudioEncoderFormat.getInteger(MediaFormat.KEY_PROFILE),mAudioEncoderFormat.getInteger(MediaFormat.KEY_SAMPLE_RATE),mAudioEncoderFormat.getInteger(MediaFormat.KEY_CHANNEL_COUNT));
                byte[] adtsBytes = new byte[adtsBuffer.capacity()];
                adtsBuffer.get(adtsBytes);
                mStream.write(adtsBytes);

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

上面是 `MainActivity ` 的实现，其中主要包含这几个部分：

- 1）在采集音频前需要设置 `Manifest.permission.RECORD_AUDIO` 权限。
- 2）通过启动和停止音频采集来驱动整个采集和编码流程。
- 3）在采集模块 `KFAudioCapture` 的数据回调中将数据交给编码模块 `KFAudioByteBufferEncoder` 进行编码。
	- 在 `KFAudioCaptureListener` 的 `onFrameAvailable` 回调中实现。
- 4）创建模块 `KFAudioByteBufferEncoder ` 的 `setup` 中 MediaFormat。
	- 对应的实现在 `KFAVTools` 类的工具方法 `static MediaFormat createVideoFormat(boolean isHEVC, Size size,int format,int bitrate,int fps,int gopDuration,int profile,int profileLevel)` 中实现。
- 5）在编码模块 `KFAudioByteBufferEncoder ` 的数据回调中获取编码后的 AAC 裸流数据，并在每个 AAC packet 前写入 ADTS 头数据，存储到文件中。
	- 在 `KFMediaCodecListener` 的 `dataOnAvailable` 回调中实现。
	- 其中生成一个 AAC packet 对应的 ADTS 头数据在 `KFAVTools` 类的工具方法 `static ByteBuffer getADTS(int size, int profile, int sampleRate, int channel)` 中实现。

`KFAVTools.java`

```java
public class KFAVTools {

    // 按音频参数生产 AAC packet 对应的 ADTS 头数据。
    // 当编码器编码的是 AAC 裸流数据时，需要在每个 AAC packet 前添加一个 ADTS 头用于解码器解码音频流。
    // 参考文档：
    // ADTS 格式参考：http://wiki.multimedia.cx/index.php?title=ADTS
    // MPEG-4 Audio 格式参考：http://wiki.multimedia.cx/index.php?title=MPEG-4_Audio#Channel_Configurations
    public static ByteBuffer getADTS(int size, int profile, int sampleRate, int channel) {
        int sampleRateIndex = getSampleRateIndex(sampleRate);// 取得采样率对应的 index。
        int fullSize = 7 + size;
        // ADTS 头固定 7 字节。
        // 填充 ADTS 数据。
        ByteBuffer adtsBuffer = ByteBuffer.allocateDirect(7);
        adtsBuffer.order(ByteOrder.nativeOrder());
        adtsBuffer.put((byte)0xFF); // 11111111 = syncword
        adtsBuffer.put((byte)0xF1);
        adtsBuffer.put((byte)(((profile - 1) << 6) + (sampleRateIndex << 2) + (channel >> 2)));
        adtsBuffer.put((byte)(((channel & 3) << 6) + (fullSize >> 11)));
        adtsBuffer.put((byte)((fullSize & 0x7FF) >> 3));
        adtsBuffer.put((byte)(((fullSize & 7) << 5) + 0x1F));
        adtsBuffer.put((byte)0xFC);
        adtsBuffer.position(0);

        return adtsBuffer;
    }

    private static int getSampleRateIndex(int sampleRate) {
        int sampleRateIndex = 0;
        switch (sampleRate) {
            case 96000:
                sampleRateIndex = 0;
                break;
            case 88200:
                sampleRateIndex = 1;
                break;
            case 64000:
                sampleRateIndex = 2;
                break;
            case 48000:
                sampleRateIndex = 3;
                break;
            case 44100:
                sampleRateIndex = 4;
                break;
            case 32000:
                sampleRateIndex = 5;
                break;
            case 24000:
                sampleRateIndex = 6;
                break;
            case 22050:
                sampleRateIndex = 7;
                break;
            case 16000:
                sampleRateIndex = 8;
                break;
            case 12000:
                sampleRateIndex = 9;
                break;
            case 11025:
                sampleRateIndex = 10;
                break;
            case 8000:
                sampleRateIndex = 11;
                break;
            case 7350:
                sampleRateIndex = 12;
                break;
            default:
                sampleRateIndex = 15;
        }
        return sampleRateIndex;
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    public static MediaFormat createVideoFormat(boolean isHEVC, Size size,int format,int bitrate,int fps,int gopDuration,int profile,int profileLevel) {
        String mimeType = isHEVC ? "video/hevc" : "video/avc";
        MediaFormat mediaFormat = MediaFormat.createVideoFormat(mimeType, size.getWidth(), size.getHeight());

        mediaFormat.setInteger(MediaFormat.KEY_COLOR_FORMAT, format); //MediaCodecInfo.CodecCapabilities.COLOR_FormatSurface
        mediaFormat.setInteger(MediaFormat.KEY_BIT_RATE, bitrate);
        mediaFormat.setInteger(MediaFormat.KEY_FRAME_RATE, fps);
        mediaFormat.setInteger(MediaFormat.KEY_I_FRAME_INTERVAL, gopDuration);
        mediaFormat.setInteger(MediaFormat.KEY_PROFILE, profile);
        mediaFormat.setInteger(MediaFormat.KEY_LEVEL, profileLevel);

        return mediaFormat;
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    public static MediaFormat createAudioFormat(int sampleRate, int channel, int bitrate) {
        String mimeType = MediaFormat.MIMETYPE_AUDIO_AAC;
        MediaFormat mediaFormat = MediaFormat.createAudioFormat(mimeType, sampleRate, channel);

        mediaFormat.setInteger(MediaFormat.KEY_BIT_RATE, bitrate);
        mediaFormat.setInteger(MediaFormat.KEY_CHANNEL_COUNT, channel);
        mediaFormat.setInteger(MediaFormat.KEY_SAMPLE_RATE, sampleRate);
        mediaFormat.setInteger(MediaFormat.KEY_AAC_PROFILE, MediaCodecInfo.CodecProfileLevel.AACObjectLC);

        return mediaFormat;
    }
}
```

## 3、用工具播放 AAC 文件

完成音频采集和编码后，可以将 `sdcard` 文件夹下面的 `test.aac` 文件拷贝到电脑上，使用 `ffplay` 播放来验证一下音频采集是效果是否符合预期：

```c
$ ffplay -i test.aac
```

这里在播放 AAC 文件时不必像播放 PCM 文件那样设置音频参数，这正是因为我们已经将对应的参数信息编码到 ADTS 头部数据中去了，播放解码时可以从中解析出这些信息从而正确的解码 AAC。

关于播放 AAC 文件的工具，可以参考[《FFmpeg 工具》第 2 节 ffplay 命令行工具](https://mp.weixin.qq.com/s/Rl7fxOP-YH37mQEvGxhfUA)和[《可视化音视频分析工具》第 1.1 节 Adobe Audition](https://mp.weixin.qq.com/s/jCYih3qgEIUctuWxn0aTGQ)。

