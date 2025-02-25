---
title: RenderDemo（6）：用 OpenGL 实现贴纸
description: 介绍用 OpenGL 实现贴纸的流程和原理，并提供 Demo 源码和解析。
author: Keyframe
date: 2025-02-23 13:58:08 +0800
categories: [音视频源码示例]
tags: [音视频源码示例, 音视频, iOS, Android, OpenGL, 渲染]
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

这里是 RenderDemo 的第六篇：**用 OpenGL 实现贴纸**。我们分别在 iOS 和 Android 平台实现了用 OpenGL 对图像进行贴纸处理并渲染出来。

![贴纸处理图片效果](assets/resource/av-demo/render-demo-sticker.png)
_贴纸处理图片效果_

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
- RenderDemo（4）：用 OpenGL 实现反色（iOS+Android）
- RenderDemo（5）：用 OpenGL 实现三分屏（iOS+Android）

这些源码对于学习和理解 iOS/Android 音视频开发非常容易上手，有需要的朋友可以扫描下面二维码加入星球获取全部源码：

![微信扫码二维码加入我们](assets/img/keyframe-zsxq.png)



---

贴纸是一种将源图像与贴纸图像进行混合的效果，本文将会给大家介绍用 OpenGL 完成贴纸的代码实现。

## 1、贴纸基础知识

贴纸特效核心原理就是图像混合，公式 = 贴纸图像颜色 * 贴纸图像透明度 + 目标图像颜色 * (1.0 - 贴纸图像透明度)，但因为系统解码通常图片已经进行过颜色预乘，所以贴纸图像颜色不需要乘以透明度。

### 1.1、Shader 实现
```c
precision highp float;

varying vec2 textureCoordinate;
uniform sampler2D inputImageTexture;
uniform sampler2D stickerTexture;

void main()
{
    vec4 sourceColor = texture2D(inputImageTexture, textureCoordinate);
    vec4 stickerColor = texture2D(stickerTexture, textureCoordinate);
    
    float r = stickerColor.r + (1.0 - stickerColor.a) * sourceColor.r;
    float g = stickerColor.g + (1.0 - stickerColor.a) * sourceColor.g;
    float b = stickerColor.b + (1.0 - stickerColor.a) * sourceColor.b;
    
    gl_FragColor = vec4(r, g, b, sourceColor.a);
}
```

### 1.2、效果对比

![贴纸效果对比](assets/resource/av-demo/render-demo-sticker-diff.gif)
_贴纸效果对比_


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

### 2.2、贴纸渲染结果渲染流程

我们在一个 `ViewController` 中分别实现了图片与视频采集 2 种实现方式。代码如下：


**1)  图片渲染模式 KFImageRenderViewController**

