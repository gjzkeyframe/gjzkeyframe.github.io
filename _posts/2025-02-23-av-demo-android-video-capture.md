---
title: Android AVDemo（7）：视频采集，代码开源并提供解析
description: 介绍 Android 视频采集流程和原理，并提供 Demo 源码和解析。
author: Keyframe
date: 2025-02-23 11:08:08 +0800
categories: [音视频源码示例]
tags: [音视频源码示例, 音视频, Android, 视频, 视频采集]
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

这里是 Android 第七篇：**Android 视频采集 Demo**。这个 Demo 里包含以下内容：

- 1）实现两个视频采集模块，分别为 `Camera` 与 `Camera2`；
- 2）实现视频采集逻辑并将采集的视频图像渲染进行预览；
- 3）详尽的代码注释，帮你理解代码逻辑和原理。

在本文中，我们将详解一下 Demo 的具体实现和源码。读完本文内容相信就能帮你掌握相关知识。

>>不过，如果你的需求是：1）直接获得全部工程源码；2）想进一步咨询音视频技术问题；3）咨询音视频职业发展问题。可以根据自己的需要考虑是否加入『关键帧的音视频开发圈』。
>>
>>![长按识别二维码→加入我们](assets/img/keyframe-zsxq.png)
>>_长按识别二维码→加入我们_

## 1、视频采集模块 Camera

首先，实现一个 `KFVideoCaptureConfig` 类用于定义视频采集参数的配置。


`KFVideoCaptureConfig.java`

```java
public class KFVideoCaptureConfig {
    ///< 摄像头方向。
    public Integer cameraFacing = CameraCharacteristics.LENS_FACING_FRONT;
    ///< 分辨率。
    public Size resolution = new Size(1080,1920);
    ///< 帧率。
    public Integer fps = 30;
}
```

这里的参数包括了：分辨率、摄像头方向、帧率这几个参数。


接下来，我们实现一个 `KFIVideoCapture` 类来实现视频采集接口。


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

上面是 `KFIVideoCapture` 的接口设计，主要包含 `初始化`、`开始采集`、`停止采集`、`切换摄像头`等接口。

下面是数据回调接口 `KFVideoCaptureListener`。

`KFVideoCaptureListener.java`


```java
public interface KFVideoCaptureListener {
    ///< 摄像机打开。
    void cameraOnOpened();
    ///< 摄像机关闭。
    void cameraOnClosed();
    ///< 摄像机出错。
    void cameraOnError(int error,String errorMsg);
    ///< 数据回调给外层。
    void onFrameAvailable(KFFrame frame);
}
```

提供了`相机打开回调`、`相机关闭回调`、以及`相机出错回调`的接口。外层可以根据 `相机打开回调` 优先将 CPU 等资源分配给相机，打开成功后执行 UI 等其它布局，提升用户体验。


接下来，我们实现一个 `KFVideoCaptureV1` 类来实现视频采集。

`KFVideoCaptureV1.java`

