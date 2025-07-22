---
title: RenderDemo（8）：用 OpenGL 实现礼物特效
description: 介绍用 OpenGL 实现礼物特效的流程和原理，并提供 Demo 源码和解析。
author: Keyframe
date: 2025-02-23 14:18:08 +0800
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

这里是 RenderDemo 的第八篇：**用 OpenGL 实现礼物特效**。我们分别在 iOS 和 Android 平台实现了用 OpenGL 对图像进行礼物特效处理并渲染出来。

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
- RenderDemo（6）：用 OpenGL 实现贴纸（iOS+Android）
- RenderDemo（7）：用 OpenGL 实现滤镜（iOS+Android）

这些源码对于学习和理解 iOS/Android 音视频开发非常容易上手，有需要的朋友可以扫描下面二维码加入星球获取全部源码：

![微信扫码二维码加入我们](assets/img/keyframe-zsxq.png)
_微信扫码二维码加入我们_


---

礼物特效是直播与短视频特效的一把利刃，设计师可以很容易的将各种 AE 效果直接进行应用。相对于 GIF 、WEBP、Lottie 等特效更适用于大礼物效果。

## 1、礼物特效基础知识

可以通过制作 Alpha 通道分离的视频素材，再在客户端上通过 OpenGL 重新实现 Alpha 通道和 RGB 通道的混合，从而实现在端上播放带透明通道的视频。

### 1.1、视频生产

生产一个礼物，不论你是 GIF、APNG、WEBP，只要把它们生成序列帧，往 AE 里面一丢。

**1）输出正常 RGB 视频，导出视频保存。**

![RGB 视频](assets/resource/av-demo/render-demo-gift-effect-design.png)
_RGB 视频_

**2）输出纯通道 Alpha 视频，导出视频保存。**

![A 视频](assets/resource/av-demo/render-demo-gift-effect-design2.png)
_A 视频_

**3）新建一个宽度乘 2 的画布，把 2 个刚导出的视频左右分别放置，最后导出视频保存。**


{%
  include embed/video.html
  src='assets/resource/av-demo/render-demo-gift-effect-source.mp4'
  types='mp4'
  poster=''
  title='原视频'
  autoplay=true
  loop=true
  muted=true
%}

### 1.2、Shader 实现

Shader 实现如下：


```c
precision highp float;

varying vec2 textureCoordinate;
uniform sampler2D inputImageTexture;

void main()
{
    vec4 aColor = texture2D(inputImageTexture, vec2(textureCoordinate.x / 2.0,textureCoordinate.y));
    vec4 vColor = texture2D(inputImageTexture, vec2(0.5 + textureCoordinate.x / 2.0,textureCoordinate.y));
     
    gl_FragColor = vec4(vColor.x,vColor.y,vColor.z,aColor.x);
}
```

### 1.3、视图混合

视频特效包含透明通道，还需要设置 GLView Layer opaque 属性，这样可以与 UIView 进行颜色混合。

```objc
_glView.layer.opaque = NO;
```

### 1.4、渲染效果

视频的效果如下：

{%
  include embed/video.html
  src='assets/resource/av-demo/render-demo-gift-effect.mp4'
  types='mp4'
  poster=''
  title='礼物特效'
  autoplay=true
  loop=true
  muted=true
%}

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

### 2.2、礼物特效渲染结果渲染流程

我们在一个 `ViewController` 中实现了礼物特效。代码如下：


