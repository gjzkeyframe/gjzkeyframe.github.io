---
title: RenderDemo（4）：用 OpenGL 实现反色
description: 介绍用 OpenGL 实现反色的流程和原理，并提供 Demo 源码和解析。
author: Keyframe
date: 2025-02-23 13:38:08 +0800
categories: [音视频源码示例]
tags: [音视频源码示例, 音视频, iOS, Android, OpenGL, 渲染]
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

这里是 RenderDemo 的第四篇：**用 OpenGL 实现反色**。我们分别在 iOS 和 Android 平台实现了用 OpenGL 对图像进行反色处理并渲染出来。

![反色处理图片效果](assets/resource/av-demo/render-demo-reverse.png)
_反色处理图片效果_

到目前我们已经在我们的付费知识星球中提供了下面这些音视频 Demo 和渲染 Demo 的工程源码，均可直接下载运行：

- iOS AVDemo（1）：音频采集
- iOS AVDemo（2）：音频编码
- iOS AVDemo（3）：音频封装
- iOS AVDemo（4）：音频解封装
- iOS AVDemo（5）：音频解码
- iOS AVDemo（6）：音频渲染
- iOS AVDemo（7）：视频采集
- iOS AVDemo（8）：视频编码
- iOS AVDemo（9）：视频封装
- iOS AVDemo（10）：视频解封装
- iOS AVDemo（11）：视频转封装
- iOS AVDemo（12）：视频编码
- iOS AVDemo（13）：视频渲染
- Android AVDemo（1）：音频采集
- Android AVDemo（2）：音频编码
- Android AVDemo（3）：音频封装
- Android AVDemo（4）：音频解封装
- Android AVDemo（5）：音频解码
- Android AVDemo（6）：音频渲染
- Android AVDemo（7）：视频采集
- Android AVDemo（8）：视频编码
- Android AVDemo（9）：视频封装
- Android AVDemo（10）：视频解封装
- Android AVDemo（11）：视频转封装
- Android AVDemo（12）：视频解码
- Android AVDemo（13）：视频渲染
- RenderDemo（1）：用 OpenGL 画一个三角形（iOS+Android）
- RenderDemo（2）：用 OpenGL 渲染视频（iOS+Android）
- RenderDemo（3）：用 OpenGL 实现高斯模糊（iOS+Android）

这些源码对于学习和理解 iOS/Android 音视频开发非常容易上手，有需要的朋友可以扫描下面二维码加入星球获取全部源码：

![微信扫码二维码加入我们](assets/img/keyframe-zsxq.png)



---

反色是一种像素颜色取反的图像效果，本文将会给大家介绍用 OpenGL 完成反色的代码实现。

## 1、反色基础知识

图片的颜色反转，原理就是用白色的 rgb 值减去当前图片颜色 rgb 值，得到后的效果就是反转后的颜色。

### 1.1、Shader 实现
```c
 precision highp float;
 varying highp vec2 textureCoordinate;
 uniform sampler2D inputImageTexture;
 
 void main()
 {
    vec4 color = texture2D(inputImageTexture, textureCoordinate);
    gl_FragColor = vec4(1.0 - color.r,1.0 - color.g,1.0 - color.b,color.a);
 }
```

### 1.2、效果对比

![反色效果对比](assets/resource/av-demo/render-demo-reverse-diff.gif)
_反色效果对比_


## 2、iOS Demo

### 2.1、渲染模块

渲染模块与 [OpenGL 高斯模糊](https://mp.weixin.qq.com/s/ibo_2HSib2KPLm0gvAFVow) 中讲到的一致，最终是封装出一个渲染视图 `KFOpenGLView` 用于展示最后的渲染结果。这里就不再细讲，只贴一下主要的类和类具体的功能：

- `KFOpenGLView`：使用 OpenGL 实现的渲染 View，提供了设置画面填充模式的接口和渲染一帧纹理的接口。
- `KFGLFilter`：实现 shader 的加载、编译和着色器程序链接，以及 FBO 的管理。同时作为渲染处理节点，提供给了接口支持多级渲染。
- `KFGLProgram`：封装了使用 GL 程序的部分 API。
- `KFGLFrameBuffer`：封装了使用 FBO 的 API。
- `KFTextureFrame`：表示一帧纹理对象。
- `KFFrame`：表示一帧，类型可以是数据缓冲或纹理。
- `KFGLTextureAttributes`：对纹理 Texture 属性的封装。
- `KFGLBase`：定义了默认的 VertexShader 和 FragmentShader。
- `KFUIImageConvertTexture `：用于实现图片转纹理。

### 2.2、反色渲染结果渲染流程

我们在一个 `ViewController` 中分别实现了图片与视频采集 2 种实现方式。代码如下：


**1) 图片渲染模式 KFImageRenderViewController**