```java
public class KFVideoCaptureV1 implements KFIVideoCapture {
    public static final int KFVideoCaptureV1CameraDisableError = -3000;
    private static final String TAG = "KFVideoCaptureV1";

    private KFVideoCaptureListener mListener = null; ///< 回调。
    private KFVideoCaptureConfig mConfig = null; ///< 配置。
    private WeakReference<Context> mContext = null;
    private boolean mCameraIsRunning = false; ///< 是否正在采集。

    private HandlerThread mCameraThread = null; ///< 采集线程。
    private Handler mCameraHandler = null;

    private KFGLContext mGLContext = null; ///< GL 特效上下文。
    private KFSurfaceTexture mSurfaceTexture = null; ///< Surface 纹理。
    private HandlerThread mRenderThread = null; ///< 渲染线程。
    private Handler mRenderHandler = null;
    private Handler mMainHandler = new Handler(Looper.getMainLooper()); ///< 主线程。

    private Camera.CameraInfo mFrontCameraInfo = null; ///< 前置摄像头信息。
    private int mFrontCameraId = -1;
    private Camera.CameraInfo mBackCameraInfo = null; ///< 后置摄像头信息。
    private int mBackCameraId = -1;
    private Camera mCamera = null; ///< 当前摄像头实例（前置或者后置）。

    public KFVideoCaptureV1() {

    }

    @Override
    public void setup(Context context, KFVideoCaptureConfig config, KFVideoCaptureListener listener, EGLContext eglShareContext) {
        mListener = listener;
        mConfig = config;
        mContext = new WeakReference<Context>(context);

        ///< 采集线程。
        mCameraThread = new HandlerThread("KFCameraThread");
        mCameraThread.start();
        mCameraHandler = new Handler((mCameraThread.getLooper()));

        ///< 渲染线程。
        mRenderThread = new HandlerThread("KFCameraRenderThread");
        mRenderThread.start();
        mRenderHandler = new Handler((mRenderThread.getLooper()));

        ///< OpenGL 上下文。
        mGLContext = new KFGLContext(eglShareContext);
    }

    @Override
    public EGLContext getEGLContext() {
        return mGLContext.getContext();
    }

    @Override
    public boolean isRunning() {
        return mCameraIsRunning;
    }

    @Override
    public void release() {
        mCameraHandler.post(() -> {
            ///< 停止视频采集 清晰视频采集实例、OpenGL 上下文、线程等。
            _stopRunning();
            mGLContext.bind();
            if (mSurfaceTexture != null) {
                mSurfaceTexture.release();
                mSurfaceTexture = null;
            }
            mGLContext.unbind();
            mGLContext.release();
            mGLContext = null;

            if (mCamera != null) {
                mCamera.release();
                mCamera = null;
            }

            mCameraThread.quit();
            mRenderThread.quit();
        });
    }

    @Override
    public void startRunning() {
        mCameraHandler.post(() -> {
            ///< 检测视频采集权限。
            if (ActivityCompat.checkSelfPermission(mContext.get(), Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
                ActivityCompat.requestPermissions((Activity) mContext.get(), new String[] {Manifest.permission.CAMERA}, 1);
            }
            ///< 检测相机是否可用。
            if (!_checkCameraService()) {
                _callBackError(KFVideoCaptureV1CameraDisableError, "相机不可用");
                return;
            }

            ///< 开启视频采集。
            _startRunning();
        });
    }

    @Override
    public void stopRunning() {
        mCameraHandler.post(() -> {
            _stopRunning();
        });
    }

    @Override
    public void switchCamera() {
        mCameraHandler.post(() -> {
            ///< 切换摄像头，先关闭相机调整方向再打开相机。
            _stopRunning();
            mConfig.cameraFacing = mConfig.cameraFacing == CameraCharacteristics.LENS_FACING_FRONT ? CameraCharacteristics.LENS_FACING_BACK : CameraCharacteristics.LENS_FACING_FRONT;
            _startRunning();
        });
    }

    private void _startRunning() {
        ///< 获取前后台摄像机信息。
        if (mFrontCameraInfo == null || mBackCameraInfo == null) {
            _initCameraInfo();
        }
        
        try {
            ///< 根据前后台摄像头 id 打开相机实例。
            mCamera = Camera.open(_getCurrentCameraId());
            if (mCamera != null) {
                ///< 设置相机各分辨率、帧率、方向。
                Camera.Parameters parameters = mCamera.getParameters();
                Size previewSize = _getOptimalSize(mConfig.resolution.getWidth(), mConfig.resolution.getHeight());
                mConfig.resolution = new Size(previewSize.getHeight(),previewSize.getWidth());
                parameters.setPreviewSize(previewSize.getWidth(),previewSize.getHeight());
                Range<Integer> selectFpsRange = _chooseFpsRange();
                if (selectFpsRange.getUpper() > 0) {
                    parameters.setPreviewFpsRange(selectFpsRange.getLower(),selectFpsRange.getUpper());
                }
                mCamera.setParameters(parameters);
                mCamera.setDisplayOrientation(_getDisplayOrientation());
                ///< 创建 Surface 纹理。
                if (mSurfaceTexture == null) {
                    mGLContext.bind();
                    mSurfaceTexture = new KFSurfaceTexture(mSurfaceTextureListener);
                    mGLContext.unbind();
                }
                ///< 设置 SurfaceTexture 给 Camera，这样 Camera 自动将数据渲染到 SurfaceTexture。
                mCamera.setPreviewTexture(mSurfaceTexture.getSurfaceTexture());
                ///< 开启预览。
                mCamera.startPreview();
                mCameraIsRunning = true;
                if (mListener != null) {
                    mMainHandler.post(()->{
                        ///< 回调相机打开。
                        mListener.cameraOnOpened();
                    });
                }
            }
        } catch (RuntimeException | IOException e) {
            e.printStackTrace();
        }
    }

    private void _stopRunning() {
        if (mCamera != null) {
            ///< 关闭相机采集。
            mCamera.setPreviewCallback(null);
            mCamera.stopPreview();
            mCamera.release();
            mCamera = null;
            mCameraIsRunning = false;
            if (mListener != null) {
                mMainHandler.post(()->{
                    ///< 回调相机关闭。
                    mListener.cameraOnClosed();
                });
            }
        }
    }

    private int _getCurrentCameraId() {
        ///< 获取当前摄像机 id。
        if (mConfig.cameraFacing == CameraCharacteristics.LENS_FACING_FRONT) {
            return mFrontCameraId;
        } else if (mConfig.cameraFacing == CameraCharacteristics.LENS_FACING_BACK) {
            return mBackCameraId;
        } else {
            throw new RuntimeException("No available camera id found.");
        }
    }

    private int _getDisplayOrientation() {
        ///< 获取摄像机需要旋转的方向。
        int orientation = 0;
        if (mConfig.cameraFacing == CameraCharacteristics.LENS_FACING_FRONT) {
            orientation = (_getCurrentCameraInfo().orientation) % 360;
            orientation = (360 - orientation) % 360;
        } else {
            orientation = (_getCurrentCameraInfo().orientation + 360) % 360;
        }
        return orientation;
    }

    private Camera.CameraInfo _getCurrentCameraInfo() {
        ///< 获取当前摄像机描述信息。
        if (mConfig.cameraFacing == CameraCharacteristics.LENS_FACING_FRONT) {
            return mFrontCameraInfo;
        } else if (mConfig.cameraFacing == CameraCharacteristics.LENS_FACING_BACK) {
            return mBackCameraInfo;
        } else {
            throw new RuntimeException("No available camera id found.");
        }
    }

    private Size _getOptimalSize(int width, int height) {
        ///< 根据外层输入分辨率查找对应最合适的分辨率。
        List<Camera.Size> sizeMap = mCamera.getParameters().getSupportedPreviewSizes();
        List<Size> sizeList = new ArrayList<>();
        for (Camera.Size option:sizeMap) {
            if (width > height) {
                if (option.width >= width && option.height >= height) {
                    sizeList.add(new Size(option.width,option.height));
                }
            } else {
                if (option.width >= height && option.height >= width) {
                    sizeList.add(new Size(option.width,option.height));
                }
            }
        }
        if (sizeList.size() > 0) {
            return Collections.min(sizeList, new Comparator<Size>() {
                @Override
                public int compare(Size o1, Size o2) {
                    return Long.signum(o1.getWidth() * o1.getHeight() - o2.getWidth() * o2.getHeight());
                }
            });
        }

        return new Size(0,0);
    }

    private Range<Integer> _chooseFpsRange() {
        ///< 根据外层设置帧率查找最合适的帧率。
        List<int[]> fpsRange = mCamera.getParameters().getSupportedPreviewFpsRange();
        for (int[] range : fpsRange) {
            if (range.length == 2 && range[1] >= mConfig.fps*1000 && range[0] <= mConfig.fps*1000) {
                return new Range<>(range[0],range[1]);///< 仅支持列表中一项，不能像 camera2 一样指定
            }
        }

        return new Range<Integer>(0,0);
    }

    private void _initCameraInfo() {
        ///< 获取前置后置摄像头描述信息与 id。
        int numberOfCameras = Camera.getNumberOfCameras();
        for (int cameraId = 0; cameraId < numberOfCameras; cameraId++) {
            Camera.CameraInfo cameraInfo = new Camera.CameraInfo();
            Camera.getCameraInfo(cameraId, cameraInfo);
            if (cameraInfo.facing == Camera.CameraInfo.CAMERA_FACING_BACK) {
                // 后置摄像头信息。
                mBackCameraId = cameraId;
                mBackCameraInfo = cameraInfo;
            } else if (cameraInfo.facing == Camera.CameraInfo.CAMERA_FACING_FRONT) {
                // 前置摄像头信息。
                mFrontCameraId = cameraId;
                mFrontCameraInfo = cameraInfo;
            }
        }
    }

    private boolean _checkCameraService() {
        ///< 检测相机是否可用。
        DevicePolicyManager dpm = (DevicePolicyManager)mContext.get().getSystemService(Context.DEVICE_POLICY_SERVICE);
        if (dpm.getCameraDisabled(null)) {
            return false;
        }
        return true;
    }

    private void _callBackError(int error, String errorMsg) {
        ///< 错误回调。
        if (mListener != null) {
            mMainHandler.post(()->{
                mListener.cameraOnError(error,TAG + errorMsg);
            });
        }
    }

    private KFSurfaceTextureListener mSurfaceTextureListener = new KFSurfaceTextureListener() {
        @Override
        ///< SurfaceTexture 数据回调。
        public void onFrameAvailable(SurfaceTexture surfaceTexture) {
            mRenderHandler.post(()->{
                long timestamp = System.nanoTime();
                mGLContext.bind();
                ///< 刷新纹理数据至 SurfaceTexture。
                mSurfaceTexture.getSurfaceTexture().updateTexImage();
                if (mListener != null) {
                    ///< 拼装好纹理数据返回给外层。
                    KFTextureFrame frame = new KFTextureFrame(mSurfaceTexture.getSurfaceTextureId(),mConfig.resolution,timestamp,true);
                    mSurfaceTexture.getSurfaceTexture().getTransformMatrix(frame.textureMatrix);
                    mListener.onFrameAvailable(frame);
                }
                mGLContext.unbind();
            });
        }
    };
}
```

