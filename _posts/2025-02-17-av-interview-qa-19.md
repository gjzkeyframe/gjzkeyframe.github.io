---
title: 音视频面试题集锦第 19 期
description: 持续更新的音视频面试题集锦。
author: Keyframe
date: 2025-02-17 03:58:38 +0800
categories: [音视频面试题集锦]
tags: [音视频面试题集锦,  面试, 音视频]
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




我们在知识星球上创建的音视频技术社群**关键帧的音视频开发圈**已经运营了一段时间了，在这里大家可以一起交流和分享音视频技术知识和实战方案。我们会不定期整理一些音视频相关的面试题，汇集一份[音视频面试题集锦（可进入免费订阅）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2380776196751425539#wechat_redirect)。也会循序渐进地归纳总结音视频技术知识，绘制一幅[音视频知识图谱（可进入免费订阅）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2349658423078092802#wechat_redirect)。



下面是第 19 期面试题精选，我们来介绍几种在 Android 开发中读取纹理数据的方法：


- **1、介绍一下 glReadPixels？**
- **2、介绍一下 ImageReader？**
- **3、介绍一下 PBO（Pixel Buffer Object）？**
- **4、介绍一下 HardwareBuffer？**




## 1、介绍一下 glReadPixels？

`glReadPixels` 是 OpenGL ES 的 API，通常用于从帧缓冲区中读取像素数据，OpenGL ES 2.0 和 3.0 均支持。 使用非常方便，但是效率也是最低的。

- 当调用 `glReadPixels` 时，首先会影响 CPU 时钟周期，同时 GPU 会等待当前帧绘制完成，读取像素完成之后，才开始下一帧的计算，造成渲染管线停滞。
- `glReadPixels` 读取的是当前绑定 FBO 的颜色缓冲区图像，所以当使用多个 FBO（帧缓冲区对象）时，需要确定好我们要读那个 FBO 的颜色缓冲区。
- `glReadPixels` 性能瓶颈一般出现在大分辨率图像的读取，所以目前通用的优化方法是在 shader 中将处理完成的 RGBA 转成 YUV （一般是 YUYV 格式），然后基于 RGBA 的格式读出 YUV 图像，这样传输数据量会降低一半，性能提升明显。


下面我们介绍两种使用 glReadPixels 来进行 RGBA 转换 NV21 的示例：

**1）直接获取 RGBA 数据**

这种方式 GPU 传输数据到 CPU 耗时比较长。

```c
glBindFramebuffer(GL_FRAMEBUFFER, fbo);
glReadPixels(0, 0, width, height, GL_RGBA, GL_UNSIGNED_BYTE, rgbaByteAddr);
libyuv::ABGRToNV21(rgbaByteAddr, width * 4, yByte, width, uvByte, width, width, height);;
glBindFramebuffer(GL_FRAMEBUFFER, 0);
```

**2）OpenGL 扩展格式 YUV**


```c
// Draw Y
TextureAttributes textureAttriburesY = {
  .minFilter = GL_LINEAR,
  .magFilter = GL_LINEAR,
  .wrapS = GL_CLAMP_TO_EDGE,
  .wrapT = GL_CLAMP_TO_EDGE,
  .internalFormat = GL_RED_EXT,
  .format = GL_RED_EXT,
  .type = GL_UNSIGNED_BYTE
};

varying vec2 textureCoordinate;
uniform sampler2D inputImageTexture;
void main()
{
 vec4 color = texture2D(inputImageTexture,textureCoordinate);
 gl_FragColor.r = color.r*0.2990+color.g*0.5870+color.b*0.1140;
}

// Draw UV
TextureAttributes textureAttriburesVU = {
  .minFilter = GL_LINEAR,
  .magFilter = GL_LINEAR,
  .wrapS = GL_CLAMP_TO_EDGE,
  .wrapT = GL_CLAMP_TO_EDGE,
  .internalFormat = GL_RG_EXT,
  .format = GL_RG_EXT,
  .type = GL_UNSIGNED_BYTE
};

varying vec2 textureCoordinate;
uniform sampler2D inputImageTexture;
void main()
{
 vec4 color = texture2D(inputImageTexture,textureCoordinate);
 gl_FragColor.rg = vec2(0.6150*color.r - 0.5150*color.g - 0.1000*color.b+0.5000,-0.1471*color.r - 0.2889*color.g + 0.4360*color.b+0.5000);
}
```

```c
glBindFramebuffer(GL_FRAMEBUFFER, yFbo);
glReadPixels(0, 0, width, height, GL_RED_EXT, GL_UNSIGNED_BYTE, yuv_byte);
glBindFramebuffer(GL_FRAMEBUFFER, 0);

glBindFramebuffer(GL_FRAMEBUFFER, uvFbo);
glReadPixels(0, 0, width / 2, height / 2, GL_RG_EXT, GL_UNSIGNED_BYTE, yuv_byte + width * height);
glBindFramebuffer(GL_FRAMEBUFFER, 0);
```





## 2、介绍一下 ImageReader？


**1）ImageReader 基础描述?**


`ImageReader` 是 Android 中的一个类，用于获取相机设备的图像数据。它可以用于捕获相机拍摄的静态图像或实时预览帧，并提供对图像数据的访问和处理。以下是一些 `ImageReader` 的特点和用法：

- 获取图像数据：通过创建一个 `ImageReader` 实例，可以指定要获取的图像的宽度、高度和图像格式。然后，可以使用`ImageReader` 的 `acquireLatestImage()` 或 `acquireNextImage()` 方法获取最新的图像或下一帧图像。这些方法返回一个 `Image` 对象，它包含了图像的数据和相关信息。
- 图像数据访问：通过 `Image` 对象，可以访问图像的像素数据。可以使用 `getPlanes()` 方法获取图像的平面数组，每个平面对应于图像的不同颜色通道。然后，可以使用 `getBuffer()` 方法获取每个平面的 `ByteBuffer`，从中读取或修改像素数据。
- 回收资源：使用完 `Image` 对象后，应调用其 `close()` 方法释放资源，以避免内存泄漏。
- 设置图像可用监听器：可以为 `ImageReader` 设置一个 `OnImageAvailableListener` 监听器，在新图像可用时收到通知，这样可以实现对图像数据的实时处理和分析。
- 配置图像输出：可以使用 `ImageReader` 的 `setOnImageAvailableListener()` 方法设置监听器，并通过 `ImageReader` 的 `getSurface()` 方法获取一个 `Surface` 对象，将其用于预览或拍照时的图像输出目标。


**2）ImageReader 如何使用？**


我们可以使用 `ImageReader` 对象的 Surface 对象搭配 OpenGL 进行数据渲染。


```c
mImageReader = ImageReader.newInstance(width, height, ImageFormat.YUV_420_888, 2);
mImageReader.setOnImageAvailableListener(mOnImageAvailableListener, mHandler);
mSurface = mImageReader.getSurface();
private ImageReader.OnImageAvailableListener mOnImageAvailableListener = new ImageReader.OnImageAvailableListener() {
    @Override
    public void onImageAvailable(ImageReader reader) {
        Image image = reader.acquireLatestImage();
        if (image != null) {
           image.close();
        }
    }
};
```

部分重要 API：


- `acquireLatestImage()` 从 `ImageReader` 队列中获取最新的一帧 `Image` ，并且将老的 `Image` 丢弃，如果没有新的可用的 `Image` 则返回 `null` 。 此操作将会从 `ImageReader` 中获取所有可获取到的 `Images` ，并且关闭除了最新的 `Image` 之外的 `Image` 。此功能大多数情况下比 `acquireNextImage` 更推荐使用，更加适用于视频实时处理。 需要注意的是 `maxImages` 应该至少为 2 ，因为丢弃除了最新的之外的所有帧需要至少两帧。换句话说，`(maxImages - currentAcquiredImages < 2)` 的情况下，丢帧将会不正常。
- `acquireNextImage()` 从 `ImageReader` 的队列中获取下一帧 `Image` ，如果没有新的则返回 `null`。 Android 推荐我们使用 `acquireLatestImage` 来代替使用此方法，因为它会自动帮我们 close 掉旧的 `Image`，并且能让效率比较差的情况下能获取到最新的 `Image` 。`acquireNextImage` 更推荐在批处理或者后台程序中使用，不恰当的使用本方法将会导致得到的 `images` 出现不断增长的延迟。
- `close()` 释放所有跟此 `ImageReader` 关联的资源。调用此方法后，`ImageReader` 不会再被使用，再调用它的方法或者调用被 `acquireLatestImage` 或 `acquireNextImage` 返回的 `Image` 会抛出 `IllegalStateException`，尝试读取之前 `Plane#getBuffer` 返回的 `ByteBuffers` 将会导致不可预测的行为。
- `newInstance(int width, int height, int format, int maxImages)` 创建新的 `reader` 以获取期望的 `size` 和 `format` 的 `Images`。`maxImages` 决定了 `ImageReader` 能同步返回的最大的 `Image` 的数量，申请越多的 `buffers` 会耗费越多的内存空间，使用合适的数量很重要。
	- `format` ：`reader` 生产的 `Image` 的格式，必须是 `ImageFormat` 或 `PixelFormat` 中的常量，并不是所有的 `formats` 都会被支持，比如 `ImageFormat.NV21` 就是不支持的，Android 一般都会支持 `ImageFormat_420_888`。那很多人可能会想，不支持你写这儿干嘛？当然这里只是说 Camera 不支持格式直出，并不是其他地方不认识这种格式，比如 `YuvImage` 就支持 `ImageFormat.NV21`。
	- `maxImages`：缓存的最大帧数，必须大于 0。






## 3、介绍一下 PBO（Pixel Buffer Object）？


**1）PBO 基础介绍。**


`OpenGL PBO（Pixel Buffer Object）`，被称为像素缓冲区对象，主要被用于异步像素传输操作。 `PBO` 仅用于执行像素传输，不连接到纹理，且与 `FBO` （帧缓冲区对象）无关。`OpenGL PBO`（像素缓冲区对象） 类似于 `VBO`（顶点缓冲区对象），`PBO` 开辟的也是 GPU 缓存，而存储的是图像数据。`PBO` 是 OpenGL ES 3.0 开始提供的一种方式，主要应用于从内存快速复制纹理到显存，或从显存复制像素数据到内存。

在使用 OpenGL 的时候经常需要在 GPU 和 CPU 之间传递数据，例如在使用 OpenGL 将 YUV 数据转换成 RGB 数据时就需要先将 YUV 数据上传到 GPU ，一般使用函数 `glTexImage2D` ,处理完毕后再将 RGB 结果数据读取到 CPU ， 这时使用函数 `glReadPixels` 即可将数据取回。但是这两个函数都是比较缓慢的，特别是在数据量比较大的时候。PBO 就是为了解决这个访问慢的问题而产生的。

不使用 PBO 加载纹理：

![](assets/resource/av-interview-qa/no-pbo.webp)


使用 PBO 加载纹理：

![](assets/resource/av-interview-qa/with-pbo.webp)






**2) PBO 如何使用？**