```objc
@interface KFImageRenderViewController ()
@property (nonatomic, strong) KFOpenGLView *glView;
@property (nonatomic, strong) KFPixelBufferConvertTexture *pixelBufferConvertTexture;
@property (nonatomic, strong) EAGLContext *context;
@property (nonatomic, strong) KFGLFilter *filter;
@end

@implementation KFImageRenderViewController
#pragma mark - Property

- (EAGLContext *)context {
    if (!_context) {
        _context = [[EAGLContext alloc] initWithAPI:kEAGLRenderingAPIOpenGLES2];
    }
    
    return _context;
}

- (KFPixelBufferConvertTexture *)pixelBufferConvertTexture {
    if (!_pixelBufferConvertTexture) {
        _pixelBufferConvertTexture = [[KFPixelBufferConvertTexture alloc] initWithContext:self.context];
    }
    
    return _pixelBufferConvertTexture;
}

- (KFGLFilter*)filter {
    if(!_filter){
        _filter = [[KFGLFilter alloc] initWithCustomFBO:NO vertexShader:KFDefaultVertexShader fragmentShader:KFReverseColorFragmentShader];
    }
    return _filter;
}

#pragma mark - Lifecycle
- (void)viewDidLoad {
    [super viewDidLoad];

    [self setupUI];
    [self applyReverseEffect];
}

- (void)viewWillLayoutSubviews {
    [super viewWillLayoutSubviews];
    self.glView.frame = self.view.bounds;
}

- (void)setupUI {
    self.edgesForExtendedLayout = UIRectEdgeAll;
    self.extendedLayoutIncludesOpaqueBars = YES;
    self.title = @"image Render";
    self.view.backgroundColor = [UIColor whiteColor];
    
    
    // 渲染 view。
    _glView = [[KFOpenGLView alloc] initWithFrame:self.view.bounds context:self.context];
    _glView.fillMode = KFGLViewContentModeFit;
    [self.view addSubview:self.glView];
}

- (void)applyReverseEffect {
    [EAGLContext setCurrentContext:self.context];
    UIImage *baseImage = [UIImage imageNamed:@"KeyframeLogo"];
    KFTextureFrame *textureFrame = [KFUIImageConvertTexture renderImage:baseImage];
    
    KFTextureFrame *filterFrame = [self.filter render:textureFrame];
    
    [self.glView displayFrame:filterFrame];
    [EAGLContext setCurrentContext:nil];
}

@end
```

**2)  视频采集渲染模式 KFVideoRenderViewController**