上面是 `KFVideoCaptureV1` 的实现，实现了 `KFIVideoCapture` 接口，结合下面这张图可以让我们更好地理解这些代码：

![相机流程图](assets/resource/av-demo/av-demo-android-video-capture1.png)
_相机流程图_

可以看到在实现采集时，我们是用 `mCamera` 来管理相关接口，通过它控制视频采集开始、结束、设置参数、设置输出目标等。重点介绍一下设置输出目标 `setPreviewTexture`，采集器内部自己创建一个 [SurfaceTexture](https://developer.android.com/reference/android/graphics/SurfaceTexture "SurfaceTexture")，将其提供给采集器，这样的优点是后续处理特效、编码等其它操作比较灵活。

从代码上可以看到主要有这几个部分：

- 1）初始化接口 `setup`。
	- 初始化采集线程、渲染线程，子线程处理的好处是有效避免主线程卡顿。
	- 初始化 OpenGL 上下文，将数据输出到自定义纹理上 `mSurfaceTexture`。
- 2）创建采集设备与开启预览 `startRunning`。
	- 检测视频采集权限 `checkSelfPermission`。
	- 检测摄像头是否可用，`_checkCameraService`。
	- 开启预览 `_startRunning`，首先获取前后台摄像机信息 `mFrontCameraInfo`、`mFrontCameraId`、`mBackCameraInfo`、`mBackCameraId`，通过 CameraId 打开摄像头，后续依次设置分辨率、帧率、方向、输出目标等参数。设置好后通过 `startPreview` 开启预览，数据则会自动同步到 `mSurfaceTexture`。
- 3）数据输出回调在 `KFSurfaceTextureListener` 的 `onFrameAvailable`。
	- 通过 `mSurfaceTexture` 的 `updateTexImage` 将数据转换为自定义纹理。
	- 将纹理数据与纹理矩阵封装为 `KFTextureFrame` 通过回调 `onFrameAvailable` 输出给外层。
- 4）实现切换摄像头的功能。
	- 在 `switchCamera` 中实现，一共分三步，停止之前摄像头、修改摄像头标记位、开启新的摄像头。
- 5）停止视频采集 `stopRunning`。
	- 设置回调为空 `setPreviewCallback`。
	- 停止预览 `stopPreview`。
	- 释放摄像头 `mCamera.release`。
- 6）清理摄像机实例 `release`。

更具体细节见上述代码及其注释。


## 2、视频采集模块 Camera2

接口类 `KFIVideoCapture` 与配置类 `KFVideoCaptureConfig` 与上面一致，这里不再介绍，我们直接分析 `KFVideoCaptureV2`，我们实现 2 套采集是因为 `Camera2` 功能更加强大（例如可以获取每帧的信息）以及性能更加高效，但它兼容性还不是很好，所以可以根据黑白名单或者跑分等策略选择合适的采集器。

`KFVideoCaptureV2.java`

```java
public class KFVideoCaptureV2 implements KFIVideoCapture {
    public static final int KFVideoCaptureV2CameraDisableError = -3000;
    private static final String TAG = "KFVideoCaptureV2";
    private KFVideoCaptureListener mListener = null; ///< 回调。
    private KFVideoCaptureConfig mConfig = null; ///< 采集配置。
    private WeakReference<Context> mContext = null;

    private CameraManager mCameraManager = null; ///< 相机系统服务，用于管理和连接相机设备。
    private String mCameraId; ///<摄像头 id。
    private CameraDevice mCameraDevice = null; ///< 相机设备类。
    private HandlerThread mCameraThread = null; ///< 采集线程。
    private Handler mCameraHandler = null;
    private CaptureRequest.Builder mCaptureRequestBuilder = null; ///< CaptureRequest 的构造器，使用 Builder 模式，设置更加方便。
    private CaptureRequest mCaptureRequest = null; ///< 相机捕获图像的设置请求，包含传感器，镜头，闪光灯等。
    private CameraCaptureSession mCameraCaptureSession = null; ///< 请求抓取相机图像帧的会话，会话的建立主要会建立起一个通道,源端是相机，另一端是 Target。
    private boolean mCameraIsRunning = false;
    private Range<Integer>[] mFpsRange;

    private KFGLContext mGLContext = null;
    private KFSurfaceTexture mSurfaceTexture = null;
    private Surface mSurface = null;
    private HandlerThread mRenderThread = null;
    private Handler mRenderHandler = null;
    private Handler mMainHandler = new Handler(Looper.getMainLooper());

    public KFVideoCaptureV2() {

    }

    @Override
    public void setup(Context context, KFVideoCaptureConfig config, KFVideoCaptureListener listener, EGLContext eglShareContext) {
        mListener = listener;
        mConfig = config;
        mContext = new WeakReference<Context>(context);

        ///< 相机采集线程。
        mCameraThread = new HandlerThread("KFCameraThread");
        mCameraThread.start();
        mCameraHandler = new Handler((mCameraThread.getLooper()));

        ///< 渲染线程。
        mRenderThread = new HandlerThread("KFCameraRenderThread");
        mRenderThread.start();
        mRenderHandler = new Handler((mRenderThread.getLooper()));

        mGLContext = new KFGLContext(eglShareContext);
    }

    @Override
    public EGLContext getEGLContext() {
        return mGLContext.getContext();
    }

    @Override
    public boolean isRunning() {
        return mCameraIsRunning;
    }

    @Override
    public void startRunning() {
        ///< 开启预览。
        mCameraHandler.post(() -> {
            _startRunning();
        });
    }

    @Override
    public void stopRunning() {
        ///< 停止预览。
        mCameraHandler.post(() -> {
            _stopRunning();
        });
    }

    @Override
    public void release() {
        mCameraHandler.post(() -> {
            ///< 关闭采集、释放 SurfaceTexture、OpenGL 上下文、线程等。
            _stopRunning();
            mGLContext.bind();
            if (mSurfaceTexture != null) {
                mSurfaceTexture.release();
                mSurfaceTexture = null;
            }

            mGLContext.unbind();
            mGLContext.release();
            mGLContext = null;

            if (mSurface != null) {
                mSurface.release();
                mSurface = null;
            }

            mCameraThread.quit();
            mRenderThread.quit();
        });
    }

    @Override
    public void switchCamera() {
        ///< 切换摄像头。
        mCameraHandler.post(() -> {
            _stopRunning();
            mConfig.cameraFacing = mConfig.cameraFacing == CameraCharacteristics.LENS_FACING_FRONT ? CameraCharacteristics.LENS_FACING_BACK : CameraCharacteristics.LENS_FACING_FRONT;
            _startRunning();
        });
    }

    private void _startRunning() {
        ///< 获取相机系统服务。
        if (mCameraManager == null) {
            mCameraManager = (CameraManager) mContext.get().getSystemService(Context.CAMERA_SERVICE);
        }
        ///< 根据外层摄像头方向查找摄像头 id。
        boolean selectSuccess = _chooseCamera();
        if (selectSuccess) {
            try {
                ///< 检测采集权限。
                if (ActivityCompat.checkSelfPermission(mContext.get(), Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
                    ActivityCompat.requestPermissions((Activity) mContext.get(), new String[] {Manifest.permission.CAMERA}, 1);
                }

                ///< 检测相机是否可用。
                if (!_checkCameraService()) {
                    _callBackError(KFVideoCaptureV2CameraDisableError,"相机不可用");
                    return;
                }

                ///< 打开相机设备。
                mCameraManager.openCamera(mCameraId, mStateCallback, mCameraHandler);
            } catch (CameraAccessException e) {
                e.printStackTrace();
            }
        }
    }

    private void _stopRunning() {
        ///< 停止采集。
        if (mCameraCaptureSession != null) {
            mCameraCaptureSession.close();
            mCameraCaptureSession = null;
        }

        if (mCameraDevice != null) {
            mCameraDevice.close();
            mCameraDevice = null;
        }
    }

    private KFSurfaceTextureListener mSurfaceTextureListener = new KFSurfaceTextureListener() {
        @Override
        //< SurfaceTexture 数据回调。
        public void onFrameAvailable(SurfaceTexture surfaceTexture) {
            mRenderHandler.post(() -> {
                long timestamp = System.nanoTime();
                mGLContext.bind();
                ///< 刷新纹理数据至 SurfaceTexture。
                mSurfaceTexture.getSurfaceTexture().updateTexImage();
                if (mListener != null) {
                    ///< 拼装好纹理数据返回给外层。
                    KFTextureFrame frame = new KFTextureFrame(mSurfaceTexture.getSurfaceTextureId(),mConfig.resolution,timestamp,true);
                    mSurfaceTexture.getSurfaceTexture().getTransformMatrix(frame.textureMatrix);
                    mListener.onFrameAvailable(frame);
                }
                mGLContext.unbind();
            });
        }
    };

    private CameraCaptureSession.StateCallback mCaputreSessionCallback = new CameraCaptureSession.StateCallback() {
        @Override
        ///< 创建会话回调。
        public void onConfigured(@NonNull CameraCaptureSession cameraCaptureSession) {
            ///< 创建CaptureRequest。
            mCaptureRequest = mCaptureRequestBuilder.build();
            mCameraCaptureSession = cameraCaptureSession;
            try {
                ///< 通过连续重复的 Capture 实现预览功能，每次 Capture 会把预览画面显示到对应的 Surface 上。
                mCameraCaptureSession.setRepeatingRequest(mCaptureRequest, null, null);
            } catch (CameraAccessException e) {
                e.printStackTrace();
            }
        }

        @Override
        ///< 创建会话出错回调。
        public void onConfigureFailed(@NonNull CameraCaptureSession cameraCaptureSession) {
            _callBackError(1005,"onConfigureFailed");
        }
    };

    private CameraDevice.StateCallback mStateCallback = new CameraDevice.StateCallback() {
        @Override
        ///< 相机打开回调。
        public void onOpened(@NonNull CameraDevice camera) {
            mCameraDevice = camera;
            try {
                ///< 通过相机设备创建构造器。
                mCaptureRequestBuilder = mCameraDevice.createCaptureRequest(CameraDevice.TEMPLATE_PREVIEW);
                Range<Integer> selectFpsRange = _chooseFpsRange();
                ///< 设置帧率。
                if (selectFpsRange.getUpper() > 0) {
                    mCaptureRequestBuilder.set(CaptureRequest.CONTROL_AE_TARGET_FPS_RANGE,selectFpsRange);
                }
            } catch (CameraAccessException e) {
                e.printStackTrace();
            }

            if (mListener != null) {
                mMainHandler.post(()->{
                    mListener.cameraOnOpened();
                });
            }
            mCameraIsRunning = true;

            if (mSurfaceTexture == null) {
                mGLContext.bind();
                mSurfaceTexture = new KFSurfaceTexture(mSurfaceTextureListener);
                mGLContext.unbind();
                mSurface = new Surface(mSurfaceTexture.getSurfaceTexture());
            }

            if (mSurface != null) {
                ///< 设置目标输出 Surface。
                mSurfaceTexture.getSurfaceTexture().setDefaultBufferSize(mConfig.resolution.getHeight(),mConfig.resolution.getWidth());
                mCaptureRequestBuilder.addTarget(mSurface);
                try {
                    ///< 创建通道会话。
                    mCameraDevice.createCaptureSession(Arrays.asList(mSurface), mCaputreSessionCallback, mCameraHandler);
                } catch (CameraAccessException e) {
                    e.printStackTrace();
                }
            }
        }

        @Override
        public void onDisconnected(@NonNull CameraDevice camera) {
            ///< 相机断开连接回调。
            camera.close();
            mCameraDevice = null;
            mCameraIsRunning = false;
        }

        @Override
        public void onClosed(@NonNull CameraDevice camera) {
            ///< 相机关闭回调。
            camera.close();
            mCameraDevice = null;
            if (mListener != null) {
                mMainHandler.post(()->{
                    mListener.cameraOnClosed();
                });
            }
            mCameraIsRunning = false;
        }

        @Override
        public void onError(@NonNull CameraDevice camera, int error) {
            ///< 相机出错回调。
            camera.close();
            mCameraDevice = null;
            _callBackError(error,"Camera onError");
            mCameraIsRunning = false;
        }
    };

    private boolean _chooseCamera() {
        try {
            ///< 根据外层配置方向选择合适的设备 id 与 FPS 区间。
            final String[] ids = mCameraManager.getCameraIdList();
            for (String cameraId : ids) {
                CameraCharacteristics characteristics = mCameraManager.getCameraCharacteristics(cameraId);
                Integer facing = characteristics.get(CameraCharacteristics.LENS_FACING);
                if (facing == mConfig.cameraFacing) {
                    mCameraId = cameraId;
                    mFpsRange = characteristics.get(CameraCharacteristics.CONTROL_AE_AVAILABLE_TARGET_FPS_RANGES);
                    StreamConfigurationMap map = characteristics.get(CameraCharacteristics.SCALER_STREAM_CONFIGURATION_MAP);
                    if (map != null) {
                        Size previewSize = _getOptimalSize(map.getOutputSizes(SurfaceTexture.class), mConfig.resolution.getWidth(), mConfig.resolution.getHeight());
                        // Range<Integer>[] fpsRanges = map.getHighSpeedVideoFpsRangesFor(previewSize); ///< high fps range
                        mConfig.resolution = new Size(previewSize.getHeight(),previewSize.getWidth());
                    }
                    return true;
                }
            }
        } catch (CameraAccessException e) {
            e.printStackTrace();
        }

        return false;
    }

    private Size _getOptimalSize(Size[] sizeMap, int width, int height) {
        ///< 根据外层配置分辨率寻找合适的分辨率。
        List<Size> sizeList = new ArrayList<>();
        for (Size option : sizeMap) {
            if (width > height) {
                if (option.getWidth() >= width && option.getHeight() >= height) {
                    sizeList.add(option);
                }
            } else {
                if (option.getWidth() >= height && option.getHeight() >= width) {
                    sizeList.add(option);
                }
            }
        }
        if (sizeList.size() > 0) {
            return Collections.min(sizeList, new Comparator<Size>() {
                @Override
                public int compare(Size o1, Size o2) {
                    return Long.signum(o1.getWidth() * o1.getHeight() - o2.getWidth() * o2.getHeight());
                }
            });
        }
        return sizeMap[0];
    }

    private boolean _checkCameraService() {
        ///< 检测相机是否可用。
        DevicePolicyManager dpm = (DevicePolicyManager)mContext.get().getSystemService(Context.DEVICE_POLICY_SERVICE);
        if (dpm.getCameraDisabled(null)) {
            return false;
        }
        return true;
    }

    private void _callBackError(int error, String errorMsg) {
        ///< 错误回调。
        if (mListener != null) {
            mMainHandler.post(()->{
                mListener.cameraOnError(error,TAG + errorMsg);
            });
        }
    }

    private Range<Integer> _chooseFpsRange() {
        ///< 根据外层配置的帧率寻找合适的帧率。
        for (Range<Integer> range : mFpsRange) {
            if (range.getUpper() >= mConfig.fps && range.getLower() <= mConfig.fps) {
                return new Range<>(range.getLower(),mConfig.fps);
            }
        }

        return new Range<Integer>(0,0);
    }
}
```

上面是 `KFVideoCaptureV2` 的实现，实现了 `KFIVideoCapture` 接口，结合下面这张图可以让我们更好地理解这些代码：

![相机流程图](assets/resource/av-demo/av-demo-android-video-capture2.png)
_相机流程图_

从代码上可以看到与 `Camera` 区别如下：

- 1）开启预览 `_startRunning`，流程如下。
	- 获取相机系统服务 `mCameraManager`，根据外层配置方向选择合适的设备id `mCameraId`。
	- 打开相机设备 `openCamera`，通过回调 `mStateCallback` 监控相机状态。
	- 相机设备打开成功会执行 `onOpened`，创建 CaptureRequest 构造器 `mCaptureRequestBuilder`，通过构造器设置参数帧率，连接输出对象 `mSurface`，`mSurface` 根据 `mSurfaceTexture` 生成。通过 `mSurface` 、 `mCaputreSessionCallback` 回调创建图像帧会话 `createCaptureSession`。
	- 图像帧会话打开成功会执行 `onConfigured`，通过连续重复的 Capture 实现预览功能，每次 Capture 会把预览画面显示到对应的 Surface 上。

更具体细节见上述代码及其注释。

## 3、采集视频并实时展示

我们在一个 `MainActivity` 中来实现视频采集并实时预览的逻辑。


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
- 3）采集数据回调 `onFrameAvailable`，将数据输入给渲染视图进行预览，预览后续会介绍，如果希望将数据存储可以借助 [ImageReader](https://developer.android.com/reference/android/media/ImageReader "ImageReader")。



更具体细节见上述代码及其注释。















---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
{: .prompt-tip }

