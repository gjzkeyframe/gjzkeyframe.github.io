---
title: RenderDemo（1）：用 OpenGL 画一个三角形
description: 介绍用 OpenGL 画一个三角形的流程和原理，并提供 Demo 源码和解析。
author: Keyframe
date: 2025-02-23 13:08:08 +0800
categories: [音视频源码示例]
tags: [音视频源码示例, 音视频, iOS, Android, OpenGL, 渲染]
pin: false
math: true
mermaid: true
---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频**、**AI** 领域的最新技术和产品信息：
>
>![微信公众号](assets/img/keyframe-mp.jpg)
_微信扫码关注我们_
{: .prompt-tip }


**渲染**是音视频技术栈相关的一个非常重要的方向，视频图像在设备上的展示、各种流行的视频特效都离不开渲染技术的支持。

在 RenderDemo 这个工程示例系列，我们将为大家展示一些渲染相关的 Demo，来向大家介绍如何在 iOS/Android 平台上手一些渲染相关的开发。

这里是第一篇：**用 OpenGL 画一个三角形**。我们分别在 iOS 和 Android 实现了用 OpenGL 画一个三角形的 Demo。在本文中，包括如下内容：

- 1）iOS OpenGL 绘制三角形 Demo；
- 2）Android OpenGL 绘制三角形 Demo；
- 3）详尽的代码注释，帮你理解代码逻辑和原理。

**如果你想要获得我们所有 Demo 的工程源码，可以在关注本公众号后，在公众号发送消息『AVDemo』来咨询。**