```objc
static NSString * const PlayerItemStatusContext = @"PlayerItemStatusContext";

@interface KFVideoRenderViewController ()<AVPlayerItemOutputPullDelegate>

@property (nonatomic, strong) KFOpenGLView *glView;
@property (nonatomic, strong) KFPixelBufferConvertTexture *pixelBufferConvertTexture;
@property (nonatomic, strong) EAGLContext *context;
@property (nonatomic, strong) KFGLFilter *filter;
@property (strong, nonatomic) AVPlayer *player;
@property (strong, nonatomic, nonnull) dispatch_queue_t videoOutputQueue;
@property (strong, nonatomic, nonnull) AVPlayerItemVideoOutput *videoOutput;
@property (strong, nonatomic, nonnull) CADisplayLink *displayLink;

@end

@implementation KFVideoRenderViewController
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
        NSString *path = [[NSBundle mainBundle] pathForResource:@"filter" ofType:@"fs"];
        _filter = [[KFGLFilter alloc] initWithCustomFBO:NO vertexShader:KFDefaultVertexShader fragmentShader:[NSString stringWithContentsOfURL:[NSURL fileURLWithPath:path] encoding:NSUTF8StringEncoding error:nil]];
        __weak typeof(self) _self = self;
        _filter.preDrawCallBack = ^(){
            __strong typeof(_self) sself = _self;
            if(sself){
            }
        };
    }
    return _filter;
}

- (void)dealloc {
}

- (void)viewDidDisappear:(BOOL)animated {
    [super viewDidDisappear:animated];
    [self->_displayLink invalidate];
    [_player.currentItem removeObserver:self forKeyPath:@"status"];
}

#pragma mark - Lifecycle
- (void)viewDidLoad {
    [super viewDidLoad];
    [self setupUI];
}

- (void)viewWillLayoutSubviews {
    [super viewWillLayoutSubviews];
    self.glView.frame = self.view.bounds;
}

#pragma mark - Action

- (void)setupUI {
    self.edgesForExtendedLayout = UIRectEdgeAll;
    self.extendedLayoutIncludesOpaqueBars = YES;
    self.title = @"Video Render";
    self.view.backgroundColor = [UIColor redColor];
    
    
    // 渲染 view。
    _glView = [[KFOpenGLView alloc] initWithFrame:self.view.bounds context:self.context];
    _glView.fillMode = KFGLViewContentModeFill;
    _glView.layer.opaque = NO;
    [self.view addSubview:self.glView];
    
    //player
    AVPlayerItem *item = [[AVPlayerItem alloc] initWithURL:[NSURL fileURLWithPath:[[NSBundle mainBundle] pathForResource:@"569" ofType:@"mp4"]]];
    item.audioTimePitchAlgorithm = AVAudioTimePitchAlgorithmTimeDomain;
    
    _videoOutputQueue = dispatch_queue_create("player.output.queue", DISPATCH_QUEUE_SERIAL);
    
    NSDictionary *attributes = @{(id) kCVPixelBufferPixelFormatTypeKey: @(kCVPixelFormatType_420YpCbCr8BiPlanarFullRange)};
    self.videoOutput = [[AVPlayerItemVideoOutput alloc] initWithPixelBufferAttributes:attributes];
    [self.videoOutput setDelegate:self queue:self.videoOutputQueue];
    
    _player = [AVPlayer playerWithPlayerItem:item];
    _player.actionAtItemEnd = AVPlayerActionAtItemEndPause;
    [_player.currentItem addOutput:self.videoOutput];
    [_player.currentItem addObserver:self forKeyPath:@"status" options:0 context:(__bridge void *)(PlayerItemStatusContext)];
    
    _displayLink = [CADisplayLink displayLinkWithTarget:self selector:@selector(displayLinkCallback:)];
    [_displayLink addToRunLoop:[NSRunLoop mainRunLoop] forMode:NSRunLoopCommonModes];
    [_displayLink setPaused:YES];
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSString *,id> *)change context:(void *)context{
    if (context == (__bridge void *)(PlayerItemStatusContext)) {
        if ([keyPath isEqualToString:@"status"]) {
            AVPlayerItem * item = (AVPlayerItem *)object;
            if (item.status == AVPlayerItemStatusReadyToPlay) { //准备好播放
                [self.player play];
                [self.displayLink setPaused:NO];
            }else if (item.status == AVPlayerItemStatusFailed){ //失败
                NSLog(@"failed");
            }
        }
    }
}

#pragma mark - CADisplayLink Callback
- (void)displayLinkCallback:(CADisplayLink *)sender {
    CMTime itemTime = [self.videoOutput itemTimeForHostTime:CACurrentMediaTime()];
    if ([self.videoOutput hasNewPixelBufferForItemTime:itemTime]) {
        CVPixelBufferRef pixelBuffer = NULL;
        pixelBuffer = [self.videoOutput copyPixelBufferForItemTime:itemTime itemTimeForDisplay:NULL];
        
        if(pixelBuffer){
            EAGLContext *preContext = [EAGLContext currentContext];
            [EAGLContext setCurrentContext:self.context];
            
            KFTextureFrame *textureFrame = [self.pixelBufferConvertTexture renderFrame:pixelBuffer time:itemTime];
            textureFrame.textureSize = CGSizeMake(textureFrame.textureSize.width / 2, textureFrame.textureSize.height);
            KFTextureFrame *filterFrame = [self.filter render:textureFrame];
            [self.glView displayFrame:filterFrame];
            
            [EAGLContext setCurrentContext:preContext];
            CFRelease(pixelBuffer);
        }
    }
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

### 3.2、礼物特效渲染结果渲染流程

我们在一个 `MainActivity` 中实现了礼物特效。代码如下：

```java
public class MainActivity extends AppCompatActivity {
    private MediaPlayer mMediaPlayer;
    private KFSurfaceTexture mSurfaceTexture = null;
    private KFGLFilter mOESConvert2DFilter;///< 特效
    private Surface mSurface = null;
    private KFRenderView mRenderView;
    private KFGLContext mGLContext;
    private KFGLFilter mGLFilter;
    private String mFragmentShaderString;
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