```c
int imgByteSize = m_Image.width * m_Image.height * 4;//RGBA

glGenBuffers(1, &uploadPboId);
glBindBuffer(GL_PIXEL_UNPACK_BUFFER, pboId);
glBufferData(GL_PIXEL_UNPACK_BUFFER, imgByteSize, 0, GL_STREAM_DRAW);

glGenBuffers(1, &downloadPboId);
glBindBuffer(GL_PIXEL_PACK_BUFFER, downloadPboId);
glBufferData(GL_PIXEL_PACK_BUFFER, imgByteSize, 0, GL_STREAM_DRAW);
```


使用两个 PBO 从帧缓冲区读回图像数据：

![](assets/resource/av-interview-qa/2-pbo.webp)


如上图所示，利用 2 个 PBO 从帧缓冲区读回图像数据，使用 `glReadPixels` 通知 GPU 将图像数据从帧缓冲区读回到 PBO1 中，同时 CPU 可以直接处理 PBO2 中的图像数据。

```c
// 交换 PBO
int index = m_FrameIndex % 2;
int nextIndex = (index + 1) % 2;

// 将图像数据从帧缓冲区读回到 PBO 中
glBindBuffer(GL_PIXEL_PACK_BUFFER, m_DownloadPboIds[index]);
glReadPixels(0, 0, m_RenderImage.width, m_RenderImage.height, GL_RGBA, GL_UNSIGNED_BYTE, nullptr);

// glMapBufferRange 获取 PBO 缓冲区指针
glBindBuffer(GL_PIXEL_PACK_BUFFER, m_DownloadPboIds[nextIndex]);
GLubyte *bufPtr = static_cast<GLubyte *>(glMapBufferRange(GL_PIXEL_PACK_BUFFER, 0,
                                                       dataSize,
                                                       GL_MAP_READ_BIT));
if (bufPtr) {
    nativeImage.ppPlane[0] = bufPtr;
    //NativeImageUtil::DumpNativeImage(&nativeImage, "/sdcard/DCIM", "PBO");
    glUnmapBuffer(GL_PIXEL_PACK_BUFFER);
}
glBindBuffer(GL_PIXEL_PACK_BUFFER, 0);
```







