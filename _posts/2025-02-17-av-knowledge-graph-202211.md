---
title: 音视频知识图谱 2022.11
description: 持续更新的音视频知识图谱。
author: Keyframe
date: 2025-02-17 08:58:08 +0800
categories: [音视频知识图谱]
tags: [音视频知识图谱, 音视频]
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


前些时间，我在知识星球上创建了一个音视频技术社群：**关键帧的音视频开发圈**，在这里群友们会一起做一些打卡任务。比如：周期性地整理音视频相关的面试题，汇集一份**音视频面试题集锦**，你可以看看这个合集：[音视频面试题集锦](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2380776196751425539#wechat_redirect)。再比如：循序渐进地归纳总结音视频技术知识，绘制一幅**音视频知识图谱**，你可以看看这个合集：[音视频知识图谱](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2349658423078092802#wechat_redirect)。

下面是 2022.11 月知识图谱新增的内容节选：

1）图谱路径：**渲染/AspectRatios**

- PAR（Pixel Aspect Ratio），单个像素的宽高比。大多数情况像素宽高比为 1:1，是一正方形像素。如果不是 1:1，则为长方形像素。常用的 PAR 有：1:1、10:11、40:33、16:11、12:11。
- SAR（Sample Aspect Ratio），采样纵横比。表示横向的像素点数和纵向的像素点数的比值，即我们通常提到的分辨率的宽高比。比如 VGA 图像 SAR 是 640/480=4:3，D-1 PAL 图像 720/576=5:4 等。
- DAR（Display Aspect Ratio），显示宽高比。即最终播放出来的画面的宽高比。比如常见的 16:9、4:3 等。缩放视频也要按这个比例来，否则会使图像看起来被拉伸了。
- 关系：PAR x SAR = DAR 或 PAR = DAR / SAR
- 播放器处理：播放器标准的播放流程，应该是先找视频容器格式也就是 container 中的 DAR，按这个比例来显示视频，进行播放；如果没有 DAR 的话，则使用 SAR 进行视频显示。
- 视频制式：视频制式按照录制设备可以分为计算机制式和电视制式。计算机制式的 PAR 常为 1:1，而电视制式的 PAR 通常不是 1:1，电视制式又分为 NTSC 或 PAL 制式，它们的 PAR 又可能不同。


2）图谱路径：**渲染/图像渲染技术/OpenGL**

- OpenGL：一套跨语言、跨平台，支持 2D、3D 图形渲染接口。这套接口由一系列的函数组成，定义了如何对简单及复杂的图形进行绘制。这套接口涉及到对设备的图像硬件进行调用，因此在不同的平台基于这套统一接口做了对应的实现。
- OpenGL ES：OpenGL 的子集，是针对手机和游戏主机等嵌入式设备而设计，去除了许多不必要和性能较低的 API 接口。
- OpenGL 在程序中角色：OpenGL 位于 GPU 驱动和平台图形绘制 API 之间；也可以直接使用用于图形绘制。驱动 GPU 芯⽚⾼效图形渲染。
- OpenGL 的渲染架构：OpenGL 的渲染架构是 Client/Server 模式。我们开发的过程就是不断用 Client 通过 OpenGL 提供的通道去向 Server 端传输渲染指令，来间接的操作 GPU 芯片。Client 向 Server 传递参数和渲染信息的通道：
	- Attribute（属性通道）：通常用来传递经常可变参数。
	- Uniform（统一变量通道）：通常用来传递不变的参数。
	- Texture Data（纹理通道）：专门用来传递纹理数据的通道。
- OpenGL 状态机：一系列的变量描述 OpenGL 此刻应当如何运行。OpenGL 的状态通常被称为 OpenGL 上下文（Context）。我们通常使用如下途径去更改 OpenGL 状态：设置选项，操作缓冲。最后，我们使用当前 OpenGL 上下文来渲染。
- OpenGL 图形渲染管线：顶点着色器 → 图元装配 → 几何着色器 → 光栅化 → 片段着色器 → 测试与混合
- EGL：OpenGL ES 渲染 API 和本地窗口系统之间的一个中间接口层，它主要由系统制造商实现。
	- 与设备的原生窗口系统通信；
	- 查询绘图图层的可用类型和配置；
	- 创建绘图图层；
	- 在 OpenGL ES 和其他图形渲染 API 之间同步渲染；
	- 管理纹理贴图等渲染资源。
	- Android EGL
		- Display 是对实际显示设备的抽象。在 Android 上的实现类是 EGLDisplay。
		- Surface 是对用来存储图像的内存区域 FrameBuffer 的抽象，包括 Color Buffer、Stencil Buffer、Depth Buffer。在 Android 上的实现类是 EGLSurface。
		- Context 存储 OpenGL ES 绘图的一些状态信息。在 Android 上的实现类是 EGLContext。
	- iOS EGL = EAGL（Embedded Apple Graphics Library）
		- OpenGL ES 系统与本地窗口（UIKit）系统的桥接由 EAGL 上下文系统实现。
		- 与 Android EGL 不同的是，iOS EAGL 不会让应用直接向 BackFrameBuffer 和 FrontFrameBuffer 进行绘制，也不会让应用直接控制双缓冲区的交换（swap），系统自己保留了这些操作权，以便可以随时使用 Core Animation 合成器来控制显示的最终外观。
- VBO、EBO 和 VAO
	- VBO（Vertex Buffer Objects）顶点缓冲区对象，指的是在 GPU 显存里面存储的顶点数据（位置、颜色）。
	- EBO/IBO（Element/Index Buffer Object）索引缓冲区对象，存储索引来达到减少重复数据。
	- VAO（Vertex Array Object）顶点数组对象。
	- VBO 和 EBO 的作用是在 GPU 显存中开辟一块存储空间来缓存顶点数据或者图元索引数据，避免每次绘制时 CPU 内存到 GPU 显存的数据拷贝，从而提升渲染性能。
	- VAO 的作用是管理 VBO 或 EBO，减少 glBindBuffer、glEnableVertexAttribArray、glVertexAttribPointer 这些调用操作，高效地实现在顶点数组配置之间切换。
- FBO：帧缓冲区对象 FBO（Frame Buffer Object）
	- 默认的帧缓冲区（Default Frame Buffer）：在建立了 OpenGL 的渲染环境后，我们相当于有了一只画笔和一块默认的画布，这块画布就是我们的屏幕，是一块默认的帧缓冲区（Default Frame Buffer）。
	- 离屏渲染：我们可以认为 OpenGL 的 FBO 就相当于是模拟了默认帧缓冲区的功能和结构创建了一种可以作为『画布』使用的 Object。从而支持离屏渲染。
	- 附着与附件：FBO 并不是一个真正的缓冲区，因为 OpenGL 并没有为它分配存储空间去存储渲染所需的几何、像素数据，它是一个指针的集合，这些指针指向了颜色缓冲区、深度缓冲区、模板缓冲区、累积缓冲区等这些真正的缓冲区对象，我们把这里的指向关系叫做『附着』。附着点类型有：颜色附着、深度附着和模板附着。这些附着点指向的缓冲区通常包含在某些对象里，我们把这些对象叫做『附件』。附件的类型有：纹理（Texture）或渲染缓冲区对象（Render Buffer Object，RBO）。

3）图谱路径：**渲染/伽马校正**

伽马校正的历史：

- 显示伽马（Display Gamma）
	- 人们在使用 CRT 时发现它有一个问题：调节电压为原来的 n 倍，对应的屏幕发光亮度并没有提高 n 倍，而是一个类似幂律曲线的关系。典型的 CRT 显示器产生的亮度约为输入电压的 2.2 次幂，这个就是『显示伽马』。
- 伽马校正（Gamma Correction）
	- 由于显示伽马问题的存在，为了使最终显示出来的图像亮度与捕捉到的真实场景的亮度是成线性比例关系，就需要在将图像输入到显示器之前对信号进行一个修正，这个修正过程就叫做『伽马校正』。
- 编码伽马（Encoding Gamma）
	- 修正显示伽马过程增加的伽马则叫做『编码伽马』。
	- 增加编码伽马通常是在图像采集设备的电路中完成的。
- 端到端伽马（End-to-End Gamma）
	- 编码伽马和显示伽马的乘积就是整个图像系统的『端到端伽马』。
	- 如果端到端伽马乘积为 1，那么显示出来的图像亮度与捕捉到的真实场景的亮度就是成线性比例的。
- 额外收益
	- 伽马校正的所做非线性转换过程除了解决显示伽马的问题外，还带来了一个额外收益：传输期间增加的噪声（模拟信号时代），在噪声比较明显的较暗信号区域（在接收器做了伽马校正后）会被减少。因为我们的视觉系统对相对亮度差别是敏感的，经过伽马校正后的非线性梯度明显对人眼感知来说更均匀。

伽马校正技术的延伸：

- sRGB 颜色空间
	- 2.2 是大多数 CRT 显示器的平均 Gamma 值。基于这个原因，1996 年，惠普与微软选择 Gamma 校准系数为 2.2 的颜色空间作为一种标准推出作为生成在因特网上浏览的图像的通用颜色空间，这就是 sRGB（Standard RGB）颜色空间，这是一个非线性的颜色空间。
	- sRGB 颜色空间得到了众多厂商支持，这样一来，遵循 sRGB 标准的图像处理都在这个非线性颜色空间中处理即可。
- LCD 显示器向前兼容显示伽马
	- LCD 显示器本身确实没有 CRT 显示器的伽马效应，但是为了兼容性，LCD 以及其他非 CRT 显示设备都模拟了这个伽马效应以实现先前兼容，甚至可以支持动态调节伽马参数。
- 光电转换函数设计目标面向人眼的特性而非显示伽马
	- 因为人眼对亮度感知是非线性的特点，我们可以用更多的码率来编码人眼敏感的中等亮度或暗部细节，从而使得编码在讨好人眼上有更好的 ROI。这样一来，我们在采集电路中采集到光信号向电信号转换时，通常会将其转换为非线性信号，以利于我们做编码，因此在传感数据上做伽马校正仍然是有用的。只是我们的伽马曲线参数要做调整了，曲线参数的目标不再是面向之前的 CRT 显示伽马，而是面向人眼的特性。这就有了后续的光电转换函数（Optical-Electro Transfer Function）和电光转换函数（Electro-Optical Transfer Function）。
	- PQ（Perceptual Quantizer，感知量化）曲线的设计更接近人眼的特点，亮度表达更准确。
	- HLG（Hybrid Log Gamma，混合对数伽马）曲线在低亮度区域基本与 Gamma 曲线重合，所以提供了与 SDR 显示设备很好的兼容性。
- 线性颜色空间仍有使用场景
	- 计算机视觉的一些图像处理场景，还是需要图像的亮度信息在线性颜色空间中才能进行处理，这时候则需要撤销伽马校正后再进行处理。在处理完成后，将图像输入显示器之前再重新做伽马校正。


4）图谱路径：**渲染/HDR**

- HDR 与 SDR 的区别：
	- SDR 支持的亮度范围在 0.1nit 到 100nit 之间，使用 Rec.709/sRGB 色域，并使用 Gamma 曲线来作为它的电光转换函数
	- HDR 支持更大的亮度范围（0.0005-10000nit）、更宽广的色域（BT.2020）、更高精度的量化（10bit 或 12bit），转换函数使用 PQ 或 HLG。
	- HDR 视频画面可以展现出更多的亮部和暗部细节，画面拥有丰富的色彩和生动自然的细节表现，因此画面更接近人眼所见。
- SDR 和 HDR 的转换函数：
	- BT.709 Gamma（SDR）
	- HLG（HDR）：HLG（Hybrid Log Gamma，混合对数伽马）曲线是另外一个重要的 HDR 转换函数曲线，由 BBC 和 NHK 公司开发。这个曲线与 PQ 曲线不同，HLG 规定的是 OETF 曲线，因为在低亮度区域基本与 Gamma 曲线重合，所以提供了与 SDR 显示设备很好的兼容性，在广播电视系统里有着广泛的应用。HLG 曲线最早在 ARIB STD-B67 中进行了标准化，后面也进入了 ITU-R BT.2100。
	- PQ-SMPTE ST2084（HDR）：PQ（Perceptual Quantizer，感知量化）曲线的设计更接近人眼的特点，亮度表达更准确。基于人眼的对比敏感度函数（Contrast Sensitivity Function，CSF），在 SMPTE ST 2084 标准中规定了 EOTF 曲线。亮度范围可从最暗 0.00005nit 到最亮 10000nit。PQ 曲线最早是由 Dolby 公司开发的，并且在 ST 2084 中进行了标准化。
- HDR 视频转 SDR 视频：
	- 1、HDR 非线性电信号转为 HDR 线性光信号（EOTF）
	- 2、HDR 线性光信号做颜色空间转换（Color Space Converting），通常是从 BT.2020 转换到 BT.709
	- 3、HDR 线性光信号色调映射为 SDR 线性光信号（Tone Mapping）
	- 4、SDR 线性光信号转 SDR 非线性电信号（OETF）



---

下面是 2022.11 月的知识图谱新增内容快照（图片被平台压缩不够清晰，可以加文章后面微信索要清晰原图）：

![2022.11 知识图谱新增内容](assets/resource/av-knowledge-graph/av-graph-add-202211.png)

---

如果你也对音视频技术感兴趣，比如，符合下面的情况：

- 在校大学生 → 学习音视频开发
- iOS/Android 客户端开发 → 转入音视频领域
- 直播/短视频业务开发 → 深入音视频底层 SDK 开发
- 音视频 SDK 开发 → 提升技能，解决优化瓶颈

可以长按识别或扫描下面二维码，了解一下这个社群，根据自己的情况按需加入：

![识别二维码加入我们](assets/img/keyframe-zsxq.png)
_识别二维码加入我们_





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