        mFragmentShaderString = getShaderString(this,"filter.fs");
        mGLContext = new KFGLContext(null);
        mRenderView = new KFRenderView(this,mGLContext.getContext());
        mRenderView.setOpaque(false);
        WindowManager windowManager = (WindowManager)this.getSystemService(this.WINDOW_SERVICE);
        Rect outRect = new Rect();
        windowManager.getDefaultDisplay().getRectSize(outRect);
        FrameLayout.LayoutParams params = new FrameLayout.LayoutParams(outRect.width(), outRect.height());
        addContentView(mRenderView,params);

        mGLContext.bind();
        mSurfaceTexture = new KFSurfaceTexture(mSurfaceTextureListener);
        mOESConvert2DFilter = new KFGLFilter(false, KFGLBase.defaultVertexShader,KFGLBase.oesFragmentShader);
        mGLContext.unbind();
        mSurface = new Surface(mSurfaceTexture.getSurfaceTexture());

        try {
            AssetManager assetManager = this.getAssets();
            AssetFileDescriptor videoAssetFile = assetManager.openFd("569.mp4");
            mMediaPlayer = new MediaPlayer();
            mMediaPlayer.setDataSource(videoAssetFile.getFileDescriptor(),
                    videoAssetFile.getStartOffset(), videoAssetFile.getLength());
            mMediaPlayer.setSurface(mSurface);
            mMediaPlayer.setLooping(true);
            mMediaPlayer.setOnPreparedListener(new MediaPlayer.OnPreparedListener() {
                @Override
                public void onPrepared(MediaPlayer mp) {
                    mMediaPlayer.start();
                }
            });
            mMediaPlayer.prepareAsync();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private KFGLFilterListener mFilterListener = new KFGLFilterListener() {
        @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
        @Override
        public void preOnDraw() {
        }

        @Override
        public void postOnDraw() {

        }
    };

    private KFSurfaceTextureListener mSurfaceTextureListener = new KFSurfaceTextureListener() {
        @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
        @Override
        //< SurfaceTexture 数据回调
        public void onFrameAvailable(SurfaceTexture surfaceTexture) {
            long timestamp = System.nanoTime();
            mGLContext.bind();
            ///< 刷新纹理数据至SurfaceTexture
            mSurfaceTexture.getSurfaceTexture().updateTexImage();
            KFTextureFrame frame = new KFTextureFrame(mSurfaceTexture.getSurfaceTextureId(),new Size(mMediaPlayer.getVideoWidth(),mMediaPlayer.getVideoHeight()),timestamp,true);
            mSurfaceTexture.getSurfaceTexture().getTransformMatrix(frame.textureMatrix);
            Matrix.scaleM(frame.positionMatrix,0,1,-1,1);
            KFTextureFrame convertFrame = (KFTextureFrame)mOESConvert2DFilter.render(frame);
            if(mGLFilter == null){
                mGLFilter = new KFGLFilter(false, KFGLBase.defaultVertexShader,mFragmentShaderString,mFilterListener,null);
            }
            convertFrame.textureSize = new Size(convertFrame.textureSize.getWidth() / 2,convertFrame.textureSize.getHeight());
            KFTextureFrame filterFrame = (KFTextureFrame)mGLFilter.render(convertFrame);
            mRenderView.render((KFTextureFrame) filterFrame);
            mGLContext.unbind();
        }
    };

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
}
```

可见，当我们用 `KFGLFilter` 将 OpenGL 渲染能力封装起来，并可以像增加渲染处理节点一样往现有渲染链中增加新的图像处理功能时，相关改动就变得很方便了。




---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
{: .prompt-tip }

