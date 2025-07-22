---
title: Android AVDemo（13）：视频渲染，代码开源并提供解析
description: 介绍 Android 视频渲染流程和原理，并提供 Demo 源码和解析。
author: Keyframe
date: 2025-02-23 12:08:08 +0800
categories: [音视频源码示例]
tags: [音视频源码示例, 音视频, Android, 视频, 视频渲染, 渲染]
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

这里是 Android 第十三篇：**Android 视频渲染 Demo**。这个 Demo 里包含以下内容：

- 1）实现一个视频采集装模块；
- 2）实现一个视频渲染模块；
- 3）串联视频采集和渲染模块，将采集的视频数据输入给渲染模块进行渲染；
- 4）详尽的代码注释，帮你理解代码逻辑和原理。



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


## 2、视频渲染模块

在之前的[《Android 视频采集 Demo》](https://mp.weixin.qq.com/s/rS077Uu4qJnDJMDD7vto5Q)那篇中，我们采集后的视频数据是通过 `KFRenderView` 来做预览渲染的。这篇我们来介绍一下使用 `KFRenderView` 内部管理 `KFSurfaceView`、`KFTextureView`，并且使用 OpenGL 实现渲染功能。


首先，我们在 `KFRenderListener` 中定义渲染回调。


`KFRenderListener.java`

```java
public interface KFRenderListener {
    void surfaceCreate(@NonNull Surface surface); ///< 渲染缓存创建.
    void surfaceChanged(@NonNull Surface surface, int width, int height); ///< 渲染缓存变更分辨率。
    void surfaceDestroy(@NonNull Surface surface); ///< 渲染缓存销毁。
}
```

然后，我们在 `KFRenderView` 中管理 `KFSurfaceView`、`KFTextureView` 以及具体渲染逻辑。


`KFRenderView.java`


```java
public class KFRenderView extends ViewGroup {
    private KFGLContext mEGLContext = null; ///< OpenGL 上下文。
    private KFGLFilter mFilter = null; ///< 特效渲染到指定 Surface。
    private EGLContext mShareContext = null; ///< 共享上下文。
    private View mRenderView = null; ///< 渲染视图基类。
    private int mSurfaceWidth = 0; ///< 渲染缓存宽。
    private int mSurfaceHeight = 0; ///< 渲染缓存高。
    private FloatBuffer mSquareVerticesBuffer = null; ///< 自定义顶点.
    private KFRenderMode mRenderMode = KFRenderMode.KFRenderModeFill; ///< 自适应模式，黑边，比例填冲。
    private boolean mSurfaceChanged = false; ///< 渲染缓存是否变更。
    private Size mLastRenderSize = new Size(0,0); ///< 标记上次渲染 Size。

    public enum KFRenderMode {
        KFRenderStretch, ///< 拉伸满，可能变形。
        KFRenderModeFit, ///< 黑边。
        KFRenderModeFill ///< 比例填充。
    };

    public KFRenderView(Context context, EGLContext eglContext) {
        super(context);
        mShareContext = eglContext; ///< 共享上下文。
        _setupSquareVertices(); ///< 初始化顶点。

        boolean isSurfaceView  = false; ///< TextureView 与 SurfaceView 开关。
        if (isSurfaceView) {
            mRenderView = new KFSurfaceView(context, mListener);
        } else {
            mRenderView = new KFTextureView(context, mListener);
        }

        this.addView(mRenderView);// 添加视图到父视图
    }

    public void release() {
        ///< 释放 GL 上下文、特效。
        if (mEGLContext != null) {
            mEGLContext.bind();
            if (mFilter != null){ 
                mFilter.release();
                mFilter = null;
            }
            mEGLContext.unbind();

            mEGLContext.release();
            mEGLContext = null;
        }
    }

    public void render(KFTextureFrame inputFrame) {
        if (inputFrame == null) {
            return;
        }

        ///< 输入纹理使用自定义特效渲染到 View 的 Surface 上。
        if (mEGLContext != null && mFilter != null) {
            boolean frameResolutionChanged = inputFrame.textureSize.getWidth() != mLastRenderSize.getWidth() || inputFrame.textureSize.getHeight() != mLastRenderSize.getHeight();
            ///< 渲染缓存变更或者视图大小变更重新设置顶点。
            if (mSurfaceChanged || frameResolutionChanged) {
                _recalculateVertices(inputFrame.textureSize);
                mSurfaceChanged = false;
                mLastRenderSize = inputFrame.textureSize;
            }

            ///< 渲染到指定 Surface。
            mEGLContext.bind();
            mFilter.setSquareVerticesBuffer(mSquareVerticesBuffer);
            GLES20.glViewport(0, 0, mSurfaceWidth, mSurfaceHeight);
            mFilter.render(inputFrame);
            mEGLContext.swapBuffers();
            mEGLContext.unbind();
        }
    }

    private KFRenderListener mListener = new KFRenderListener() {
        @Override
        ///< 渲染缓存创建。
        public void surfaceCreate(@NonNull Surface surface) {
            mEGLContext = new KFGLContext(mShareContext,surface);
            ///< 初始化特效。
            mEGLContext.bind();
            _setupFilter();
            mEGLContext.unbind();
        }

        @Override
        ///< 渲染缓存变更。
        public void surfaceChanged(@NonNull Surface surface, int width, int height) {
            mSurfaceWidth = width;
            mSurfaceHeight = height;
            mSurfaceChanged = true;
            ///< 设置 GL 上下文 Surface。
            mEGLContext.bind();
            mEGLContext.setSurface(surface);
            mEGLContext.unbind();
        }

        @Override
        public void surfaceDestroy(@NonNull Surface surface) {

        }
    };

    private void _setupFilter() {
        ///< 初始化特效。
        if (mFilter == null) {
            mFilter = new KFGLFilter(true, KFGLBase.defaultVertexShader,KFGLBase.defaultFragmentShader);
        }
    }

    private void _setupSquareVertices() {
        ///< 初始化顶点缓存。
        final float squareVertices[] = {
                -1.0f, -1.0f,
                1.0f, -1.0f,
                -1.0f,  1.0f,
                1.0f,  1.0f,
        };

        ByteBuffer squareVerticesByteBuffer = ByteBuffer.allocateDirect(4 * squareVertices.length);
        squareVerticesByteBuffer.order(ByteOrder.nativeOrder());
        mSquareVerticesBuffer = squareVerticesByteBuffer.asFloatBuffer();
        mSquareVerticesBuffer.put(squareVertices);
        mSquareVerticesBuffer.position(0);
    }

    private void _recalculateVertices(Size inputImageSize) {
        ///< 按照适应模式创建顶点。
        if (mSurfaceWidth == 0 || mSurfaceHeight == 0) {
            return;
        }

        Size renderSize = new Size(mSurfaceWidth,mSurfaceHeight);
        float heightScaling = 1, widthScaling = 1;
        Size insetSize = new Size(0,0);
        float inputAspectRatio = (float) inputImageSize.getWidth() / (float)inputImageSize.getHeight();
        float outputAspectRatio = (float)renderSize.getWidth() / (float)renderSize.getHeight();
        boolean isAutomaticHeight = inputAspectRatio <= outputAspectRatio ? false : true;

        if (isAutomaticHeight) {
            float insetSizeHeight = (float)inputImageSize.getHeight() / ((float)inputImageSize.getWidth() / (float)renderSize.getWidth());
            insetSize = new Size(renderSize.getWidth(),(int)insetSizeHeight);
        } else {
            float insetSizeWidth = (float)inputImageSize.getWidth() / ((float)inputImageSize.getHeight() / (float)renderSize.getHeight());
            insetSize = new Size((int)insetSizeWidth,renderSize.getHeight());
        }

        switch (mRenderMode) {
            case KFRenderStretch: {
                widthScaling = 1;
                heightScaling = 1;
            }; break;
            case KFRenderModeFit: {
                widthScaling = (float)insetSize.getWidth() / (float)renderSize.getWidth();
                heightScaling = (float)insetSize.getHeight() / (float)renderSize.getHeight();
            }; break;
            case KFRenderModeFill: {
                widthScaling = (float) renderSize.getHeight() / (float)insetSize.getHeight();
                heightScaling = (float)renderSize.getWidth() / (float)insetSize.getWidth();
            }; break;
        }

        final float squareVertices[] = {
                -1.0f, -1.0f,
                1.0f, -1.0f,
                -1.0f,  1.0f,
                1.0f,  1.0f,
        };

        final float customVertices[] = {
                -widthScaling, -heightScaling,
                widthScaling, -heightScaling,
                -widthScaling,  heightScaling,
                widthScaling,  heightScaling,
        };
        ByteBuffer squareVerticesByteBuffer = ByteBuffer.allocateDirect(4 * customVertices.length);
        squareVerticesByteBuffer.order(ByteOrder.nativeOrder());
        mSquareVerticesBuffer = squareVerticesByteBuffer.asFloatBuffer();
        mSquareVerticesBuffer.put(customVertices);
        mSquareVerticesBuffer.position(0);
    }

    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        ///< 视图变更 Size。
        this.mRenderView.layout(left,top,right,bottom);
    }
}
```


上面是 `KFRenderView` 的实现，继承自 ViewGroup，其中主要包含这几个部分：


- 1）管理 `KFSurfaceView`、`KFTextureView`。
	- 通过 `isSurfaceView` 开关来控制使用 `TextureView` 或 `SurfaceView`。 
	- `SurfaceView` 性能高一些，可以在自线程更新 UI 的 View，遵循双缓冲机制，但无法像正常视图实现动画。
	- `TextureView` 性能稍微差一点，重载了 Draw 方法，可以像正常视图实现动画。
- 2）创建 OpenGL 上下文。
	- 这里需要注意的是，我们通过 `mEGLContext` 管理上下文，初始化方法需要输入渲染视图 Surface 作为参数，这样就可以将绘制结果直接渲染到指定 Surface。
- 3）创建渲染特效。
	- 通过 `mFilter` 将纹理数据渲染到渲染视图 Surface。
	- 通过 `mRenderMode` 控制自定义渲染比例，在方法 `_recalculateVertices` 中实时计算顶点数据来实现。
- 4）渲染回调通知 Surface 生命周期。
	- 在 `KFRenderListener ` 的 `surfaceCreate` 回调中通知 Surface 创建。
	- 在 `KFRenderListener ` 的 `surfaceChanged` 回调中通知 Surface 变更。
	- 在 `KFRenderListener ` 的 `surfaceDestroy` 回调中通知 Surface 销毁。
	
接下来是内部渲染视图 `KFSurfaceView` 的实现，需要注意 Surface 是通过 `surfaceHolder` 方法 `getSurface` 　获取：

`KFSurfaceView.java`

```java
public class KFSurfaceView extends SurfaceView implements SurfaceHolder.Callback {
    private KFRenderListener mListener = null; ///< 回调。
    private SurfaceHolder mHolder = null; ///< Surface 的抽象接口。

    public KFSurfaceView(Context context, KFRenderListener listener) {
        super(context);
        mListener = listener;
        getHolder().addCallback(this);
    }

    @Override
    public void surfaceCreated(@NonNull SurfaceHolder surfaceHolder) {
        ///< Surface 创建。
        mHolder = surfaceHolder;
        ///< 根据 SurfaceHolder 创建 Surface。
        if (mListener != null) {
            mListener.surfaceCreate(surfaceHolder.getSurface());
        }
    }

    @Override
    public void surfaceChanged(@NonNull SurfaceHolder surfaceHolder, int format, int width, int height) {
        ///< Surface 分辨率变更。
        if (mListener != null) {
            mListener.surfaceChanged(surfaceHolder.getSurface(),width,height);
        }
    }

    @Override
    public void surfaceDestroyed(@NonNull SurfaceHolder surfaceHolder) {
        ///< Surface 销毁。
        if (mListener != null) {
            mListener.surfaceDestroy(surfaceHolder.getSurface());
        }
    }
}
```

接下来是内部渲染视图 `KFTextureView` 的实现，需要注意 Surface 是通过 `surfaceTexture` 创建生成：

`KFTextureView.java`

```java
public class KFTextureView extends TextureView implements TextureView.SurfaceTextureListener {
    private KFRenderListener mListener = null; ///< 回调。
    private Surface mSurface = null; ///< 渲染缓存。
    private SurfaceTexture mSurfaceTexture = null; ///< 纹理缓存。

    public KFTextureView(Context context, KFRenderListener listener) {
        super(context);
        this.setSurfaceTextureListener(this);
        mListener = listener;
    }

    @Override
    public void onSurfaceTextureAvailable(@NonNull SurfaceTexture surfaceTexture, int width, int height) {
        ///< 纹理缓存创建。
        mSurfaceTexture = surfaceTexture;
        ///< 根据 SurfaceTexture 创建 Surface。
        mSurface = new Surface(surfaceTexture);
        if (mListener != null) {
            ///< 创建时候回调一次分辨率变更，对其 SurfaceView 接口。
            mListener.surfaceCreate(mSurface);
            mListener.surfaceChanged(mSurface,width,height);
        }
    }

    @Override
    public void onSurfaceTextureSizeChanged(@NonNull SurfaceTexture surfaceTexture, int width, int height) {
        ///< 纹理缓存变更分辨率。
        if (mListener != null) {
            mListener.surfaceChanged(mSurface,width,height);
        }
    }

    @Override
    public void onSurfaceTextureUpdated(@NonNull SurfaceTexture surfaceTexture) {

    }

    @Override
    public boolean onSurfaceTextureDestroyed(@NonNull SurfaceTexture surfaceTexture) {
        ///< 纹理缓存销毁。
        if (mListener != null) {
            mListener.surfaceDestroy(mSurface);
        }
        if (mSurface != null) {
            mSurface.release();
            mSurface = null;
        }
        return false;
    }
}
```


更具体细节见上述代码及其注释。


## 3、采集视频数据并渲染



我们在一个 `MainActivity` 中来实现对采集的视频数据进行渲染播放。


`MainActivity.java`


```java
public class MainActivity extends AppCompatActivity {

    private KFIVideoCapture mCapture; ///< 相机采集。
    private KFVideoCaptureConfig mCaptureConfig; ///< 相机采集配置。
    private KFRenderView mRenderView; ///< 渲染视图。
    private KFGLContext mGLContext; ///< OpenGL 上下文。


    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ///< 检测采集相关权限。
        if (ActivityCompat.checkSelfPermission(this, Manifest.permission.RECORD_AUDIO) != PackageManager.PERMISSION_GRANTED || ActivityCompat.checkSelfPermission(this, Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED ||
                ActivityCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED ||
                ActivityCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions((Activity) this,
                    new String[] {Manifest.permission.CAMERA,Manifest.permission.RECORD_AUDIO, Manifest.permission.READ_EXTERNAL_STORAGE, Manifest.permission.WRITE_EXTERNAL_STORAGE},
                    1);
        }

        ///< OpenGL 上下文。
        mGLContext = new KFGLContext(null);
        ///< 渲染视图。
        mRenderView = new KFRenderView(this,mGLContext.getContext());
        WindowManager windowManager = (WindowManager)this.getSystemService(this.WINDOW_SERVICE);
        Rect outRect = new Rect();
        windowManager.getDefaultDisplay().getRectSize(outRect);
        FrameLayout.LayoutParams params = new FrameLayout.LayoutParams(outRect.width(), outRect.height());
        addContentView(mRenderView,params);

        ///< 采集配置：摄像头方向、分辨率、帧率。
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
    }

    private KFVideoCaptureListener mVideoCaptureListener = new KFVideoCaptureListener() {
        @Override
        ///< 相机打开回调。
        public void cameraOnOpened(){}

        @Override
        ///< 相机关闭回调。
        public void cameraOnClosed() {
        }

        @Override
        ///< 相机出错回调。
        public void cameraOnError(int error,String errorMsg) {

        }

        @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
        @Override
        ///< 相机数据回调。
        public void onFrameAvailable(KFFrame frame) {
            mRenderView.render((KFTextureFrame) frame);
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
- 3）采集数据回调 `onFrameAvailable`，将数据输入给渲染视图进行预览。

更具体细节见上述代码及其注释。







---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
{: .prompt-tip }