```objc
@interface KFVideoRenderViewController ()
@property (nonatomic, strong) KFVideoCaptureConfig *videoCaptureConfig;
@property (nonatomic, strong) KFVideoCapture *videoCapture;
@property (nonatomic, strong) KFOpenGLView *glView;
@property (nonatomic, strong) KFPixelBufferConvertTexture *pixelBufferConvertTexture;
@property (nonatomic, strong) EAGLContext *context;
@property (nonatomic, strong) KFGLFilter *filter;
@end

@implementation KFVideoRenderViewController
#pragma mark - Property
- (KFVideoCaptureConfig *)videoCaptureConfig {
    if (!_videoCaptureConfig) {
        _videoCaptureConfig = [[KFVideoCaptureConfig alloc] init];
    }
    
    return _videoCaptureConfig;
}

- (EAGLContext *)context {
    if (!_context) {
        _context = [[EAGLContext alloc] initWithAPI:kEAGLRenderingAPIOpenGLES2];
    }
    
    return _context;
}

- (KFPixelBufferConvertTexture *)pixelBufferConvertTexture {
    if (!_pixelBufferConvertTexture) {
        _pixelBufferConvertTexture = [[KFPixelBufferConvertTexture alloc] initWithContext:self.context];
    }
    
    return _pixelBufferConvertTexture;
}

- (KFVideoCapture *)videoCapture {
    if (!_videoCapture) {
        _videoCapture = [[KFVideoCapture alloc] initWithConfig:self.videoCaptureConfig];
        __weak typeof(self) weakSelf = self;
        _videoCapture.sampleBufferOutputCallBack = ^(CMSampleBufferRef sampleBuffer) {
             // 视频采集数据回调。将采集回来的数据给渲染模块渲染。
            [EAGLContext setCurrentContext:weakSelf.context];
            KFTextureFrame *textureFrame = [weakSelf.pixelBufferConvertTexture renderFrame:CMSampleBufferGetImageBuffer(sampleBuffer) time:CMSampleBufferGetPresentationTimeStamp(sampleBuffer)];
            KFTextureFrame *filterFrame = [weakSelf.filter render:textureFrame];
            [weakSelf.glView displayFrame:filterFrame];
            [EAGLContext setCurrentContext:nil];
        };
        _videoCapture.sessionErrorCallBack = ^(NSError* error) {
            NSLog(@"KFVideoCapture Error:%zi %@", error.code, error.localizedDescription);
        };
    }
    
    return _videoCapture;
}

- (KFGLFilter*)filter {
    if(!_filter){
        _filter = [[KFGLFilter alloc] initWithCustomFBO:NO vertexShader:KFDefaultVertexShader fragmentShader:KFReverseColorFragmentShader];
    }
    return _filter;
}

#pragma mark - Lifecycle
- (void)viewDidLoad {
    [super viewDidLoad];

    [self requestAccessForVideo];
    [self setupUI];
}

- (void)viewWillLayoutSubviews {
    [super viewWillLayoutSubviews];
    self.glView.frame = self.view.bounds;
}

#pragma mark - Action
- (void)changeCamera {
    [self.videoCapture changeDevicePosition:self.videoCapture.config.position == AVCaptureDevicePositionBack ? AVCaptureDevicePositionFront : AVCaptureDevicePositionBack];
}

#pragma mark - Private Method
- (void)requestAccessForVideo {
    __weak typeof(self) weakSelf = self;
    AVAuthorizationStatus status = [AVCaptureDevice authorizationStatusForMediaType:AVMediaTypeVideo];
    switch (status) {
        case AVAuthorizationStatusNotDetermined:{
            // 许可对话没有出现，发起授权许可。
            [AVCaptureDevice requestAccessForMediaType:AVMediaTypeVideo completionHandler:^(BOOL granted) {
                if (granted) {
                    [weakSelf.videoCapture startRunning];
                } else {
                    // 用户拒绝。
                }
            }];
            break;
        }
        case AVAuthorizationStatusAuthorized:{
            // 已经开启授权，可继续。
            [weakSelf.videoCapture startRunning];
            break;
        }
        default:
            break;
    }
}

- (void)setupUI {
    self.edgesForExtendedLayout = UIRectEdgeAll;
    self.extendedLayoutIncludesOpaqueBars = YES;
    self.title = @"Video Render";
    self.view.backgroundColor = [UIColor whiteColor];
    
    // Navigation item.
    UIBarButtonItem *cameraBarButton = [[UIBarButtonItem alloc] initWithTitle:@"Camera" style:UIBarButtonItemStylePlain target:self action:@selector(changeCamera)];
    self.navigationItem.rightBarButtonItems = @[cameraBarButton];
    
    // 渲染 view。
    _glView = [[KFOpenGLView alloc] initWithFrame:self.view.bounds context:self.context];
    _glView.fillMode = KFGLViewContentModeFill;
    [self.view addSubview:self.glView];
}

@end
```

通过上面的代码，可以看到我们是用 `KFGLFilter` 来封装一次 OpenGL 的处理节点，它可以接收一个 `KFTextureFrame` 对象，加载 Shader 对其进行渲染处理，处理完后输出处理后的 `KFTextureFrame`，然后可以接着交给下一个 `KFGLFilter` 来处理，就像一条渲染链。

## 3、Android Demo

### 3.1、渲染模块

