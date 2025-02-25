---
title: Android AVDemo（1）：音频采集，代码开源并提供解析
description: 介绍 Android 音频采集的流程和原理，并提供 Demo 源码和解析。
author: Keyframe
date: 2025-02-23 09:38:08 +0800
categories: [音视频源码示例]
tags: [音视频源码示例, 音视频, Android, 音频, 音频采集]
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

这里是 Android 第一篇：**Android 音频采集 Demo**。这个 Demo 里包含以下内容：

- 1）实现一个音频采集模块；
- 2）实现音频采集逻辑并将采集的音频存储为 PCM 数据；
- 3）详尽的代码注释，帮你理解代码逻辑和原理。


在本文中，我们将详解一下 Demo 的具体实现和源码。读完本文内容相信就能帮你掌握相关知识。

>>不过，如果你的需求是：1）直接获得全部工程源码；2）想进一步咨询音视频技术问题；3）咨询音视频职业发展问题。可以根据自己的需要考虑是否加入『关键帧的音视频开发圈』，这是一个收费的社群服务，我们在这里答疑解惑和分享资料，你可以通过下面的二维码加入。
>>![长按识别二维码→加入我们](assets/img/keyframe-zsxq.png)
>>_长按识别二维码→加入我们_



## 1、音频采集模块