```objc
@interface KFImageRenderViewController (){
    GLuint _texture;
}
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

- (void)dealloc {
    EAGLContext *preContext = [EAGLContext currentContext];
    [EAGLContext setCurrentContext:self.context];
    if(_texture){
        glDeleteTextures(1, &_texture);
    }
    [EAGLContext setCurrentContext:preContext];
}

- (KFPixelBufferConvertTexture *)pixelBufferConvertTexture {
    if (!_pixelBufferConvertTexture) {
        _pixelBufferConvertTexture = [[KFPixelBufferConvertTexture alloc] initWithContext:self.context];
    }
    
    return _pixelBufferConvertTexture;
}

- (KFGLFilter*)filter {
    if(!_filter){
        NSString *path = [[NSBundle mainBundle] pathForResource:@"filter" ofType:@"fs"];
        _filter = [[KFGLFilter alloc] initWithCustomFBO:NO vertexShader:KFDefaultVertexShader fragmentShader:[NSString stringWithContentsOfURL:[NSURL fileURLWithPath:path] encoding:NSUTF8StringEncoding error:nil]];
        __weak typeof(self) _self = self;
        _filter.preDrawCallBack = ^(){
            __strong typeof(_self) sself = _self;
            if(sself){
                glActiveTexture(GL_TEXTURE5);
                glBindTexture(GL_TEXTURE_2D, sself->_texture);
                glUniform1i([sself->_filter.getProgram getUniformLocation:@"stickerTexture"], 5);
            }
        };
    }
    return _filter;
}

#pragma mark - Lifecycle
- (void)viewDidLoad {
    [super viewDidLoad];

    _texture = -1;
    [self setupUI];
    [self applyEffect];
}

- (void)viewDidAppear:(BOOL)animated {
    [super viewDidAppear:animated];
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

- (void)applyEffect {
    [EAGLContext setCurrentContext:self.context];
    UIImage *baseImage = [UIImage imageNamed:@"KeyframeLogo"];
    KFTextureFrame *textureFrame = [KFUIImageConvertTexture renderImage:baseImage];
    if(self->_texture == -1){
        self->_texture = [KFGLBase newTextureForImage:[self.class stickerImage:CGSizeMake(272, 272)]];
    }
    
    
    KFTextureFrame *filterFrame = [self.filter render:textureFrame];
    
    [self.glView displayFrame:filterFrame];
    [EAGLContext setCurrentContext:nil];
}

+ (UIImage *)stickerImage:(CGSize)size
{
    UIGraphicsBeginImageContextWithOptions(size, NO, [UIScreen mainScreen].scale);
    
    UIImage *logoImage = [UIImage imageNamed:@"sticker.jpg"];
    [logoImage drawInRect:CGRectMake(272 - 50 - 20, 272 - 50 - 20, 50, 50)];
    
    UIImage *image= UIGraphicsGetImageFromCurrentImageContext();
    
    UIGraphicsEndImageContext();
    
    return image;
}

@end
```

**2) 视频采集渲染模式 KFVideoRenderViewController**