渲染模块与 [OpenGL 高斯模糊](https://mp.weixin.qq.com/s/ibo_2HSib2KPLm0gvAFVow) 中讲到的一致，最终是封装出一个渲染视图 `KFRenderView` 用于展示最后的渲染结果。这里就不再细讲，只贴一下主要的类和类具体的功能：

- `KFGLContext`：负责创建 OpenGL 环境，负责管理和组装 EGLDisplay、EGLSurface、EGLContext。
- `KFGLFilter`：实现 shader 的加载、编译和着色器程序链接，以及 FBO 的管理。同时作为渲染处理节点，提供给了接口支持多级渲染。
- `KFGLProgram`：负责加载和编译着色器，创建着色器程序容器。
- `KFGLBase`：定义了默认的 VertexShader 和 FragmentShader。
- `KFSurfaceView`：KFSurfaceView 继承自 SurfaceView 来实现渲染。
- `KFTextureView`：KFTextureView 继承自 TextureView 来实现渲染。
- `KFFrame`：表示一帧，类型可以是数据缓冲或纹理。
- `KFRenderView`：KFRenderView 是一个容器，可以选择使用 KFSurfaceView 或 KFTextureView 作为实际的渲染视图。

### 3.2、反色渲染结果渲染流程

我们在一个 `MainActivity` 中分别实现了图片与视频采集 2 种实现方式。代码如下：


**1) 图片渲染模式 KFImageRenderActivity**


```java
public class KFImageRenderActivity extends AppCompatActivity {
    public static String reverseFragmentShader =
            //
            "precision mediump float;\n" +
                    "uniform sampler2D inputImageTexture;\n" +
                    "varying vec2 textureCoordinate;\n" +
                    "void main() {\n" +
                    "  vec4 color = texture2D(inputImageTexture, textureCoordinate);\n" +
                    "  gl_FragColor = vec4(1.0-color.r,1.0-color.g,1.0-color.b,color.a);\n" +
                    "}\n";

    private KFRenderView mRenderView;
    private KFGLContext mGLContext;
    private KFGLFilter mGLFilter;
    private Bitmap mLogoBitmap;
    private int mLogoTexture;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_kfimage_render3);
        mLogoBitmap = getImageFromAssetsFile(this,"KeyframeLogo.jpg");
        mLogoBitmap = getTurnOverBitmap(mLogoBitmap);

        mGLContext = new KFGLContext(null);
        mRenderView = new KFRenderView(this, mGLContext.getContext(), new KFRenderListener() {
            @Override
            public void surfaceCreate(@NonNull Surface surface) {

            }

            @Override
            public void surfaceChanged(@NonNull Surface surface, int width, int height) {
                applyReverseEffect();
            }

            @Override
            public void surfaceDestroy(@NonNull Surface surface) {

            }
        });
        mRenderView.setFillMode(KFRenderView.KFRenderMode.KFRenderModeFit);
        WindowManager windowManager = (WindowManager)this.getSystemService(this.WINDOW_SERVICE);
        Rect outRect = new Rect();
        windowManager.getDefaultDisplay().getRectSize(outRect);
        FrameLayout.LayoutParams params = new FrameLayout.LayoutParams(outRect.width(), outRect.height());
        addContentView(mRenderView,params);
    }


    public void applyReverseEffect() {
        mGLContext.bind();
        if(mGLFilter == null){
            mGLFilter = new KFGLFilter(false, KFGLBase.defaultVertexShader,reverseFragmentShader);

            int[] textures = new int[1];
            glGenTextures(1, textures, 0);
            mLogoTexture = textures[0];
            glActiveTexture(GLES20.GL_TEXTURE0);
            glBindTexture(GLES20.GL_TEXTURE_2D, mLogoTexture);
            glTexParameteri(GLES20.GL_TEXTURE_2D, GLES20.GL_TEXTURE_MIN_FILTER, GLES20.GL_LINEAR);
            glTexParameteri(GLES20.GL_TEXTURE_2D, GLES20.GL_TEXTURE_MAG_FILTER, GLES20.GL_LINEAR);
            glTexParameteri(GLES20.GL_TEXTURE_2D, GLES20.GL_TEXTURE_WRAP_S, GLES20.GL_CLAMP_TO_EDGE);
            glTexParameteri(GLES20.GL_TEXTURE_2D, GLES20.GL_TEXTURE_WRAP_T, GLES20.GL_CLAMP_TO_EDGE);
            GLUtils.texImage2D(GLES20.GL_TEXTURE_2D, 0, mLogoBitmap, 0);
            mLogoBitmap.recycle();
        }
        KFTextureFrame frame = new KFTextureFrame(mLogoTexture,new Size(mLogoBitmap.getWidth(),mLogoBitmap.getHeight()),0);
        KFFrame filterFrame = mGLFilter.render((KFTextureFrame)frame);
        mRenderView.render((KFTextureFrame) filterFrame);
        mGLContext.unbind();
    }

    public Bitmap getTurnOverBitmap(Bitmap bitmap) {
        Canvas canvas = new Canvas();
        Bitmap output = Bitmap.createBitmap(bitmap.getWidth(),
                bitmap.getHeight(), Bitmap.Config.ARGB_8888);
        canvas.setBitmap(output);
        Matrix matrix = new Matrix();
        // 缩放 当sy为-1时向上翻转 当sx为-1时向左翻转 sx、sy都为-1时相当于旋转180°
        matrix.postScale(1, -1);
        // 因为向上翻转了所以y要向下平移一个bitmap的高度
        matrix.postTranslate(0, bitmap.getHeight());
        canvas.drawBitmap(bitmap, matrix, null);
        return output;
    }

    private Bitmap getImageFromAssetsFile(Context context, String fileName) {
        Bitmap image = null;
        AssetManager am = context.getResources().getAssets();
        try {
            InputStream is = am.open(fileName);
            image = BitmapFactory.decodeStream(is);
            is.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return image;
    }
}
```