在继续阅读下文前，你可能需要对 OpenGL 的基础知识有一些了解，你可以看看这篇文章：[OpenGL 基础知识](https://mp.weixin.qq.com/s/XNiDto9ABfTCCpptDktHng)。


如果我们了解了 OpenGL ES 就会知道，虽然它定义了一套移动设备的图像渲染 API，但是并没有定义窗口系统。为了让 GLES 能够适配各种平台，GLES 需要与知道如何通过操作系统创建和访问窗口的库结合使用，这就有了 **EGL**，EGL 是 OpenGL ES 渲染 API 和本地窗口系统之间的一个中间接口层，它主要由系统制造商实现。EGL 提供如下机制：

- 与设备的原生窗口系统通信；
- 查询绘图表面的可用类型和配置；
- 创建绘图表面；
- 在 OpenGL ES 和其他图形渲染 API 之间同步渲染；
- 管理纹理贴图等渲染资源。

**EGL 是 OpenGL ES 与设备的桥梁，以实现让 OpenGL ES 能够在当前设备上进行绘制。**







## 1、iOS Demo


iOS 平台对 EGL 的实现是 **EAGL（Embedded Apple Graphics Library）**，其中 `CAEAGLLayer` 就是一种可以支持 OpenGL ES 绘制的图层类型，我们的 Demo 里会用它作为绘制三角形的图层。



代码比较简单，我们直接上代码：

`DMTriangleRenderView.m`


```
#import "DMTriangleRenderView.h"
#import <OpenGLES/ES2/gl.h>

// 定义顶点的数据结构：包括顶点坐标和颜色维度。
#define PositionDimension 3
#define ColorDimension 4
typedef struct {
    GLfloat position[PositionDimension]; // { x, y, z }
    GLfloat color[ColorDimension]; // {r, g, b, a}
} SceneVertex;


@interface DMTriangleRenderView ()
@property (nonatomic, assign) GLsizei width;
@property (nonatomic, assign) GLsizei height;

@property (nonatomic, strong) CAEAGLLayer *eaglLayer;
@property (nonatomic, strong) EAGLContext *eaglContext;

@property (nonatomic, assign) GLuint simpleProgram;

@property (nonatomic, assign) GLuint renderBuffer;
@property (nonatomic, assign) GLuint frameBuffer;

@end

@implementation DMTriangleRenderView

#pragma mark - Setup
- (instancetype)initWithFrame:(CGRect)frame {
    self = [super initWithFrame:frame];
    if (self) {
        _width = frame.size.width;
        _height = frame.size.height;
        [self render];
    }
    return self;
}

#pragma mark - Action
- (void)render {
    // 1、设定 layer 的类型。
    _eaglLayer = (CAEAGLLayer *) self.layer;
    _eaglLayer.opacity = 1.0;
    _eaglLayer.drawableProperties = @{kEAGLDrawablePropertyRetainedBacking: @(NO),
                                      kEAGLDrawablePropertyColorFormat: kEAGLColorFormatRGBA8};

    
    // 2、创建 OpenGL 上下文。
    EAGLRenderingAPI api = kEAGLRenderingAPIOpenGLES2; // 使用的 OpenGL API 的版本。
    EAGLContext *context = [[EAGLContext alloc] initWithAPI:api];
    if (!context) {
        NSLog(@"Create context failed!");
        return;
    }
    BOOL r = [EAGLContext setCurrentContext:context];
    if (!r) {
        NSLog(@"setCurrentContext failed!");
        return;
    }
    _eaglContext = context;

    
    // 3、申请并绑定渲染缓冲区对象 RBO 用来存储即将绘制到屏幕上的图像数据。
    glGenRenderbuffers(1, &_renderBuffer); // 创建 RBO。
    glBindRenderbuffer(GL_RENDERBUFFER, _renderBuffer); // 绑定 RBO 到 OpenGL 渲染管线。
    [_eaglContext renderbufferStorage:GL_RENDERBUFFER fromDrawable:_eaglLayer]; // 将渲染图层（_eaglLayer）的存储绑定到 RBO。

    
    // 4、申请并绑定帧缓冲区对象 FBO。FBO 本身不能用于渲染，只有绑定了纹理（Texture）或者渲染缓冲区（RBO）等作为附件之后才能作为渲染目标。
    glGenFramebuffers(1, &_frameBuffer); // 创建 FBO。
    glBindFramebuffer(GL_FRAMEBUFFER, _frameBuffer); // 绑定 FBO 到 OpenGL 渲染管线。
    // 将 RBO 绑定为 FBO 的一个附件，绑定后，OpenGL 对 FBO 的绘制会同步到 RBO 后再上屏。
    glFramebufferRenderbuffer(GL_FRAMEBUFFER,
                              GL_COLOR_ATTACHMENT0,
                              GL_RENDERBUFFER,
                              _renderBuffer);

    
    // 5、清理窗口颜色，并设置渲染窗口。
    glClearColor(0.5, 0.5, 0.5, 1.0); // 设置渲染窗口颜色。这里是灰色。
    glClear(GL_COLOR_BUFFER_BIT); // 清空旧渲染缓存。
    glViewport(0, 0, _width, _height); // 设置渲染窗口区域。


    // 6、加载和编译 shader，并链接到着色器程序。
    if (_simpleProgram) {
        glDeleteProgram(_simpleProgram);
        _simpleProgram = 0;
    }
    // 加载和编译 shader。
    NSString *simpleVSH = [[NSBundle mainBundle] pathForResource:@"simple" ofType:@"vsh"];
    NSString *simpleFSH = [[NSBundle mainBundle] pathForResource:@"simple" ofType:@"fsh"];
    _simpleProgram = [self loadShaderWithVertexShader:simpleVSH fragmentShader:simpleFSH];
    // 链接 shader program。
    glLinkProgram(_simpleProgram);
    // 打印链接日志。
    GLint linkStatus;
    glGetProgramiv(_simpleProgram, GL_LINK_STATUS, &linkStatus);
    if (linkStatus == GL_FALSE) {
        GLint infoLength;
        glGetProgramiv(_simpleProgram, GL_INFO_LOG_LENGTH, &infoLength);
        if (infoLength > 0) {
            GLchar *infoLog = malloc(sizeof(GLchar) * infoLength);
            glGetProgramInfoLog(_simpleProgram, infoLength, NULL, infoLog);
            NSLog(@"%s", infoLog);
            free(infoLog);
        }
    }
    glUseProgram(_simpleProgram);
    
    
    // 7、根据三角形顶点信息申请顶点缓冲区对象 VBO 和拷贝顶点数据。
    // 设置三角形 3 个顶点数据，包括坐标信息和颜色信息。
    const SceneVertex vertices[] = {
        {
            {-0.5,  0.5, 0.0},
            { 1.0, 0.0, 0.0, 1.000}
        }, // 左下 // 红色
        {
            {-0.5, -0.5, 0.0}, 
            { 0.0, 1.0, 0.0, 1.000}
        }, // 右下 // 绿色
        {
            { 0.5, -0.5, 0.0},
            { 0.0, 0.0, 1.0, 1.000}
        }, // 左上 // 蓝色
    };
    // 申请并绑定 VBO。VBO 的作用是在显存中提前开辟好一块内存，用于缓存顶点数据，从而避免每次绘制时的 CPU 与 GPU 之间的内存拷贝，可以提升渲染性能。
    GLuint vertexBufferID;
    glGenBuffers(1, &vertexBufferID); // 创建 VBO。
    glBindBuffer(GL_ARRAY_BUFFER, vertexBufferID); // 绑定 VBO 到 OpenGL 渲染管线。
    // 将顶点数据 (CPU 内存) 拷贝到 VBO（GPU 显存）。
    glBufferData(GL_ARRAY_BUFFER, // 缓存块类型。
                 sizeof(vertices), // 创建的缓存块尺寸。
                 vertices, // 要绑定的顶点数据。
                 GL_STATIC_DRAW); // 缓存块用途。
    
    // 8、绘制三角形。
    // 获取与 Shader 中对应的参数信息：
    GLuint vertexPositionLocation = glGetAttribLocation(_simpleProgram, "v_position");
    GLuint vertexColorLocation = glGetAttribLocation(_simpleProgram, "v_color");
    // 顶点位置属性。
    glEnableVertexAttribArray(vertexPositionLocation); // 启用顶点位置属性通道。
    // 关联顶点位置数据。
    glVertexAttribPointer(vertexPositionLocation, // attribute 变量的下标，范围是 [0, GL_MAX_VERTEX_ATTRIBS - 1]。
                          PositionDimension, // 指顶点数组中，一个 attribute 元素变量的坐标分量是多少（如：position, 程序提供的就是 {x, y, z} 点就是 3 个坐标分量）。
                          GL_FLOAT, // 数据的类型。
                          GL_FALSE, // 是否进行数据类型转换。
                          sizeof(SceneVertex), // 每一个数据在内存中的偏移量，如果填 0 就是每一个数据紧紧相挨着。
                          (const GLvoid*) offsetof(SceneVertex, position)); // 数据的内存首地址。
    // 顶点颜色属性。
    glEnableVertexAttribArray(vertexColorLocation);
    // 关联顶点颜色数据。
    glVertexAttribPointer(vertexColorLocation,
                          ColorDimension,
                          GL_FLOAT,
                          GL_FALSE,
                          sizeof(SceneVertex),
                          (const GLvoid*) offsetof(SceneVertex, color));
    // 绘制所有图元。
    glDrawArrays(GL_TRIANGLES, // 绘制的图元方式。
                 0, // 从第几个顶点下标开始绘制。
                 sizeof(vertices) / sizeof(vertices[0])); // 有多少个顶点下标需要绘制。
    // 把 Renderbuffer 的内容显示到窗口系统 (CAEAGLLayer) 中。
    [_eaglContext presentRenderbuffer:GL_RENDERBUFFER];
    
    
    // 9、清理。
    glDisableVertexAttribArray(vertexColorLocation); // 关闭顶点颜色属性通道。
    glDisableVertexAttribArray(vertexPositionLocation); // 关闭顶点位置属性通道。
    glBindBuffer(GL_ARRAY_BUFFER, 0); // 解绑 VBO。
    glBindFramebuffer(GL_FRAMEBUFFER, 0); // 解绑 FBO。
    glBindRenderbuffer(GL_RENDERBUFFER, 0); // 解绑 RBO。
}

#pragma mark - Utility
- (GLuint)loadShaderWithVertexShader:(NSString *)vert fragmentShader:(NSString *)frag {
    GLuint verShader, fragShader;
    GLuint program = glCreateProgram(); // 创建 Shader Program 对象。
    
    [self compileShader:&verShader type:GL_VERTEX_SHADER file:vert];
    [self compileShader:&fragShader type:GL_FRAGMENT_SHADER file:frag];
    
    // 装载 Vertex Shader 和 Fragment Shader。
    glAttachShader(program, verShader);
    glAttachShader(program, fragShader);
    
    glDeleteShader(verShader);
    glDeleteShader(fragShader);
    
    return program;
}

- (void)compileShader:(GLuint *)shader type:(GLenum)type file:(NSString *)file {
    NSString *content = [NSString stringWithContentsOfFile:file encoding:NSUTF8StringEncoding error:nil];
    const GLchar *source = (GLchar *) [content UTF8String];
    *shader = glCreateShader(type); // 创建一个着色器对象。
    glShaderSource(*shader, 1, &source, NULL); // 关联顶点、片元着色器的代码。
    glCompileShader(*shader); // 编译着色器代码。
    
    // 打印编译日志。
    GLint compileStatus;
    glGetShaderiv(*shader, GL_COMPILE_STATUS, &compileStatus);
    if (compileStatus == GL_FALSE) {
        GLint infoLength;
        glGetShaderiv(*shader, GL_INFO_LOG_LENGTH, &infoLength);
        if (infoLength > 0) {
            GLchar *infoLog = malloc(sizeof(GLchar) * infoLength);
            glGetShaderInfoLog(*shader, infoLength, NULL, infoLog);
            NSLog(@"%s -> %s", (type == GL_VERTEX_SHADER) ? "vertex shader" : "fragment shader", infoLog);
            free(infoLog);
        }
    }
}

#pragma mark - Override
+ (Class)layerClass {
    return [CAEAGLLayer class];
}

@end
```

上面的代码包括了这些过程：

- 1）定义顶点的数据结构：包括顶点坐标和颜色维度；
- 2）设定 layer 的类型；
- 3）创建 OpenGL 上下文；
- 4）申请并绑定渲染缓冲区对象 RBO 用来存储即将绘制到屏幕上的图像数据；
- 5）申请并绑定帧缓冲区对象 FBO；
	- 需要注意，FBO 本身不能用于渲染，只有绑定了纹理（Texture）或者渲染缓冲区（RBO）等作为附件之后才能作为渲染目标。
- 6）清理窗口颜色，并设置渲染窗口；
- 7）加载和编译 shader，并链接到着色器程序；
- 8）根据三角形顶点信息申请顶点缓冲区对象 VBO 和拷贝顶点数据；
	- 这里 VBO 的作用是在显存中提前开辟好一块内存，用于缓存顶点数据，从而避免每次绘制时的 CPU 与 GPU 之间的内存拷贝，可以提升渲染性能。
- 9）绘制三角形；
- 10）清理状态机。

更具体细节见上述代码及其注释。

最终我们画出的三角形如下图所示：

![OpenGL 绘制三角形（iOS）](assets/resource/av-demo/render-demo-draw-a-triangle-1.png)
_OpenGL 绘制三角形（iOS）_



## 2、Android Demo

Android 平台自 2.0 版本之后图形系统的底层渲染均由 OpenGL ES 负责，其 EGL 架构实现如下图所示：

![Android EGL 架构](assets/resource/av-demo/render-demo-draw-a-triangle-2.png)
_Android EGL 架构_

- **Display** 是对实际显示设备的抽象。在 Android 上的实现类是 `EGLDisplay`。
- **Surface** 是对用来存储图像的内存区域 FrameBuffer 的抽象，包括 Color Buffer、Stencil Buffer、Depth Buffer。在 Android 上的实现类是 `EGLSurface`。
- **Context** 存储 OpenGL ES 绘图的一些状态信息。在 Android 上的实现类是 `EGLContext`。


Android Demo 的代码由于平台 EGL 实现的原因以及对模块略作了封装，所以看起来对稍微复杂一些，包括了如下几个文件：

- `KFGLContext.java`，KFGLContext 负责管理和组装 EGLDisplay、EGLSurface、EGLContext。
- `KFGLProgram.java`，KFGLProgram 负责加载和编译着色器，创建着色器程序容器。
- `KFSurfaceView.java`，KFSurfaceView 继承自 SurfaceView 来实现渲染。
- `KFTextureView.java`，KFTextureView 继承自 TextureView 来实现渲染。
- `KFRenderView.java`，KFRenderView 是一个容器，可以选择使用 KFSurfaceView 或 KFTextureView 作为实际的渲染视图。
- `KFRenderListener.java`，KFRenderListener 是 KFRenderView 用来监听渲染视图的事件回调。





代码如下：

`KFGLContext.java`

```java
// ...省略 import 代码...
public class KFGLContext {
    private Surface mSurface = null;
    private EGLDisplay mEGLDisplay = EGL_NO_DISPLAY; // 实际显示设备的抽象
    private EGLContext mEGLContext = EGL_NO_CONTEXT; // 渲染上下文
    private EGLSurface mEGLSurface = EGL_NO_SURFACE; // 存储图像的内存区域
    private EGLContext mEGLShareContext = EGL_NO_CONTEXT; // 共享渲染上下文
    private EGLConfig mEGLConfig = null; // 渲染表面的配置信息

    private EGLContext mLastContext = EGL_NO_CONTEXT; // 存储之前系统上下文
    private EGLDisplay mLastDisplay = EGL_NO_DISPLAY; // 存储之前系统设备
    private EGLSurface mLastSurface = EGL_NO_SURFACE; // 存储之前系统内存区域
    private boolean mIsBind = false;

    public KFGLContext(EGLContext shareContext) {
        mEGLShareContext = shareContext;
        // 创建 OpenGL 上下文
        _eglSetup();
    }

    public KFGLContext(EGLContext shareContext, Surface surface) {
        mEGLShareContext = shareContext;
        mSurface = surface;
        // 创建 OpenGL 上下文
        _eglSetup();
    }

    public void setSurface(Surface surface) {
        if (surface == null || surface == mSurface) {
            return;
        }

        // 释放渲染表面 Surface
        if (mEGLDisplay != EGL_NO_DISPLAY && mEGLSurface != EGL_NO_SURFACE) {
            eglDestroySurface(mEGLDisplay, mEGLSurface);
        }
        // 创建 Surface
        int[] surfaceAttribs = {
                EGL14.EGL_NONE
        };
        mSurface = surface;
        mEGLSurface = eglCreateWindowSurface(mEGLDisplay, mEGLConfig, mSurface, surfaceAttribs, 0);
        if (mEGLSurface == null) {
            throw new RuntimeException("surface was null");
        }
    }

    public EGLContext getContext() {
        return mEGLContext;
    }

    public Surface getSurface() {
        return mSurface;
    }

    public boolean swapBuffers() {
        // 将后台绘制的缓冲显示到前台
        if (mEGLDisplay != EGL_NO_DISPLAY && mEGLSurface != EGL_NO_SURFACE) {
            return eglSwapBuffers(mEGLDisplay, mEGLSurface);
        } else {
            return false;
        }
    }

    @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN_MR2)
    public void setPresentationTime(long nsecs) {
        // 设置时间戳
        if (mEGLDisplay != EGL_NO_DISPLAY && mEGLSurface != EGL_NO_SURFACE) {
            eglPresentationTimeANDROID(mEGLDisplay, mEGLSurface, nsecs);
        }
    }

    public void bind() {
        // 绑定当前上下文
        mLastSurface = eglGetCurrentSurface(EGL14.EGL_READ);
        mLastContext = eglGetCurrentContext();
        mLastDisplay = eglGetCurrentDisplay();
        if (!eglMakeCurrent(mEGLDisplay, mEGLSurface, mEGLSurface, mEGLContext)) {
            throw new RuntimeException("eglMakeCurrent failed");
        }
        mIsBind = true;
    }

    public void unbind() {
        if (!mIsBind) {
            return;
        }

        // 绑定回系统之前上下文
        if (mLastSurface != EGL_NO_SURFACE && mLastContext != EGL_NO_CONTEXT  && mLastDisplay != EGL_NO_DISPLAY) {
            eglMakeCurrent(mLastDisplay, mLastSurface, mLastSurface, mLastContext);
            mLastDisplay = EGL_NO_DISPLAY;
            mLastSurface = EGL_NO_SURFACE;
            mLastContext = EGL_NO_CONTEXT;
        } else {
            if (mEGLDisplay != EGL_NO_DISPLAY) {
                eglMakeCurrent(mEGLDisplay, EGL_NO_SURFACE, EGL_NO_SURFACE, EGL_NO_CONTEXT);
            }
        }
        mIsBind = false;
    }

    public void release() {
        unbind();

        // 释放设备、Surface
        if (mEGLDisplay != EGL_NO_DISPLAY && mEGLSurface != EGL_NO_SURFACE) {
            eglDestroySurface(mEGLDisplay, mEGLSurface);
        }

        if (mEGLDisplay != EGL_NO_DISPLAY && mEGLContext != EGL_NO_CONTEXT) {
            eglDestroyContext(mEGLDisplay, mEGLContext);
        }

        if(mEGLDisplay != EGL_NO_DISPLAY){
            eglTerminate(mEGLDisplay);
        }

        mSurface = null;
        mEGLShareContext = null;

        mEGLDisplay = null;
        mEGLContext = null;
        mEGLSurface = null;
    }

    private void _eglSetup() {
        // 创建设备
        mEGLDisplay = eglGetDisplay(EGL14.EGL_DEFAULT_DISPLAY);
        if (mEGLDisplay == EGL_NO_DISPLAY) {
            throw new RuntimeException("unable to get EGL14 display");
        }

        int[] version = new int[2];
        // 根据版本初始化设备
        if (!eglInitialize(mEGLDisplay, version, 0, version, 1)) {
            mEGLDisplay = null;
            throw new RuntimeException("unable to initialize EGL14");
        }

        // 定义 EGLConfig 属性配置，定义红、绿、蓝、透明度、深度、模板缓冲的位数
        int[] attribList = {
                EGL14.EGL_BUFFER_SIZE, 32,
                EGL14.EGL_ALPHA_SIZE, 8, // 颜色缓冲区中透明度用几位来表示
                EGL14.EGL_BLUE_SIZE, 8,
                EGL14.EGL_GREEN_SIZE, 8,
                EGL14.EGL_RED_SIZE, 8,
                EGL14.EGL_RENDERABLE_TYPE, EGL14.EGL_OPENGL_ES2_BIT,
                EGL14.EGL_SURFACE_TYPE, EGL14.EGL_WINDOW_BIT,
                EGL14.EGL_NONE
        };
        EGLConfig[] configs = new EGLConfig[1];
        int[] numConfigs = new int[1];
        // 找到符合要求的 EGLConfig 配置
        if (!eglChooseConfig(mEGLDisplay, attribList, 0, configs, 0, configs.length,
                numConfigs, 0)) {
            throw new RuntimeException("unable to find RGB888+recordable ES2 EGL config");
        }
        mEGLConfig = configs[0];

        // 指定 OpenGL 使用版本
        int[] attrib_list = {
                EGL14.EGL_CONTEXT_CLIENT_VERSION, 2,
                EGL14.EGL_NONE
        };
        // 创建 GL 上下文
        mEGLContext = eglCreateContext(mEGLDisplay, mEGLConfig, mEGLShareContext != null ? mEGLShareContext : EGL_NO_CONTEXT, attrib_list, 0);
        if (mEGLContext == null) {
            throw new RuntimeException("null context");
        }

        // 创建 Surface 配置信息
        int[] surfaceAttribs = {
                EGL14.EGL_NONE
        };

        // 创建 Surface
        if (mSurface != null) {
            mEGLSurface = eglCreateWindowSurface(mEGLDisplay, mEGLConfig, mSurface,
                    surfaceAttribs, 0);
        } else {
            mEGLSurface = eglCreatePbufferSurface(mEGLDisplay, configs[0], surfaceAttribs, 0);
        }

        if (mEGLSurface == null) {
            throw new RuntimeException("surface was null");
        }
    }
}
```

`KFGLProgram.java`

```java
// ...省略 import 代码...
public class KFGLProgram {
    private static final String TAG = "KFGLProgram";
    private int mProgram = 0; // 着色器程序容器
    private int mVertexShader = 0; // 顶点着色器
    private int mFragmentShader = 0; // 片元着色器

    public KFGLProgram(String vertexShader, String fragmentShader) {
        _createProgram(vertexShader,fragmentShader);
    }

    public void release() {
        // 释放顶点、片元着色器、着色器容器
        if (mVertexShader != 0) {
            glDeleteShader(mVertexShader);
            mVertexShader = 0;
        }

        if (mFragmentShader != 0) {
            glDeleteShader(mFragmentShader);
            mFragmentShader = 0;
        }

        if (mProgram != 0) {
            glDeleteProgram(mProgram);
            mProgram = 0;
        }
    }

    public void use() {
        // 使用当前的着色器
        if (mProgram != 0) {
            glUseProgram(mProgram);
        }
    }

    public int getUniformLocation(String uniformName) {
        // 获取着色器 uniform 对应下标
        return glGetUniformLocation(mProgram, uniformName);
    }

    public int getAttribLocation(String uniformName) {
        // 获取着色器 attribute 变量对应下标
        return glGetAttribLocation(mProgram, uniformName);
    }

    private void  _createProgram(String vertexSource, String fragmentSource) {
        // 创建着色器容器
        // 创建顶点、片元着色器
        mVertexShader = _loadShader(GLES20.GL_VERTEX_SHADER,   vertexSource);
        mFragmentShader = _loadShader(GLES20.GL_FRAGMENT_SHADER, fragmentSource);

        //
        if(mVertexShader != 0 && mFragmentShader != 0){
            // 创建一个空的着色器容器
            mProgram = GLES20.glCreateProgram();
            // 将顶点、片元着色器添加至着色器容器
            glAttachShader(mProgram, mVertexShader);
            glAttachShader(mProgram, mFragmentShader);

            // 链接着色器容器
            glLinkProgram(mProgram);
            int[] linkStatus = new int[1];
            glGetProgramiv(mProgram, GLES20.GL_LINK_STATUS, linkStatus, 0);
            // 获取链接状态
            if (linkStatus[0] != GLES20.GL_TRUE) {
                Log.e(TAG, "Could not link program:");
                Log.e(TAG, glGetProgramInfoLog(mProgram));
                glDeleteProgram(mProgram);
                mProgram = 0;
            }
        }
    }

    private int _loadShader(int shaderType, String source) {
        // 根据类型创建顶点、片元着色器
        int shader = glCreateShader(shaderType);
        // 设置着色器中的源代码
        glShaderSource(shader, source);
        // 编译着色器
        glCompileShader(shader);

        int[] compiled = new int[1];
        glGetShaderiv(shader, GLES20.GL_COMPILE_STATUS, compiled, 0);
        // 获取编译后状态
        if (compiled[0] != GLES20.GL_TRUE) {
            Log.e(TAG, "Could not compile shader(TYPE=" + shaderType + "):");
            Log.e(TAG, glGetShaderInfoLog(shader));
            glDeleteShader(shader);
            shader = 0;
        }

        return shader;
    }
}
```


`KFSurfaceView.java`

```java
// ...省略 import 代码...
public class KFSurfaceView extends SurfaceView implements SurfaceHolder.Callback {
    private KFRenderListener mListener = null; // 回调
    private SurfaceHolder mHolder = null; // Surface 的抽象接口

    public KFSurfaceView(Context context, KFRenderListener listener) {
        super(context);
        mListener = listener;
        getHolder().addCallback(this);
    }

    @Override
    public void surfaceCreated(@NonNull SurfaceHolder surfaceHolder) {
        // Surface 创建
        mHolder = surfaceHolder;
        // 根据 SurfaceHolder 创建 Surface
        if(mListener != null){
            mListener.surfaceCreate(surfaceHolder.getSurface());
        }
    }

    @Override
    public void surfaceChanged(@NonNull SurfaceHolder surfaceHolder, int format, int width, int height) {
        // Surface 分辨率变更
        if(mListener != null){
            mListener.surfaceChanged(surfaceHolder.getSurface(),width,height);
        }
    }

    @Override
    public void surfaceDestroyed(@NonNull SurfaceHolder surfaceHolder) {
        // Surface 销毁
        if (mListener != null) {
            mListener.surfaceDestroy(surfaceHolder.getSurface());
        }
    }
}
```



`KFTextureView.java`

```java
// ...省略 import 代码...
public class KFTextureView extends TextureView implements TextureView.SurfaceTextureListener{
    private KFRenderListener mListener = null; // 回调
    private Surface mSurface = null; // 渲染缓存
    private SurfaceTexture mSurfaceTexture = null; // 纹理缓存

    public KFTextureView(Context context, KFRenderListener listener) {
        super(context);
        this.setSurfaceTextureListener(this);
        mListener = listener;
    }

    @Override
    public void onSurfaceTextureAvailable(@NonNull SurfaceTexture surfaceTexture, int width, int height) {
        // 纹理缓存创建
        mSurfaceTexture = surfaceTexture;
        // 根据 SurfaceTexture 创建 Surface
        mSurface = new Surface(surfaceTexture);
        if (mListener != null) {
            // 创建时候回调一次分辨率变更，对应 SurfaceView 接口
            mListener.surfaceCreate(mSurface);
            mListener.surfaceChanged(mSurface,width,height);
        }
    }

    @Override
    public void onSurfaceTextureSizeChanged(@NonNull SurfaceTexture surfaceTexture, int width, int height) {
        // 纹理缓存变更分辨率
        if (mListener != null) {
            mListener.surfaceChanged(mSurface,width,height);
        }
    }

    @Override
    public void onSurfaceTextureUpdated(@NonNull SurfaceTexture surfaceTexture) {

    }

    @Override
    public boolean onSurfaceTextureDestroyed(@NonNull SurfaceTexture surfaceTexture) {
        // 纹理缓存销毁
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



`KFRenderView.java`

```java
// ...省略 import 代码...
public class KFRenderView extends ViewGroup {
    // 顶点着色器
    public static String customVertexShader =
                    "attribute vec4 v_position;\n" +
                    "attribute vec4 v_color;\n" +
                    "varying mediump vec4 f_color;\n" +
                    "void main() {\n" +
                    "  f_color = v_color;\n" +
                    "  gl_Position = v_position;\n" +
                    "}\n";

    // 片元着色器
    public static String customFragmentShader =
                    "varying mediump vec4 f_color;\n" +
                    "void main() {\n" +
                    "  gl_FragColor = f_color;\n" +
                    "}\n";

    private KFGLContext mEGLContext = null; // OpenGL 上下文
    private EGLContext mShareContext = null; // 共享上下文
    private View mRenderView = null; // 渲染视图基类
    private int mSurfaceWidth = 0; // 渲染缓存宽
    private int mSurfaceHeight = 0; // 渲染缓存高
    private boolean mSurfaceChanged = false; // 渲染缓存是否变更
    private KFGLProgram mProgram;
    private FloatBuffer mVerticesBuffer = null; // 顶点 buffer
    private int mPositionAttribute = -1; // 顶点坐标
    private int mColorAttribute = -1; // 顶点颜色

    public KFRenderView(Context context, EGLContext eglContext) {
        super(context);
        mShareContext = eglContext; // 共享上下文

        // 1、选择实际的渲染视图
        boolean isSurfaceView  = false; // TextureView 与 SurfaceView 开关
        if (isSurfaceView) {
            mRenderView = new KFSurfaceView(context, mListener);
        } else {
            mRenderView = new KFTextureView(context, mListener);
        }

        this.addView(mRenderView); // 添加视图到父视图
    }

    public void release() {
        // 释放 OpenGL 上下文、特效
        if (mEGLContext != null) {
            mEGLContext.bind();
            mEGLContext.unbind();

            mEGLContext.release();
            mEGLContext = null;
        }
    }

    public void render() {
        mProgram.use();

        // 设置帧缓存背景色
        glClearColor(0.5f,0.5f,0.5f,1);
        // 清空帧缓存颜色
        glClear(GLES20.GL_COLOR_BUFFER_BIT);
        // 设置渲染窗口区域
        GLES20.glViewport(0, 0, mSurfaceWidth, mSurfaceHeight);

        // 启用顶点着色器顶点坐标属性
        glEnableVertexAttribArray(mPositionAttribute);
        mVerticesBuffer.position(0); // 定位到第一个位置分量
        glVertexAttribPointer(
                mPositionAttribute,
                3, // x, y, z 有 3 个分量
                GLES20.GL_FLOAT,
                false,
                7 * 4, // 每个顶点有 xyzrgba 7 个分量，每个分量是 4 字节，所以步进为 7 * 4 字节
                mVerticesBuffer);

        // 启用顶点着色器顶点颜色属性
        glEnableVertexAttribArray(mColorAttribute);
        mVerticesBuffer.position(3); // 定位到第一个颜色分量
        glVertexAttribPointer(
                mColorAttribute,
                4, // r, g, b, a 有 4 个分量
                GLES20.GL_FLOAT,
                false,
                7 * 4, // 每个顶点有 xyzrgba 7 个分量，每个分量是 4 字节，所以步进为 7 * 4 字节
                mVerticesBuffer);

        // 绘制三角形
        glDrawArrays(GLES20.GL_TRIANGLES, 0, 3);

        // 关闭顶点着色器顶点坐标属性
        glDisableVertexAttribArray(mPositionAttribute);

        // 关闭顶点着色器顶点颜色属性
        glDisableVertexAttribArray(mColorAttribute);

        mEGLContext.swapBuffers();
    }

    private KFRenderListener mListener = new KFRenderListener() {
        @Override
        // 渲染缓存创建
        public void surfaceCreate(@NonNull Surface surface) {
            // 2、创建 OpenGL 上下文
            mEGLContext = new KFGLContext(mShareContext, surface);
            mEGLContext.bind();
            // 3、初始化 GL 相关环境：加载和编译 shader、链接到着色器程序、设置顶点数据
            _setupGL();
            mEGLContext.unbind();
        }

        @Override
        // 渲染缓存变更
        public void surfaceChanged(@NonNull Surface surface, int width, int height) {
            mSurfaceWidth = width;
            mSurfaceHeight = height;
            mSurfaceChanged = true;
            mEGLContext.bind();
            // 4、设置 OpenGL 上下文 Surface
            mEGLContext.setSurface(surface);
            // 5、绘制三角形
            render();
            mEGLContext.unbind();
        }

        @Override
        public void surfaceDestroy(@NonNull Surface surface) {

        }
    };

    private void _setupGL() {
        // 加载和编译 shader，并链接到着色器程序
        mProgram = new KFGLProgram(customVertexShader, customFragmentShader);

        // 获取与 shader 中对应的属性信息
        mPositionAttribute = mProgram.getAttribLocation("v_position");
        mColorAttribute = mProgram.getAttribLocation("v_color");

        // 3 个顶点，每个顶点有 7 个分量：x, y, z, r, g, b, a
        final float vertices[] = {
                -0.5f,  0.5f, 0.0f,  1.0f, 0.0f, 0.0f, 1.0f, // 左下 // 红色
                -0.5f, -0.5f, 0.0f,  0.0f, 1.0f, 0.0f, 1.0f, // 右下 // 绿色
                 0.5f, -0.5f, 0.0f,  0.0f, 0.0f, 1.0f, 1.0f, // 左上 // 蓝色
        };
        ByteBuffer verticesByteBuffer = ByteBuffer.allocateDirect(4 * vertices.length);
        verticesByteBuffer.order(ByteOrder.nativeOrder());
        mVerticesBuffer = verticesByteBuffer.asFloatBuffer();
        mVerticesBuffer.put(vertices);
        mVerticesBuffer.position(0);
    }

    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        // 视图变更 Size
        this.mRenderView.layout(left, top, right, bottom);
    }
}
```

`KFRenderListener.java`

```java
// ...省略 import 代码...
public interface KFRenderListener {
    void surfaceCreate(@NonNull Surface surface); // 渲染缓存创建
    void surfaceChanged(@NonNull Surface surface, int width, int height); // 渲染缓存变更分辨率
    void surfaceDestroy(@NonNull Surface surface); // 渲染缓存销毁
}
```

其中绘制三角形的代码实现在 `KFRenderView.java` 中，包括这些过程：

- 1）选择实际的渲染视图；
- 2）创建 OpenGL 上下文；
- 3）初始化 GL 相关环境：加载和编译 shader、链接到着色器程序、设置顶点数据；
- 4）设置 OpenGL 上下文 Surface；
- 5）绘制三角形。

具体细节见上述代码及其注释。


最终我们画出的三角形如下图所示：

![OpenGL 绘制三角形（Android）](assets/resource/av-demo/render-demo-draw-a-triangle-3.png)
_OpenGL 绘制三角形（Android）_










