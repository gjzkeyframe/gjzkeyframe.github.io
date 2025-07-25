---
title: 音视频面试题集锦第 16 期
description: 持续更新的音视频面试题集锦。
author: Keyframe
date: 2025-02-17 03:58:18 +0800
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

下面是第 16 期面试题精选：


- **1、谈谈 iOS 音视频采集相关接口和数据结构的设计？**
- **2、如何降低处理音视频链路中的内存峰值？**
- **3、OpenGL 如何实现二分屏效果？**
- **4、使用 OpenGL 绘制时对于二维坐标需要注意什么？**



## 1、谈谈 iOS 音视频采集相关接口和数据结构的设计？


**1）整体框架**

通常我们通过 AVCaptureSession 相关的 API 来进行音视频的采集，其中主要组件分为 Input、Output、Session 几个部分：

- Input：AVCaptureDeviceInput，以 Device 作为输入，分为：视频采集设备、音频采集设备，可以同时添加多个 Input。
- Output：可以指定图片、视频文件、音视频裸帧数据等作为输出，可以同时添加多个 Output。
	- AVCaptureStillImageOutput 图片
	- AVCaptureMovieFileOutput 视频文件
	- AVCaptureAudioDataOutput 音频裸帧
	- AVCaptureVideoDataOutput 视频裸帧，目前支持三种格式的输出：
		- kCVPixelFormatType_32BGRA 
		- kCVPixelFormatType_420YpCbCr8BiPlanarVideoRange（nv12 420v）
		- kCVPixelFormatType_420YpCbCr8BiPlanarFullRange（nv12 420f）
- AVCaptureSession 主要负责管理各个 Input、Output 以及它们的内部链路组合。

可以通过如下代码获取视频输出支持的格式：

```c
AVCaptureVideoDataOutput *videoOut = [[AVCaptureVideoDataOutput alloc] init];
NSArray<NSNumber *> *types = [videoOut availableVideoCVPixelFormatTypes];
NSLog(@"type count: %ld", types.count); // 3 种
for (NSNumber *type in types) {
	NSLog(@"type: %@", type);
}
```


**2）视频采集**

对于视频采集，一般直接使用 AVCaptureSession 的 API 即可，需要注意的是：相机（前后置一样）吐出的视频帧，默认是横屏模式的 (横屏，Home 键在右边，也就是顺时针旋转 90 度就变成 Home 键在下边的正常竖屏状态)，跟 Android 略有差异。

视频采集时会有一个 10 多帧的缓存，当我们没有及时归还相机吐出的视频帧，导致采集吐帧的这个缓存空了，就会导致相机不吐帧。


**3）音频采集**

对于音频采集，除了可以使用 AVCaptureSession 来进行音频采集外，还可以使用 AudioUnit。

使用 AVCaptureSession 可以和视频采集在一起处理，也可以单独创建新的 AVCaptureSession 进行音频采集。

- 优点：由系统根据 AVAudioSession 自动处理音频采集的开始和暂停，也就是系统中断不用做特殊处理。
- 缺点：没有办法设置音频采样格式，所以在线路切换时，比如：从正常扬声器切到蓝牙耳机，采样率可能会发生变化，这是就要进行重采样，是采样率保持一致。


使用 AudioUnit 音频采集：

- 优点：更底层，更高效；在创建 unit 后，可以直接设置音频采集格式（如：通道数等）。
- 缺点：需要自己处理音频中断等情况。











## 2、如何降低处理音视频链路中的内存峰值？

音视频处理链路中的内存峰值一般是视频数据导致的，要降低内存峰值一般可以从两个方面入手：

- 降低采集参数：
	- 降低采集视频分辨率
	- 降低采集视频帧率
- 降低并发任务数量：
	- 将任务分优先级，按照优先级串行执行，这样既能降低内存峰值，也会降低 CPU 峰值





## 3、OpenGL 如何实现二分屏效果？

以纹理 y 坐标中间分屏为例，代码如下：

```c
precision highp float;
varying lowp vec2 varyTextCoord;
uniform sampler2D inputTexture;

void main()
{
    float y;
    if (varyTextCoord.y >= 0.0 && varyTextCoord.y <= 0.5) {
        y = varyTextCoord.y + 0.25;
    } else {
        y = varyTextCoord.y - 0.25;
    }
    gl_FragColor = texture2D(inputTexture, vec2(varyTextCoord.x, y));
}
```



## 4、使用 OpenGL 绘制时对于二维坐标需要注意什么？


**1）搞清楚顶点坐标与纹理坐标**

- 顶点坐标是左下角是 `(-1, -1)`，右上角是 `(1, 1)`；
- 纹理坐标是左下角是 `(0, 0)`，右上角是 `(1, 1)`，也就是 `(0, 0)` 对应图片的左下角，`(1, 1)` 对应着图片的右上角；
- 顶点坐标与纹理坐标一一对应，默认设置的纹理坐标是平铺满整个顶点坐标的，所以在设置 fill、fit 模式时，只用设置顶点坐标即可。


```c
GLfloat vertexes[] =
{
    -1.0, -1.0, //左下
    1.0, -1.0,  //右下
    -1.0, 1.0,  //左上
    1.0, 1.0    //右上
};

GLfloat textures[] = {
    0.0f, 0.0f, //左下
    1.0f, 0.0f, //右下
    0.0f, 1.0f, //左上
    1.0f, 1.0f  //右上
};
```

**2）FBO 与 glViewport**

首先绘制前 FBO 需要绑定了一个尺寸一致的 texture，绘制的内容会被绘制到这张 texture 上，这个就是 RTT，如果 FBO 为 0 则是屏幕绘制，否则是离屏绘制，可以将 FBO 看作画板，texture 看做这张画布。

一般我们会将 viewport 设置为: `(0, 0, FBO.width, FBO.height)`，这样绘制会占满整个 FBO，而顶点的的4个顶点是与 viewport 的4个顶点一一对应的，当然纹理也是一样对应的。

viewport 的 frame 和 FBO 不一致时，就会只在 viewport 的那块区域进行绘制对应的内容，也就是将输入的纹理在 viewport 的 frame 上进行绘制。



---

**更多的音视频知识、面试题、技术方案干货可以进群来看：**

![微信扫码加入](assets/img/keyframe-zsxq.png)
_微信扫码加入_











---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
{: .prompt-tip }

