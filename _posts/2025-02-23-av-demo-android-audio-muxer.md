---
title: Android AVDemo（3）：音频封装，代码开源并提供解析
description: 介绍 Android 音频封装的流程和原理，并提供 Demo 源码和解析。
author: Keyframe
date: 2025-02-23 10:18:08 +0800
categories: [音视频源码示例]
tags: [音视频源码示例, 音视频, Android, 音频, 音频封装]
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

这里是 Android 第三篇：**Android 音频封装 Demo**。这个 Demo 里包含以下内容：

- 1）实现一个音频采集模块；
- 2）实现一个音频编码模块；
- 3）实现一个音频封装模块；
- 4）串联音频采集、编码、封装模块，将采集到的音频数据输入给 AAC 编码模块进行编码，再将编码后的数据输入给 M4A 封装模块封装和存储；
- 5）详尽的代码注释，帮你理解代码逻辑和原理。




>>如果你想获得全部源码和参与音视频技术讨论，可以通过下面二维码加入『关键帧的音视频开发圈』，当然也可以跳过直接看后续的内容。
>>
>>![长按识别二维码→加入我们](assets/img/keyframe-zsxq.png)
>>_长按识别二维码→加入我们_




## 1、音频采集模块

在这个 Demo 中，音频采集模块 `KFAudioCapture` 的实现与 [《Android 音频采集 Demo》](https://mp.weixin.qq.com/s/o0JqYnjY0aR3oGCEXk7vcQ) 中一样，这里就不再重复介绍了，其接口如下：


`KFAudioCapture.java`

```java
public class KFAudioCapture {
	public KFAudioCapture(KFAudioCaptureConfig config,KFAudioCaptureListener listener);
	public void startRunning(); // 开始采集音频数据。
	public void stopRunning(); // 停止采集音频数据。
	public void release(); // 释放音频采集。
}
```

## 2、音频编码模块



同样的，音频编码模块 `KFAudioByteBufferEncoder` 的实现与[《Android 音频编码 Demo》](https://mp.weixin.qq.com/s/yI6XMPbLfNvaJNZEmQkNqQ)中一样，这里就不再重复介绍了，其接口如下：


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




## 3、音频封装模块


接下来，我们来实现一个音频封装模块，在这里输入编码后的数据，输出封装后的文件。

这次我们要封装的格式是 M4A，属于 MPEG-4 标准，通常普通的 MPEG-4 文件扩展名是 `.mp4`，只包含音频的 MPEG-4 文件扩展名用 `.m4a`。所以，其实我们这里实现的是一个 MP4 封装模块，支持将音频编码数据封装成 M4A，也支持将音视频数据封装成 MP4。关于 MP4 格式，可以看一看[《MP4 格式》](https://mp.weixin.qq.com/s/GAYkhrakhQlkpnWwKrFeyg)这篇文章了解一下。


由于 MP4 封装涉及到一些参数设置，所以我们先实现一个 `KFMuxerConfig` 类用于定义 MP4 封装的参数的配置。这里包括了：封装文件输出地址、封装文件类型这几个参数。


`KFMuxerConfig.java`


```java
public class KFMuxerConfig {
    ///< 输出路径。
    public String outputPath = null;
    ///< 封装仅音频、仅视频、音视频。
    public KFMediaBase.KFMediaType muxerType = KFMediaBase.KFMediaType.KFMediaAV;

    public KFMuxerConfig(String path) {
        outputPath = path;
    }
}
```

其中用到的 `KFMediaType` 是定义在 `KFMediaBase` 中的一个枚举：

`KFMediaBase.java`

```java
public class KFMediaBase {
    public enum KFMediaType {
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


接下来，我们来实现 `KFMP4Muxer` 模块。


`KFMP4Muxer.java`

```java
public class KFMP4Muxer {
    public static final int KFMuxerErrorCreate = -2200;
    public static final int KFMuxerErrorAudioAddTrack = -2201;
    public static final int KFMuxerErrorVideoAddTrack = -2202;

    private static final String TAG = "KFMuxer";
    private KFMuxerConfig mConfig = null; ///< 封装配置
    private KFMuxerListener mListener = null; ///< 回调
    private MediaMuxer mMediaMuxer = null; ///< 封装实例
    private int mVideoTrackIndex = -1; ///< 视频 track 轨道下标
    private MediaFormat mVideoFormat = null; ///< 视频输入视频格式描述
    private List<KFBufferFrame> mVideoList = new ArrayList<>(); ///< 视频输入缓存
    private int mAudioTrackIndex = -1; ///< 音频 track 轨道下标
    private MediaFormat mAudioFormat = null; ///< 音频输入视频格式描述
    private List<KFBufferFrame> mAudioList = new ArrayList<>(); ///< 音频输入缓存
    private boolean mIsStart = false;
    private Handler mMainHandler = new Handler(Looper.getMainLooper()); ///< 主线程

    public KFMP4Muxer(KFMuxerConfig config, KFMuxerListener listener) {
        mConfig = config;
        mListener = listener;
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    public void start() {
        _setupMuxer();
    }

    public void stop() {
        _stop();
    }

    public void setVideoMediaFormat(MediaFormat mediaFormat) {
        mVideoFormat = mediaFormat;
    }

    public void setAudioMediaFormat(MediaFormat mediaFormat) {
        mAudioFormat = mediaFormat;
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    ///< 写入音视频数据（编码后数据）。
    public void writeSampleData(boolean isVideo, ByteBuffer buffer, MediaCodec.BufferInfo bufferInfo) {
        if ((bufferInfo.flags & BUFFER_FLAG_CODEC_CONFIG) != 0) {
            return;
        }

        if (buffer ==null || bufferInfo == null || mMediaMuxer == null || bufferInfo.size == 0) {
            return;
        }

        ///< 校验视频数据是否进入。
        if (!_hasAudioTrack() && !isVideo) {
            return;
        }

        ///< 校验视频数据是否进入。
        if (!_hasVideoTrack() && isVideo) {
            return;
        }

        ///< 数据转换结构体 KFBufferFrame。
        KFBufferFrame packet = new KFBufferFrame();
        ByteBuffer newBuffer = ByteBuffer.allocateDirect(bufferInfo.size);
        newBuffer.put(buffer).position(0);

        MediaCodec.BufferInfo newInfo = new MediaCodec.BufferInfo();
        newInfo.size = bufferInfo.size;
        newInfo.flags = bufferInfo.flags;
        newInfo.presentationTimeUs = bufferInfo.presentationTimeUs;

        packet.buffer = newBuffer;
        packet.bufferInfo = newInfo;
        if (isVideo) {
            ///< 初始化视频 Track。
            if (mVideoFormat != null && mVideoTrackIndex == -1) {
                _setupVideoTrack();
            }
            mVideoList.add(packet);
        } else {
            ///< 初始化音频Track
            if (mAudioFormat != null && mAudioTrackIndex == -1) {
                _setupAudioTrack();
            }
            mAudioList.add(packet);
        }

        ///< 校验音视频 Track 是否都初始化好。
        if ((_hasAudioTrack() && _hasVideoTrack() && mAudioTrackIndex >=0 && mVideoTrackIndex >= 0) ||
                (_hasAudioTrack() && !_hasVideoTrack() && mAudioTrackIndex >= 0) ||
                (!_hasAudioTrack() && _hasVideoTrack() && mVideoTrackIndex >= 0)) {
            if (!mIsStart) {
                _start();
                mIsStart = true;
            }

            ///< 音视频交错，目的音视频时间戳尽量不跳跃。
            if(mIsStart){
                _avInterleavedBuffers();
            }
        }
    }

    public void release() {
        _stop();
    }

    private void _start() {
        ///< 开启封装。
        try {
            if (mMediaMuxer != null) {
                mMediaMuxer.start();
            }
        } catch (Exception e) {
            Log.e(TAG, "start" + e);
        }
    }

    private void _stop() {
        ///< 关闭封装
        try {
            if (mMediaMuxer != null) {
                ///< 兜底一路没进来的 case，如果外层配置音视频一起封装但最终只进来一路也会处理。
                if (!mIsStart && (mVideoTrackIndex != 0 || mAudioTrackIndex != 0) && (mVideoList.size() > 0 || mAudioList.size() > 0)) {
                    mMediaMuxer.start();
                    mIsStart = true;
                }

                ///< 将缓冲中数据推入封装器。
                if (mIsStart) {
                    _appendAudioBuffers();
                    _appendVideoBuffers();
                    mMediaMuxer.stop();
                }

                ///< 释放封装器实例。
                mMediaMuxer.release();
                mMediaMuxer = null;
            }
        } catch (Exception e) {
            Log.e(TAG, "stop release" + e);
        }
        ///< 清空相关缓存与标记位。
        mVideoTrackIndex = -1;
        mAudioTrackIndex = -1;
        mIsStart = false;
        mVideoList.clear();
        mAudioList.clear();
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    private boolean _hasAudioTrack() {
        return (mConfig.muxerType.value() & KFMediaBase.KFMediaType.KFMediaAudio.value()) != 0;
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    private boolean _hasVideoTrack() {
        return (mConfig.muxerType.value() & KFMediaBase.KFMediaType.KFMediaVideo.value()) != 0;
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    private void _setupMuxer() {
        ///< 初始化封装器。
        if(mMediaMuxer == null){
            try {
                mMediaMuxer = new MediaMuxer(mConfig.outputPath, MediaMuxer.OutputFormat.MUXER_OUTPUT_MPEG_4);
            } catch (IOException e) {
                Log.e(TAG, "new MediaMuxer" + e);
                _callBackError(KFMuxerErrorCreate,e.getMessage());
                return;
            }
        }
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    private void _setupVideoTrack() {
        ///< 根据外层输入格式描述初始化视频 Track。
        if (mVideoFormat != null) {
            ///< 添加视频 Track。
            try {
                mVideoTrackIndex = mMediaMuxer.addTrack(mVideoFormat);
            } catch (Exception e) {
                Log.e(TAG, "addTrack" + e);
                _callBackError(KFMuxerErrorVideoAddTrack,e.getMessage());
            }
        }
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    private void _setupAudioTrack() {
        ///< 根据外层输入格式描述初始化音频 Track。
        if(mAudioFormat != null){
            ///< 添加音频 Track。
            try {
                mAudioTrackIndex = mMediaMuxer.addTrack(mAudioFormat);
            } catch (Exception e) {
                Log.e(TAG, "addTrack" + e);
                _callBackError(KFMuxerErrorAudioAddTrack,e.getMessage());
            }
        }
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    private void _avInterleavedBuffers() {
        ///< 音视频交错，通过对比时间戳大小交错进入。
        if (_hasVideoTrack() && _hasAudioTrack()) {
            while (mAudioList.size() > 0 && mVideoList.size() > 0) {
                KFBufferFrame audioPacket = mAudioList.get(0);
                KFBufferFrame videoPacket = mVideoList.get(0);

                if (audioPacket.bufferInfo.presentationTimeUs >= videoPacket.bufferInfo.presentationTimeUs) {
                    mMediaMuxer.writeSampleData(mVideoTrackIndex,videoPacket.buffer,videoPacket.bufferInfo);
                    mVideoList.remove(0);
                } else {
                    mMediaMuxer.writeSampleData(mAudioTrackIndex,audioPacket.buffer,audioPacket.bufferInfo);
                    mAudioList.remove(0);
                }
            }
        } else if (_hasVideoTrack()) {
            _appendVideoBuffers();
        } else if (_hasAudioTrack()) {
            _appendAudioBuffers();
        }
    }

    private void _appendAudioBuffers() {
        ///< 音频队列缓冲区推到封装器。
        while (mAudioList.size() > 0) {
            KFBufferFrame packet = mAudioList.get(0);
            mMediaMuxer.writeSampleData(mAudioTrackIndex,packet.buffer,packet.bufferInfo);
            mAudioList.remove(0);
        }
    }

    private void _appendVideoBuffers() {
        ///< 视频队列缓冲区推到封装器。
        while (mVideoList.size() > 0) {
            KFBufferFrame packet = mVideoList.get(0);
            mMediaMuxer.writeSampleData(mVideoTrackIndex,packet.buffer,packet.bufferInfo);
            mVideoList.remove(0);
        }
    }

    private void _callBackError(int error, String errorMsg) {
        ///< 错误回调。
        if (mListener != null) {
            mMainHandler.post(()->{
                mListener.muxerOnError(error,TAG + errorMsg);
            });
        }
    }
}
```

上面是 `KFMP4Muxer` 的实现，从代码上可以看到主要有这几个部分：

- 1）创建封装器实例。调用 `start`。
	- 在 `_setupMuxer` 方法中实现，通过输出路径与格式 2 个参数生成。
- 2）创建音视频轨道及添加音频和视频数据。调用 `writeSampleData:` 检测音视频数据会创建对应的轨道。
	- 在 `_setupVideoTrack` 与 `_setupAudioTrack` 方法中实现。音频和视频的格式描述分别为`mVideoFormat`、`mAudioFormat`。
	- 当音频轨道与视频轨道都创建好后，会触发真正的开始 `_start`。这样设计的原因是外层可能优先输入音频或视频，但封装器开始前又需要创建音视频轨道，所以这里实现了等待逻辑。
- 3）用两个队列作为缓冲区，分别管理音频和视频待封装数据。
	- 这两个队列分别是 `mAudioList` 和 `mVideoList`，存储数据类型为 `KFBufferFrame`。
	- 每次当外部调用 `writeSampleData:` 方法送入待封装数据时，都是把数据放入两个队列中的一个，以便根据情况进行后续的音视频数据交织。
- 4）同时封装音频和视频数据时，进行音视频数据交织。
	- 在 `_avInterleavedBuffers` 方法中实现音视频数据交织。当带封装的数据既有音频又有视频，就需要根据他们的时间戳信息进行交织，这样便于在播放该音视频时提升体验。
- 5）音视频数据写入封装。
	- 同时封装音频和视频数据时，在做完音视频交织后，即分别将交织后的音视频数据写入封装器 `mMediaMuxer writeSampleData`。在 `_avInterleavedBuffers` 中实现。
	- 单独封装音频或视频数据时，则直接将数据写入封装器 `mMediaMuxer writeSampleData`。分别在 `_appendAudioBuffers` 和 `_appendVideoBuffers` 方法中实现。
- 6）停止写入。
	- 在 `stop` → `_stop` 方法中实现。
	- 在停止前，还需要消费掉 `mAudioList ` 和 `mVideoList ` 的剩余数据，要调用 `_appendAudioBuffers` 与 `_appendVideoBuffers`。
	- 封装器执行停止操作 `mMediaMuxer stop`。


更具体细节见上述代码及其注释。




## 4、采集音频数据进行 AAC 编码以及 M4A 封装和存储


我们还是在一个 `MainActivity` 中来实现采集音频数据进行 AAC 编码、M4A 封装和存储的逻辑。

`MainActivity.java`

```java
public class MainActivity extends AppCompatActivity {
    private KFAudioCapture mAudioCapture = null; ///< 音频采集
    private KFAudioCaptureConfig mAudioCaptureConfig = null; ///< 音频采集配置
    private KFMediaCodecInterface mEncoder = null; ///< 音频编码
    private MediaFormat mAudioEncoderFormat = null; ///< 音频编码格式描述
    private KFMP4Muxer mMuxer; ///< 封装起器
    private KFMuxerConfig mMuxerConfig; ///< 封装器配置
    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ///< 申请存储、音频采集权限。
        if (ActivityCompat.checkSelfPermission(this, Manifest.permission.RECORD_AUDIO) != PackageManager.PERMISSION_GRANTED || ActivityCompat.checkSelfPermission(this, Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED ||
                ActivityCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED ||
                ActivityCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions((Activity) this,
                    new String[] {Manifest.permission.CAMERA,Manifest.permission.RECORD_AUDIO, Manifest.permission.READ_EXTERNAL_STORAGE, Manifest.permission.WRITE_EXTERNAL_STORAGE},
                    1);
        }

        ///< 创建采集实例。
        mAudioCaptureConfig = new KFAudioCaptureConfig();
        mAudioCapture = new KFAudioCapture(mAudioCaptureConfig,mAudioCaptureListener);
        mAudioCapture.startRunning();

        mMuxerConfig = new KFMuxerConfig(Environment.getExternalStorageDirectory().getPath() + "/test.m4a");
        mMuxerConfig.muxerType = KFMediaBase.KFMediaType.KFMediaAudio;

        FrameLayout.LayoutParams startParams = new FrameLayout.LayoutParams(200, 120);
        startParams.gravity = Gravity.CENTER_HORIZONTAL;
        Button startButton = new Button(this);
        startButton.setTextColor(Color.BLUE);
        startButton.setText("开始");
        startButton.setVisibility(View.VISIBLE);
        startButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                ///< 创建音频编码实例。
                if (mEncoder == null) {
                    mEncoder = new KFAudioByteBufferEncoder();
                    MediaFormat mediaFormat = KFAVTools.createAudioFormat(mAudioCaptureConfig.sampleRate,mAudioCaptureConfig.channel,96*1000);
                    mEncoder.setup(true,mediaFormat,mAudioEncoderListener,null);
                    ((Button)view).setText("停止");
                    mMuxer = new KFMP4Muxer(mMuxerConfig,mMuxerListener);
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
    }

    private KFAudioCaptureListener mAudioCaptureListener = new KFAudioCaptureListener() {
        @Override
        public void onError(int error, String errorMsg) {
            Log.e("KFAudioCapture","errorCode" + error + "msg"+errorMsg);
        }

        @Override
        public void onFrameAvailable(KFFrame frame) {
            ///< 采集回调输入编码。
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

        @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
        @Override
        public void dataOnAvailable(KFFrame frame) {
            ///< 编码回调写入封装器。
            if (mAudioEncoderFormat == null && mEncoder != null) {
                mAudioEncoderFormat = mEncoder.getOutputMediaFormat();
                mMuxer.setAudioMediaFormat(mEncoder.getOutputMediaFormat());
                mMuxer.start();
            }

            if (mMuxer != null) {
                mMuxer.writeSampleData(false,((KFBufferFrame)frame).buffer,((KFBufferFrame)frame).bufferInfo);
            }
        }
    };

    private KFMuxerListener mMuxerListener = new KFMuxerListener() {
        @Override
        public void muxerOnError(int error, String errorMsg) {
            ///< 音频封装错误回调。
            Log.i("KFMuxerListener","error" + error + "msg" + errorMsg);
        }
    };
}
```

上面是 `MainActivity` 的实现，其中主要包含这几个部分：

- 1）在采集音频前需要设置 `Manifest.permission.RECORD_AUDIO` 权限。
- 2）通过启动和停止音频采集来驱动整个采集和编码流程。
- 3）在采集模块 `KFAudioCapture` 的数据回调中将数据交给编码模块 `KFAudioByteBufferEncoder` 进行编码。
	- 在 `KFAudioCaptureListener` 的 `onFrameAvailable` 回调中实现。
- 4）在编码模块 `KFAudioByteBufferEncoder ` 的数据回调中获取编码后的 AAC 裸流数据，并将数据交给封装器 `KFMP4Muxer` 进行封装。
	- 在 `KFMediaCodecListener` 的 `dataOnAvailable` 回调中实现。
- 5）在调用 `stop` 停止整个流程后，如果没有出现错误，封装的 M4A 文件会被存储到 `mMuxerConfig` 设置的路径。




## 5、用工具播放 M4A 文件


完成音频采集和编码后，可以将 `sdcard` 文件夹下面的 `test.m4a` 文件拷贝到电脑上，使用 `ffplay` 播放来验证一下音频采集是效果是否符合预期：

```c
$ ffplay -i test.m4a
```

关于播放 M4A 文件的工具，可以参考[《FFmpeg 工具》第 2 节 ffplay 命令行工具](https://mp.weixin.qq.com/s/Rl7fxOP-YH37mQEvGxhfUA)和[《可视化音视频分析工具》第 1.1 节 Adobe Audition](https://mp.weixin.qq.com/s/jCYih3qgEIUctuWxn0aTGQ)。


上面我们讲过 M4A 格式是属于 MPEG-4 标准，所以我们这里还可以用[《可视化音视频分析工具》第 3.1 节 MP4Box.js
](https://mp.weixin.qq.com/s/jCYih3qgEIUctuWxn0aTGQ) 等工具来查看它的格式：


![Demo 生成的 M4A 文件结构](assets/resource/av-demo/av-demo-ios-audio-muxer-1.png)
_Demo 生成的 M4A 文件结构_