```objc
@interface KFVideoRenderViewController (){
    GLuint _texture;
}
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
        NSString *path = [[NSBundle mainBundle] pathForResource:@"filter" ofType:@"fs"];
        _filter = [[KFGLFilter alloc] initWithCustomFBO:NO vertexShader:KFDefaultVertexShader fragmentShader:[NSString stringWithContentsOfURL:[NSURL fileURLWithPath:path] encoding:NSUTF8StringEncoding error:nil]];
        __weak typeof(self) _self = self;
        _filter.preDrawCallBack = ^(){
            __strong typeof(_self) sself = _self;
            if(sself){
                if(sself->_texture == -1){
                    sself->_texture = [KFGLBase newTextureForImage:[sself.class stickerImage:CGSizeMake(720, 1280)]];
                }
                
                glActiveTexture(GL_TEXTURE5);
                glBindTexture(GL_TEXTURE_2D, sself->_texture);
                glUniform1i([sself->_filter.getProgram getUniformLocation:@"stickerTexture"], 5);
            }
        };
    }
    return _filter;
}

#pragma mark - Lifecycle
- (void)viewDidLoad {
    [super viewDidLoad];

    _texture = -1;
    [self requestAccessForVideo];
    [self setupUI];
}

- (void)viewWillLayoutSubviews {
    [super viewWillLayoutSubviews];
    self.glView.frame = self.view.bounds;
}

- (void)dealloc {
    EAGLContext *preContext = [EAGLContext currentContext];
    [EAGLContext setCurrentContext:self.context];
    if(_texture){
        glDeleteTextures(1, &_texture);
    }
    [EAGLContext setCurrentContext:preContext];
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

+ (UIImage *)stickerImage:(CGSize)size
{
    UIGraphicsBeginImageContextWithOptions(size, NO, [UIScreen mainScreen].scale);
    
    UIImage *logoImage = [UIImage imageNamed:@"sticker.jpg"];
    [logoImage drawInRect:CGRectMake(720 - 100 - 20, 1280 - 100 - 20, 100, 100)];
    
    UIImage *image= UIGraphicsGetImageFromCurrentImageContext();
    
    UIGraphicsEndImageContext();
    
    return image;
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

### 3.2、贴纸渲染结果渲染流程

我们在一个 `MainActivity` 中分别实现了图片与视频采集 2 种实现方式。代码如下：


**1)  图片渲染模式 KFImageRenderActivity**


```java
public class KFImageRenderActivity extends AppCompatActivity {
    private KFRenderView mRenderView;
    private KFGLContext mGLContext;
    private KFGLFilter mGLFilter;
    private Bitmap mStickerBitmap;
    private int mStickerTexture;
    private Bitmap mLogoBitmap;
    private int mLogoTexture;
    private String mFragmentShaderString;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_kfimage_render3);

        mLogoBitmap = getImageFromAssetsFile(this,"KeyframeLogo.jpg");
        mLogoBitmap = getTurnOverBitmap(mLogoBitmap);

        mStickerBitmap = stickerBitmap();
        mStickerBitmap = getTurnOverBitmap(mStickerBitmap);
        mFragmentShaderString = getShaderString(this,"filter.fs");

        mGLContext = new KFGLContext(null);
        mRenderView = new KFRenderView(this, mGLContext.getContext(), new KFRenderListener() {
            @Override
            public void surfaceCreate(@NonNull Surface surface) {

            }

            @Override
            public void surfaceChanged(@NonNull Surface surface, int width, int height) {
                applyEffect();
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

    private KFGLFilterListener mFilterListener = new KFGLFilterListener() {
        @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
        @Override
        public void preOnDraw() {
            glActiveTexture(GLES20.GL_TEXTURE5);
            glBindTexture(GLES20.GL_TEXTURE_2D, mStickerTexture);
            glUniform1i(mGLFilter.getOutputProgram().getUniformLocation("stickerTexture"), 5);
        }

        @Override
        public void postOnDraw() {

        }
    };

    public void applyEffect() {
        mGLContext.bind();
        if(mGLFilter == null){
            mGLFilter = new KFGLFilter(false, KFGLBase.defaultVertexShader,mFragmentShaderString,mFilterListener,null);

            int[] textures = new int[1];
            glGenTextures(1, textures, 0);
            mStickerTexture = textures[0];
            glActiveTexture(GLES20.GL_TEXTURE0);
            glBindTexture(GLES20.GL_TEXTURE_2D, mStickerTexture);
            glTexParameteri(GLES20.GL_TEXTURE_2D, GLES20.GL_TEXTURE_MIN_FILTER, GLES20.GL_LINEAR);
            glTexParameteri(GLES20.GL_TEXTURE_2D, GLES20.GL_TEXTURE_MAG_FILTER, GLES20.GL_LINEAR);
            glTexParameteri(GLES20.GL_TEXTURE_2D, GLES20.GL_TEXTURE_WRAP_S, GLES20.GL_CLAMP_TO_EDGE);
            glTexParameteri(GLES20.GL_TEXTURE_2D, GLES20.GL_TEXTURE_WRAP_T, GLES20.GL_CLAMP_TO_EDGE);
            GLUtils.texImage2D(GLES20.GL_TEXTURE_2D, 0, mStickerBitmap, 0);
            mStickerBitmap.recycle();

            int[] textures1 = new int[1];
            glGenTextures(1, textures1, 0);
            mLogoTexture = textures1[0];
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

    private static String getShaderString(Context context,String fileName) {
        StringBuilder stringBuilder = new StringBuilder();
        try {
            AssetManager assetManager = context.getAssets();
            BufferedReader bf = new BufferedReader(new InputStreamReader(
                    assetManager.open(fileName)));
            String line;
            while ((line = bf.readLine()) != null) {
                stringBuilder.append(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return stringBuilder.toString();
    }

    private Bitmap stickerBitmap() {
        Bitmap sBitmap = getImageFromAssetsFile(this,"sticker.jpg");
        Bitmap bitmap = Bitmap.createBitmap(mLogoBitmap.getWidth(),mLogoBitmap.getHeight(),Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(bitmap);
        Rect srcRect = new Rect(0,0,sBitmap.getWidth(),sBitmap.getHeight());
        Rect dstRect = new Rect(mLogoBitmap.getWidth() - 100 - 20, mLogoBitmap.getHeight() - 200 - 20, mLogoBitmap.getWidth() - 20, mLogoBitmap.getHeight() - 100 - 20);
        canvas.drawBitmap(sBitmap,srcRect,dstRect,null);

        return bitmap;
    }
}
```

**2)  视频采集渲染模式 KFVideoRenderActivity**

```java
public class KFVideoRenderActivity extends AppCompatActivity {
    private KFIVideoCapture mCapture;
    private KFVideoCaptureConfig mCaptureConfig;
    private KFRenderView mRenderView;
    private KFGLContext mGLContext;
    private KFGLFilter mGLFilter;
    private Bitmap mStickerBitmap;
    private int mStickerTexture;
    private String mFragmentShaderString;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_kfvideo_render);

        mStickerBitmap = stickerBitmap();
        mStickerBitmap = getTurnOverBitmap(mStickerBitmap);
        mFragmentShaderString = getShaderString(this,"filter.fs");

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
                mGLFilter = new KFGLFilter(false, KFGLBase.defaultVertexShader,mFragmentShaderString,mFilterListener,null);
                int[] textures = new int[1];
                glGenTextures(1, textures, 0);
                mStickerTexture = textures[0];
                glActiveTexture(GLES20.GL_TEXTURE0);
                glBindTexture(GLES20.GL_TEXTURE_2D, mStickerTexture);
                glTexParameteri(GLES20.GL_TEXTURE_2D, GLES20.GL_TEXTURE_MIN_FILTER, GLES20.GL_LINEAR);
                glTexParameteri(GLES20.GL_TEXTURE_2D, GLES20.GL_TEXTURE_MAG_FILTER, GLES20.GL_LINEAR);
                glTexParameteri(GLES20.GL_TEXTURE_2D, GLES20.GL_TEXTURE_WRAP_S, GLES20.GL_CLAMP_TO_EDGE);
                glTexParameteri(GLES20.GL_TEXTURE_2D, GLES20.GL_TEXTURE_WRAP_T, GLES20.GL_CLAMP_TO_EDGE);
                GLUtils.texImage2D(GLES20.GL_TEXTURE_2D, 0, mStickerBitmap, 0);
                mStickerBitmap.recycle();
            }
            KFFrame filterFrame = mGLFilter.render((KFTextureFrame)frame);
            mRenderView.render((KFTextureFrame) filterFrame);
            mGLContext.unbind();
        }

        private KFGLFilterListener mFilterListener = new KFGLFilterListener() {
            @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
            @Override
            public void preOnDraw() {
                glActiveTexture(GLES20.GL_TEXTURE5);
                glBindTexture(GLES20.GL_TEXTURE_2D, mStickerTexture);
                glUniform1i(mGLFilter.getOutputProgram().getUniformLocation("stickerTexture"), 5);
            }

            @Override
            public void postOnDraw() {

            }
        };
    };

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

    private static String getShaderString(Context context,String fileName) {
        StringBuilder stringBuilder = new StringBuilder();
        try {
            AssetManager assetManager = context.getAssets();
            BufferedReader bf = new BufferedReader(new InputStreamReader(
                    assetManager.open(fileName)));
            String line;
            while ((line = bf.readLine()) != null) {
                stringBuilder.append(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return stringBuilder.toString();
    }

    private Bitmap stickerBitmap() {
        Bitmap sBitmap = getImageFromAssetsFile(this,"sticker.jpg");
        Bitmap bitmap = Bitmap.createBitmap(720,1280,Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(bitmap);
        Rect srcRect = new Rect(0,0,sBitmap.getWidth(),sBitmap.getHeight());
        Rect dstRect = new Rect(720 - 100 - 20, 1280 - 200 - 20, 720 - 20, 1280 - 100 - 20);
        canvas.drawBitmap(sBitmap,srcRect,dstRect,null);

        return bitmap;
    }
}
```

可见，当我们用 `KFGLFilter` 将 OpenGL 渲染能力封装起来，并可以像增加渲染处理节点一样往现有渲染链中增加新的图像处理功能时，相关改动就变得很方便了。