## 4、介绍一下 HardwareBuffer？




**1）HardwareBuffer 基础介绍**

`HardwareBuffer` 官方介绍为一种底层的内存 buffer 对象，可在不同进程间共享，可映射到不同硬件系统，如 GPU、传感器等，从构造函数可以看出，其可以指定 format 和 usage，用来让底层选择最合适的实现，目前 format 主要是渲染相关的纹理格式，Android 11 之后支持了 BLOB 格式，可用来做 NN 相关的数据共享。

如果看一下 `HardwareBuffer` 的实现，会发现其只是 `GraphicBuffer` 的一个包装，只是 Android 低版本并没有开放 `GraphicBuffer` 相关 API，而前面提到的 Surface ，其底层就是基于 `GraphicBuffer` 来实现的，因此本质上是 Android 系统开放了更底层的 API，我们才可以有更高效的实现，接下来看具体如何基于 `HardwareBuffer` 跨进程传输纹理。


![](assets/resource/av-interview-qa/hardware-buffer.webp)



**2）HardwareBuffer 如何使用？**

`AHardwareBuffer` 创建纹理：

```c
if(textureID == 0){
    AHardwareBuffer_Desc h_buffer_desc = {0};
    h_buffer_desc.stride = frameData->i32Width;
    h_buffer_desc.height = frameData->i32Height;
    h_buffer_desc.width = frameData->i32Width;
    h_buffer_desc.layers = 1;
    h_buffer_desc.format = 0x11;
    h_buffer_desc.usage = AHARDWAREBUFFER_USAGE_CPU_WRITE_OFTEN | AHARDWAREBUFFER_USAGE_GPU_SAMPLED_IMAGE;
 
    int ret = AHardwareBuffer_allocate(&h_buffer_desc, &inputHWBuffer);
    EGLint attr[] = {EGL_NONE};
    EGLDisplay edp;
    edp = (EGLDisplay)eglGetCurrentDisplay();
    inputEGLImage) = eglCreateImageKHR(edp, EGL_NO_CONTEXT, EGL_NATIVE_BUFFER_ANDROID, eglGetNativeClientBufferANDROID(inputHWBuffer), attr);
    glGenTextures(1, &textureID);
    glBindTexture(GL_TEXTURE_EXTERNAL_OES, textureID);
    glTexParameteri(GL_TEXTURE_EXTERNAL_OES , GL_TEXTURE_MIN_FILTER, GL_LINEAR);
    glTexParameteri(GL_TEXTURE_EXTERNAL_OES , GL_TEXTURE_MAG_FILTER, GL_LINEAR);
    glEGLImageTargetTexture2DOES(GL_TEXTURE_EXTERNAL_OES , (GLeglImageOES)inputEGLImage);
}
AHardwareBuffer_Planes planes_info = {0}; int ret = AHardwareBuffer_lockPlanes(inputHWBuffer, AHARDWAREBUFFER_USAGE_CPU_WRITE_MASK, -1,nullptr,&planes_info);
if (ret == 0) {
    memcpy(planes_info.planes[0].data,frameData->ppu8Plane[0],frameData->i32Width * frameData->i32Height*3/2);
    ret = AHardwareBuffer_unlock(inputHWBuffer, nullptr); 
}
glBindTexture(GL_TEXTURE_EXTERNAL_OES, textureID);

```


`AHardwareBuffer` 读取纹理图像数据：


```c
unsigned char *ptrReader = nullptr;
ret = AHardwareBuffer_lock(inputHWBuffer, AHARDWAREBUFFER_USAGE_CPU_READ_OFTEN, -1,     nullptr, (void **) &ptrReader); 
memcpy(dstBuffer, ptrReader, imgWidth * imgHeight * 3 / 2);
ret = AHardwareBuffer_unlock(inputHWBuffer, nullptr);
```

`ImageReader`、 `PBO` 和 `HardwareBuffer` 明显优于 `glReadPixels` 方式，`HardwareBuffer`、`ImageReader` 以及 `PBO` 三种方式性能相差不大，但是理论上 `HardwareBuffer` 性能最优。


---

**更多的音视频知识、面试题、技术方案干货可以进群来看：**

![微信扫码加入](assets/img/keyframe-zsxq.png)
_微信扫码加入_





---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
>
>你还可以加入我们的微信群：
>
>![关键帧的音视频开发群](assets/img/av-wechat-group.jpg){: w="600" }
>_微信扫码进群_
{: .prompt-tip }

