---
title: Android AVDemo（6）：音频渲染，代码开源并提供解析
description: 介绍 Android 音频渲染的流程和原理，并提供 Demo 源码和解析。
author: Keyframe
date: 2025-02-23 10:48:08 +0800
categories: [音视频源码示例]
tags: [音视频源码示例, 音视频, Android, 音频, 音频渲染]
pin: false
math: true
mermaid: true
---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频**、**AI** 领域的最新技术和产品信息：
>
>![微信公众号](assets/img/keyframe-mp.jpg)
_微信扫码关注我们_
{: .prompt-tip }

iOS/Android 客户端开发同学如果想要开始学习音视频开发，最丝滑的方式是对[音视频基础概念知识](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2140155659944787969#wechat_redirect)有一定了解后，再借助 iOS/Android 平台的音视频能力上手去实践音视频的`采集 → 编码 → 封装 → 解封装 → 解码 → 渲染`过程，并借助[音视频工具](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2216997905264082945#wechat_redirect)来分析和理解对应的音视频数据。

在[音视频工程示例](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2273301900659851268#wechat_redirect)这个栏目，我们将通过拆解`采集 → 编码 → 封装 → 解封装 → 解码 → 渲染`流程并实现 Demo 来向大家介绍如何在 iOS/Android 平台上手音视频开发。

这里是 Android 第六篇：**Android 音频渲染 Demo**。这个 Demo 里包含以下内容：


- 1）实现一个音频解封装模块；
- 2）实现一个音频解码模块；
- 3）实现一个音频渲染模块；
- 4）实现对 MP4 文件中音频部分的解封装和解码逻辑，并将解封装、解码后的数据送给渲染模块播放；
- 5）详尽的代码注释，帮你理解代码逻辑和原理。

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

同样的，解码模块 `KFByteBufferCodec` 的实现与 [《Android 音频解码 Demo》](https://mp.weixin.qq.com/s?__biz=MjM5MTkxOTQyMQ==&mid=2257485614&idx=1&sn=636683b05eacc4f4728fb2849a445ded&chksm=a5d4e37c92a36a6a8a4d3d1991cebc5fde3775086b4da22d78924d7f965e78fccbf6605c0be5&token=991245230&lang=zh_CN#rd) 中一样，这里就不再重复介绍了，其接口如下：

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




## 3、音频渲染模块

接下来，我们来实现一个音频渲染模块 `KFAudioRender`，在这里输入解码后的数据进行渲染播放。

`KFAudioRenderListener.java`

```java
public interface KFAudioRenderListener {
    ///< 出错回调。
    void onError(int error,String errorMsg);
    ///< 获取PCM数据。
    byte[] audioPCMData(int size);
}
```
上面是 `KFAudioRenderListener` 接口的设计，主要是有音频渲染`数据输入回调`和`错误回调`的接口。

这里重点需要看一下音频渲染`数据输入回调`接口，系统的音频渲染单元每次会主动通过回调的方式要数据，我们这里封装的 `KFAudioRender` 则是用`数据输入回调`接口来从外部获取一组待渲染的音频数据送给系统的音频渲染单元。


`KFAudioRender.java`

```java
public class KFAudioRender {
    private static final String TAG = "KFAudioRender";
    public static final int KFAudioRenderErrorCreate = -2700;
    public static final int KFAudioRenderErrorPlay = -2701;
    public static final int KFAudioRenderErrorStop = -2702;
    public static final int KFAudioRenderErrorPause = -2703;

    private static final int KFAudioRenderMaxCacheSize = 500*1024; ///< 音频PCM缓存最大值。
    private KFAudioRenderListener mListener = null; ///< 回调。
    private Handler mMainHandler = new Handler(Looper.getMainLooper()); ///< 主线程。
    private HandlerThread mThread = null; ///< 音频管控线程。
    private Handler mHandler = null;
    private HandlerThread mRenderThread = null; ///< 音频渲染线程。
    private Handler mRenderHandler = null;
    private AudioTrack mAudioTrack = null; ///< 音频播放实例。
    private int mMinBufferSize = 0;
    private byte mCache[] = new byte[KFAudioRenderMaxCacheSize]; ///< 音频PCM缓存。
    private int mCacheSize = 0;

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    public KFAudioRender(KFAudioRenderListener listener, int sampleRate, int channel) {
        mListener = listener;
        ///< 创建音频管控线程。
        mThread = new HandlerThread("KFAudioRenderThread");
        mThread.start();
        mHandler = new Handler((mThread.getLooper()));
        ///< 创建音频渲染线程。
        mRenderThread = new HandlerThread("KFAudioGetDataThread");
        mRenderThread.start();
        mRenderHandler = new Handler((mRenderThread.getLooper()));

        mHandler.post(()->{
            ///< 初始化音频播放实例。
           _setupAudioTrack(sampleRate,channel);
        });
    }

    public void release() {
        mHandler.post(()-> {
            ///< 停止与释放音频播放实例。
            if (mAudioTrack != null) {
                try {
                    mAudioTrack.stop();
                    mAudioTrack.release();
                } catch (Exception e) {
                    Log.e(TAG, "release: " + e.toString());
                }
                mAudioTrack = null;
            }

            mThread.quit();
            mRenderThread.quit();
        });
    }

    public void play() {
        mHandler.post(()-> {
            ///< 音频实例播放。
            try {
                mAudioTrack.play();
            } catch (Exception e){
                _callBackError(KFAudioRenderErrorPlay,e.getMessage());
                return;
            }

            mRenderHandler.post(()->{
                ///< 循环写入PCM数据，写入系统缓冲区，当读取到最大值或者状态机不等于STATE_INITIALIZED 则退出循环。
                while (mAudioTrack.getState() == STATE_INITIALIZED){
                    if (mListener != null && mCacheSize < KFAudioRenderMaxCacheSize) {
                        byte[] bytes = mListener.audioPCMData(mMinBufferSize);
                        if (bytes != null && bytes.length > 0) {
                            System.arraycopy(bytes,0,mCache,mCacheSize,bytes.length);
                            mCacheSize += bytes.length;
                            if (mCacheSize >= mMinBufferSize) {
                                int writeSize = mAudioTrack.write(mCache,0,mMinBufferSize);
                                if (writeSize > 0) {
                                    mCacheSize -= writeSize;
                                    System.arraycopy(mCache,writeSize,mCache,0,mCacheSize);
                                }
                            }
                        } else {
                            break;
                        }
                    }
                }
            });
        });
    }

    public void stop() {
        ///< 停止音频播放。
        mHandler.post(()-> {
            try {
                mAudioTrack.stop();
            } catch (Exception e){
                _callBackError(KFAudioRenderErrorStop,e.getMessage());
            }
            mCacheSize = 0;
        });
    }

    public void pause() {
        ///< 暂停音频播放。
        mHandler.post(()-> {
            try {
                mAudioTrack.pause();
            } catch (Exception e){
                _callBackError(KFAudioRenderErrorPause,e.getMessage());
            }
        });
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    public void  _setupAudioTrack(int sampleRate, int channel) {
        ///< 根据采样率、声道获取每次音频播放塞入数据大小，根据采样率、声道、数据大小创建音频播放实例。
        if (mAudioTrack == null) {
            try {
                mMinBufferSize = AudioRecord.getMinBufferSize(sampleRate, channel, AudioFormat.ENCODING_PCM_16BIT);
                mAudioTrack = new AudioTrack(AudioManager.STREAM_MUSIC,sampleRate,channel == 2 ? AudioFormat.CHANNEL_OUT_STEREO : AudioFormat.CHANNEL_OUT_MONO,AudioFormat.ENCODING_PCM_16BIT,mMinBufferSize,AudioTrack.MODE_STREAM);
            } catch (Exception e){
                _callBackError(KFAudioRenderErrorCreate,e.getMessage());
            }
        }
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

上面是 `KFAudioRender` 的实现，从代码上可以看到主要有这几个部分：

- 1）创建音频渲染实例。
    - 在 `_setupAudioTrack` 方法中实现，根据采样率、声道、单次输入数据大小 等几个参数生成。 
- 2）处理音频渲染实例的数据回调，并在回调中通过 `KFAudioRender` 的对外数据输入回调接口向更外层要待渲染的数据。
    - 通过 `audioPCMData` 回调接口向更外层要数据。
- 3）实现开始渲染和停止渲染逻辑。
    - 分别在 `play` 和 `stop` 方法中实现。注意，这里是开始和停止操作都是放在串行队列中通过 `mHandler.post` 异步处理的，这里主要是为了防止主线程卡顿。
    - 开启播放后会循环向外层获取 PCM 数据，通过 `write` 方法写入 `mAudioTrack`。
- 4）清理音频渲染实例。
    - 在 `release` 方法中实现。


更具体细节见上述代码及其注释。







## 4、解封装和解码 MP4 文件中的音频部分并渲染播放

我们在一个 `MainActivity` 中来实现从 MP4 文件中解封装和解码音频数据进行渲染播放。


`MainActivity.java`



```java
public class MainActivity extends AppCompatActivity {
    private KFDemuxer mDemuxer; ///< 音频解封装实例。
    private KFDemuxerConfig mDemuxerConfig; ///< 音频解决封装配置。
    private KFMediaCodecInterface mDecoder; ///< 音频解码实例。 
    private KFAudioRender mRender; ///< 音频渲染实例。
    private byte[] mPCMCache = new byte[10*1024*1024]; ///< PCM数据缓存。
    private int mPCMCacheSize = 0;
    private ReentrantLock mLock = new ReentrantLock(true);

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ///< 获取音频采集、本地存储权限。
        if (ActivityCompat.checkSelfPermission(this, Manifest.permission.RECORD_AUDIO) != PackageManager.PERMISSION_GRANTED || ActivityCompat.checkSelfPermission(this, Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED ||
                ActivityCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED ||
                ActivityCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions((Activity) this,
                    new String[] {Manifest.permission.CAMERA,Manifest.permission.RECORD_AUDIO, Manifest.permission.READ_EXTERNAL_STORAGE, Manifest.permission.WRITE_EXTERNAL_STORAGE},
                    1);
        }

        ///< 创建音频解封装配置。
        mDemuxerConfig = new KFDemuxerConfig();
        mDemuxerConfig.path = Environment.getExternalStorageDirectory().getPath() + "/test.aac";
        mDemuxerConfig.demuxerType = KFGLBase.KFMediaType.KFMediaAudio;

        ///< 创建音频解封装实例。
        mDemuxer = new KFDemuxer(mDemuxerConfig,mDemuxerListener);
        mDecoder = new KFByteBufferCodec();
        mDecoder.setup(false,mDemuxer.audioMediaFormat(),mDecoderListener,null);

        ///< 循环获取解封装数据塞入解码器。
        MediaCodec.BufferInfo bufferInfo = new MediaCodec.BufferInfo();
        ByteBuffer nextBuffer = mDemuxer.readAudioSampleData(bufferInfo);
        while (nextBuffer != null) {
            mDecoder.processFrame(new KFBufferFrame(nextBuffer,bufferInfo));
            nextBuffer = mDemuxer.readAudioSampleData(bufferInfo);
        }

        ///< 创建音频渲染实例。
        mRender = new KFAudioRender(mRenderListener,mDemuxer.samplerate(),mDemuxer.channel());
        mRender.play();
    }

    private KFDemuxerListener mDemuxerListener = new KFDemuxerListener() {
        @Override
        ///< 解封装出错。
        public void demuxerOnError(int error, String errorMsg) {
            Log.i("KFDemuxer","error" + error + "msg" + errorMsg);
        }
    };

    private KFMediaCodecListener mDecoderListener = new KFMediaCodecListener() {
        @Override
        ///< 解码出错。
        public void onError(int error, String errorMsg) {

        }

        @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
        @Override
        ///< 解码数据回调存储到本地 PCM 缓存，Demo 处理比较简单，没有考虑到渲染暂停解码不暂停等case，可能存在缓冲区溢出。
        public void dataOnAvailable(KFFrame frame) {
            KFBufferFrame bufferFrame = (KFBufferFrame)frame;
            if (bufferFrame.buffer != null && bufferFrame.bufferInfo.size > 0) {
                byte[] bytes = new byte[bufferFrame.bufferInfo.size];
                bufferFrame.buffer.get(bytes);
                mLock.lock();
                System.arraycopy(bytes,0,mPCMCache,mPCMCacheSize,bytes.length);
                mPCMCacheSize += bytes.length;
                mLock.unlock();
            }
        }
    };

    private KFAudioRenderListener mRenderListener = new KFAudioRenderListener() {
        @Override
        ///< 音频渲染出错。
        public void onError(int error, String errorMsg) {

        }

        @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
        @Override
        ///< 音频播放模块获取音频 PCM 数据。
        public byte[] audioPCMData(int size) {
            if (mPCMCacheSize >= size) {
                byte[] dst = new byte[size];
                mLock.lock();
                System.arraycopy(mPCMCache,0,dst,0,size);
                mPCMCacheSize -= size;
                System.arraycopy(mPCMCache,size,mPCMCache,0,mPCMCacheSize);
                mLock.unlock();
                return dst;
            }
            return null;
        }
    };
}
```


上面是 `MainActivity` 的实现，其中主要包含这几个部分：


- 1）在页面加载完成后就启动解封装和解码模块，并且循环读取音频数据传递给解码器。
	- 在 `onCreate` 中实现。
- 2）在解码模块 `KFByteBufferCodec` 的数据回调中获取解码后的 PCM 数据缓冲起来等待渲染。
	- 在 `KFMediaCodecListener` 的 `dataOnAvailable` 回调中实现。
- 3）在渲染模块 `KFAudioRender` 的输入数据回调中把缓冲区的数据交给系统音频渲染单元渲染。
	- 在 `KFAudioRenderListener` 的 `audioPCMData` 回调中实现。


更具体细节见上述代码及其注释。