首先，实现一个 `KFAudioConfig` 类用于定义音频采集参数的配置。这里包括了：采样率、声道数这几个参数。这几个参数的含义在前面介绍声音基础的文章[声音的表示（3）：声音的数字化](https://mp.weixin.qq.com/s/lexAVx_O3Kz3-51OZ3TlLw)中有过介绍。


`KFAudioCaptureConfig.java`

```java
public class KFAudioCaptureConfig {
    public int sampleRate = 44100;
    public int channel = 1;
}
```

接下来，我们实现一个 `KFAudioCaptureListener` 类来实现采集回调，包含错误回调与数据回调。

`KFAudioCaptureListener.java`

```java
public interface KFAudioCaptureListener {
    void onError(int error,String errorMsg);
    void onFrameAvailable(KFFrame frame);
}
```

上面的 `KFFrame` 是音频数据对象，数据包含 Buffer 数据与 Texture 数据，音频仅涉及 Buffer 数据。

`KFFrame.java`

```java
@RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
public class KFFrame {
    public enum KFFrameType {
        KFFrameBuffer,
        KFFrameTexture;
    }

    public KFFrameType frameType = KFFrameType.KFFrameBuffer;
    public KFFrame(KFFrameType type) {
        frameType = type;
    }
}
```

音频 Buffer 数据 `KFBufferFrame`，继承自 `KFFrame`，包含 ByteBuffer 数据与 BufferInfo 数据信息。BufferInfo 为了提供时间戳 presentationTimeUs 与 size。

`KFBufferFrame.java`

```java
public class KFBufferFrame extends KFFrame {
    public ByteBuffer buffer;
    public MediaCodec.BufferInfo bufferInfo;

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    public KFBufferFrame() {
        super(KFFrameBuffer);
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    public KFBufferFrame(ByteBuffer inputBuffer, MediaCodec.BufferInfo inputBufferInfo) {
        super(KFFrameBuffer);
        buffer = inputBuffer;
        bufferInfo = inputBufferInfo;
    }

    public KFFrameType frameType() {
        return KFFrameBuffer;
    }
}
```

最后我们实现一个 `KFAudioCapture ` 类来实现音频采集。

`KFAudioCapture.java`

```java
public class KFAudioCapture {
    public static int KFAudioCaptureErrorCreate = -2600;
    public static int KFAudioCaptureErrorStart = -2601;
    public static int KFAudioCaptureErrorStop = -2602;

    private static final String TAG = "KFAudioCapture";
    private KFAudioCaptureConfig mConfig = null; ///< 音频配置
    private KFAudioCaptureListener mListener = null; ///< 音频回调
    private HandlerThread mRecordThread = null; ///< 音频采集线程
    private Handler mRecordHandle = null;

    private HandlerThread mReadThread = null; ///< 音频读数据线程
    private Handler mReadHandle = null;
    private int mMinBufferSize = 0;

    private AudioRecord mAudioRecord = null; ///< 音频采集实例
    private boolean mRecording = false;
    private Handler mMainHandler = new Handler(Looper.getMainLooper()); ///< 主线程用作错误回调

    public KFAudioCapture(KFAudioCaptureConfig config,KFAudioCaptureListener listener) {
        mConfig = config;
        mListener = listener;

        mRecordThread = new HandlerThread("KFAudioCaptureThread");
        mRecordThread.start();
        mRecordHandle = new Handler((mRecordThread.getLooper()));

        mReadThread = new HandlerThread("KFAudioCaptureReadThread");
        mReadThread.start();
        mReadHandle = new Handler((mReadThread.getLooper()));

        mRecordHandle.post(()->{
            ///< 初始化音频采集实例。
            _setupAudioRecord();
        });
    }

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    public void startRunning() {
        ///< 开启音频采集。
        mRecordHandle.post(()->{
            if (mAudioRecord != null && !mRecording) {
                try {
                    mAudioRecord.startRecording();
                    mRecording = true;
                } catch (Exception e) {
                    Log.e(TAG,e.getMessage());
                    _callBackError(KFAudioCaptureErrorStart,e.getMessage());
                }
                
                ///< 音频采集采用拉数据模式，通过读数据线程开启循环无限拉取 PCM 数据，拉到数据后进行回调。
                mReadHandle.post(()->{
                    while (mRecording) {
                        final byte[] pcmData = new byte[mMinBufferSize];
                        int readSize = mAudioRecord.read(pcmData, 0, mMinBufferSize);
                        if (readSize > 0) {
                            ///< 处理音频数据 data。
                            ByteBuffer buffer = ByteBuffer.allocateDirect(readSize).put(pcmData).order(ByteOrder.nativeOrder());
                            buffer.position(0);
                            MediaCodec.BufferInfo bufferInfo = new MediaCodec.BufferInfo();
                            bufferInfo.presentationTimeUs = System.nanoTime() / 1000;
                            bufferInfo.size = readSize;
                            KFBufferFrame bufferFrame = new KFBufferFrame(buffer,bufferInfo);
                            if (mListener != null) {
                                mListener.onFrameAvailable(bufferFrame);
                            }
                        }
                    }
                });
            }
        });
    }

    public void stopRunning() {
        ///< 关闭音频采集。
        mRecordHandle.post(()->{
            if (mAudioRecord != null && mRecording) {
                try {
                    mAudioRecord.stop();
                    mRecording = false;
                } catch (Exception e) {
                    Log.e(TAG,e.getMessage());
                    _callBackError(KFAudioCaptureErrorStart,e.getMessage());
                }
            }
        });
    }

    public void release() {
        ///< 外层主动触发释放，释放采集实例、线程。
        mRecordHandle.post(()->{
            if (mAudioRecord != null) {
                if (mRecording) {
                    try {
                        mAudioRecord.stop();
                        mRecording = false;
                    } catch (Exception e) {
                        Log.e(TAG,e.getMessage());
                    }
                }

                try {
                    mAudioRecord.release();
                } catch (Exception e) {
                    Log.e(TAG,e.getMessage());
                }
                mAudioRecord = null;
            }

            mRecordThread.quit();
            mReadThread.quit();
        });
    }

    private void _setupAudioRecord() {
        if (mAudioRecord == null) {
            ///< 根据指定采样率、声道、位深获取每次回调数据大小。
            mMinBufferSize = AudioRecord.getMinBufferSize(mConfig.sampleRate, mConfig.channel, AudioFormat.ENCODING_PCM_16BIT);
            try {
                ///< 根据采样率、声道、位深每次回调数据大小生成采集实例。
                mAudioRecord = new AudioRecord(MediaRecorder.AudioSource.MIC,mConfig.sampleRate,mConfig.channel, AudioFormat.ENCODING_PCM_16BIT,mMinBufferSize);
            } catch (Exception e) {
                Log.e(TAG,e.getMessage());
                _callBackError(KFAudioCaptureErrorCreate,e.getMessage());
            };
        }
    }

    private void _callBackError(int error, String errorMsg) {
        ///< 错误回调。
        if (mListener != null) {
            mMainHandler.post(()->{
                mListenjavaer.onError(error,TAG + errorMsg);
            });
        }
    }
}
```

上面是 `KFAudioCapture` 的实现，从代码上可以看到主要有这几个部分：

- 1）创建音频采集实例，`_setupAudioRecord` 根据采样率、声道、位深、回调数据大小来创建音频采集实例。每次回调数据大小这里反应拉取数据的频率，对于直播等场景可以设置小一些，有利于降低延迟。
- 2）开启音频采集，`startRunning`，这里需要关注开启单独线程拉取 PCM 数据任务，将拉取到的数据回调给外层。
- 3）关闭音频采集，`stopRunning`。
- 4）清理音频采集实例，`release`。



## 2、采集音频存储为 PCM 文件

我们在一个 `MainActivity` 中来实现音频采集逻辑并将采集的音频存储为 PCM 数据。

`MainActivity.java`

```java
public class MainActivity extends AppCompatActivity {
    private FileOutputStream mStream = null;
    private KFAudioCapture mAudioCapture = null;
    private KFAudioCaptureConfig mAudioCaptureConfig = null;

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ///< 音频录制权限。
        if (ActivityCompat.checkSelfPermission(this, Manifest.permission.RECORD_AUDIO) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions((Activity) this,
                    new String[] {Manifest.permission.CAMERA,Manifest.permission.RECORD_AUDIO},
                    1);
        }

        mAudioCaptureConfig = new KFAudioCaptureConfig();
        mAudioCapture = new KFAudioCapture(mAudioCaptureConfig,mAudioCaptureListener);
        mAudioCapture.startRunning();
        
        if (mStream == null) {
            try {
                mStream = new FileOutputStream(Environment.getExternalStorageDirectory().getPath() + "/test.pcm");
            } catch (FileNotFoundException e) {
                e.printStackTrace();
            }
        }
    }

    ///< 音频采集回调。
    private KFAudioCaptureListener mAudioCaptureListener = new KFAudioCaptureListener() {
        @Override
        public void onError(int error, String errorMsg) {
            Log.e("KFAudioCapture","errorCode" + error + "msg"+errorMsg);
        }

        @Override
        public void onFrameAvailable(KFFrame frame) {
            ///< 获取到音频 Buffer 数据存储到本地 PCM。
            try {
                ByteBuffer pcmData = ((KFBufferFrame)frame).buffer;
                byte[] ppsBytes = new byte[pcmData.capacity()];
                pcmData.get(ppsBytes);
                mStream.write(ppsBytes);
            }  catch (IOException e) {
                e.printStackTrace();
            }
        }
    };
}
```

上面是 `MainActivity` 的实现，这里需要注意的是在采集音频前需要判断录制权限 `Manifest.permission.RECORD_AUDIO`。


## 3、用工具播放 PCM 文件


完成音频采集后，可以将 `sdcard` 文件夹下面的 `test.pcm` 文件拷贝到电脑上，使用 `ffplay ` 播放来验证一下音频采集是效果是否符合预期：


```c
$ ffplay -ar 44100 -channels 1 -f s16le -i test.pcm
```

注意这里的参数要对齐在工程代码中设置的`采样率`、`声道数`、`采样位深`。

关于播放 PCM 文件的工具，可以参考[《FFmpeg 工具》第 2 节 ffplay 命令行工具](https://mp.weixin.qq.com/s/Rl7fxOP-YH37mQEvGxhfUA)和[《可视化音视频分析工具》第 1.1 节 Adobe Audition](https://mp.weixin.qq.com/s/jCYih3qgEIUctuWxn0aTGQ)。