**2)  视频采集渲染模式 KFVideoRenderActivity**

```java
public class KFVideoRenderActivity extends AppCompatActivity {
    public static String reverseFragmentShader =
            //
            "precision mediump float;\n" +
                    "uniform sampler2D inputImageTexture;\n" +
                    "varying vec2 textureCoordinate;\n" +
                    "void main() {\n" +
                    "  vec4 color = texture2D(inputImageTexture, textureCoordinate);\n" +
                    "  gl_FragColor = vec4(1.0-color.r,1.0-color.g,1.0-color.b,color.a);\n" +
                    "}\n";

    private KFIVideoCapture mCapture;
    private KFVideoCaptureConfig mCaptureConfig;
    private KFRenderView mRenderView;
    private KFGLContext mGLContext;
    private KFGLFilter mGLFilter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_kfvideo_render);

        if (ActivityCompat.checkSelfPermission(this, Manifest.permission.RECORD_AUDIO) != PackageManager.PERMISSION_GRANTED || ActivityCompat.checkSelfPermission(this, Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED ||
                ActivityCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED ||
                ActivityCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions((Activity) this,
                    new String[] {Manifest.permission.CAMERA,Manifest.permission.RECORD_AUDIO, Manifest.permission.READ_EXTERNAL_STORAGE, Manifest.permission.WRITE_EXTERNAL_STORAGE},
                    1);
        }

        mGLContext = new KFGLContext(null);
        mRenderView = new KFRenderView(this,mGLContext.getContext());
        WindowManager windowManager = (WindowManager)this.getSystemService(this.WINDOW_SERVICE);
        Rect outRect = new Rect();
        windowManager.getDefaultDisplay().getRectSize(outRect);
        FrameLayout.LayoutParams params = new FrameLayout.LayoutParams(outRect.width(), outRect.height());
        addContentView(mRenderView,params);

        mCaptureConfig = new KFVideoCaptureConfig();
        mCaptureConfig.cameraFacing = LENS_FACING_FRONT;
        mCaptureConfig.resolution = new Size(720,1280);
        mCaptureConfig.fps = 30;
        boolean useCamera2 = false;
        if(useCamera2){
            mCapture = new KFVideoCaptureV2();
        }else{
            mCapture = new KFVideoCaptureV1();
        }
        mCapture.setup(this,mCaptureConfig,mVideoCaptureListener,mGLContext.getContext());
        mCapture.startRunning();
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
            mGLContext.bind();
            if(mGLFilter == null){
                mGLFilter = new KFGLFilter(false, KFGLBase.defaultVertexShader,reverseFragmentShader);
            }
            KFFrame filterFrame = mGLFilter.render((KFTextureFrame)frame);
            mRenderView.render((KFTextureFrame) filterFrame);
            mGLContext.unbind();
        }
    };
}
```

可见，当我们用 `KFGLFilter` 将 OpenGL 渲染能力封装起来，并可以像增加渲染处理节点一样往现有渲染链中增加新的图像处理功能时，相关改动就变得很方便了。




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

