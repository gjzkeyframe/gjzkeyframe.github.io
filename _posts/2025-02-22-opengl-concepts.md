---
title: OpenGL 基础概念：一篇文章入门 OpenGL
description: 介绍 OpenGL 渲染架构、状态机、图形渲染管线、EGL 等基础概念和知识。
author: Keyframe
date: 2025-02-22 08:30:08 +0800
categories: [音视频基础知识]
tags: [音视频基础知识, 音视频, 渲染, OpenGL]
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


这篇文章是[音视频基础](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2140155659944787969#wechat_redirect)专栏系列关于渲染的第一篇文章，我们来聊一聊 OpenGL，希望能做到让没接触过 OpenGL 的同学能比较容易的建立起一个初步的印象。

这篇文章的内容包括：

- **常见的移动端图形渲染技术**
- **OpenGL 在图形应用程序中的角色**
- **OpenGL 的渲染架构**
- **OpenGL 状态机思想**
- **OpenGL 的图形渲染管线**


## 1、移动端的图形渲染技术

提到移动设备的图形渲染，我们经常会听到 OpenGL、OpenGL ES、Metal、Vulkan 等方案，它们有什么差别呢？

- **OpenGL** 是一套跨语言、跨平台，支持 2D、3D 图形渲染**接口**。这套接口由一系列的函数组成，定义了如何对简单及复杂的图形进行绘制。这套接口涉及到对设备的图像硬件进行调用，因此在不同的平台基于这套统一接口做了对应的实现。
- **OpenGL ES** 是 OpenGL 的**子集**，是针对手机和游戏主机等嵌入式设备而设计，去除了许多不必要和性能较低的 API 接口。
- **Metal** 是苹果为了解决 3D 渲染性能问题而推出的框架，该技术将 3D 图形渲染性能提高了 10 倍。
- **Vulkan** 是一套新的跨平台支持 2D、3D 图形渲染的接口。Vulkan 针对全平台即时 3D 程序（如电子游戏和交互媒体）设计，并提供高性能与更均衡的 CPU/GPU 使用。


这些渲染方案之间还有着一定的**历史渊源**：

OpenGL 已经发展了 25 年以上，不断满足着行业需求。但是，随着 CPU、GPU 等硬件技术的发展和 3D 等更复杂场景对性能的需要，OpenGL 已经逐渐满足不了行业的需要了。后来在 2013 年，AMD 主导开发了 `Mantle`。Mantle 是面向 3D 游戏的新一代图形渲染接口，可以让开发人员直接操作 GPU 硬件底层，从而提高硬件利用率和游戏性能，效果显著。Mantle 很好的带动了图形行业发展，微软参考 AMD Mantle 的思路开发了 `DirectX 12`，苹果则提出了 `Metal`。但是因为 AMD 行业影响力和领导力不足，Mantle 没有发展成为全行业的标准。2015 年，AMD 宣布不在维护 Mantle，Mantle 功成身退。Khronos 接过 AMD 手中的接力棒，在 Mantle 的基础上推出了 `Vulkan`，Khronos 最先把 Vulkan API 称为『下一代 OpenGL 行动（glNext）』，但在正式宣布 Vulkan 之后这些名字就没有再使用了。

2014 年之前苹果一直是使用 OpenGL ES 来处理底层渲染，之后慢慢的把渲染框架迁移到了 Metal。到 iOS 12 苹果已经开始弃用 OpenGL，完全使用 Metal 实现底层渲染。不过 OpenGL 是跨平台的且相当稳定，目前 Metal 还只是用于苹果体系。

谷歌则是从 2016 年的 Android N（安卓 7.0）开始支持 Vulkan API。当然 OpenGL ES 也仍是持续支持的。

可以看到移动设备的渲染方案基本上都是从 OpenGL 的思想上继承和发展而来的，所以了解 OpenGL 就变得很有必要，我们接着往下讲。


## 2、OpenGL 的角色

要了解 OpenGL，首先可以看看它在一个应用程序中的**位置和角色**。

OpenGL 不能开发程序、构建后台，它只是一套处理图形图像的统一规则。它在一个图形应用程序中的角色大致如下图所示：

![OpenGL 在图形应用中的角色（iOS）](assets/resource/av-basic-knowledge/render-opengl-1.png)
_OpenGL 在图形应用中的角色（iOS）_

上图是基于 iOS 平台的，图中的 `Core Graphics`、`Core Animation`、`Core Image` 是 iOS 平台封装的绘制相关的上层 API，在 Android 平台则是其他的 API，这里不必深究。

在日常开发中，开发者一般通过使用上层 API 来构建和绘制界面，而调用 API 时系统最终还是通过 OpenGL/Metal/Vulkan 来实现视图的渲染。开发者也可以直接使用 OpenGL/Metal/Vulkan 来驱动 GPU 芯⽚⾼效渲染图形图像以满足一些特殊的需求。


## 3、OpenGL 的渲染架构

知道了 OpenGL 在整个应用程序中的定位和角色后，那它在内部是怎么实现串联上下游的呢？这就涉及到其**渲染架构**的设计了。

OpenGL 的渲染架构是 **Client/Server 模式**：Client（客户端）指的是我们在 CPU 上运行的一些代码，比如我们会编写 OC/C++/Java 代码调用 OpenGL 的一些 API；而 Server（服务端）则对应的是图形渲染管线，会调用 GPU 芯片。我们开发的过程就是不断用 Client 通过 OpenGL 提供的通道去向 Server 端传输渲染指令，来间接的操作 GPU 芯片。

![OpenGL 渲染架构](assets/resource/av-basic-knowledge/render-opengl-2.png)
_OpenGL 渲染架构_


渲染架构的 Client 和 Server 是怎么通信和交互的呢？这又涉及到 C/S **通道**的设计，下面我们来接着介绍，不过这里会提到一些你可能不太熟悉的名词，可以先不用深究，有个印象就可以了。

OpenGL 提供了 3 个通道来让我们从 Client 向 Server 中的顶点着色器（Vertex Shader）和片元着色器（Fragment Shader）传递参数和渲染信息，如下图所示：

![OpenGL 渲染架构及数据交互通道](assets/resource/av-basic-knowledge/render-opengl-3.png)
_OpenGL 渲染架构及数据交互通道_


这 3 个通道分别是：

- **Attribute（属性通道）**：通常用来传递经常可变参数。比如颜色数据、顶点数据、纹理坐标、光照法线这些变量。
- **Uniform（统一变量通道）**：通常用来传递不变的参数。比如变化矩阵。一个图形做旋转的时候，实质上是这个图形的所有顶点都做相应的变化，而这个变化的矩阵就是一个常量，可以用 Uniform 通道传递参数到顶点着色器的一个实例。再比如视频的颜色空间通常用 YUV，但是 YUV 颜色空间想要正常渲染到屏幕上面，需要转化成 RGBA 颜色空间，这个转换就需要把 YUV 的颜色值乘以一个转换矩阵转换为 RGBA 颜色值，这个转换矩阵也是一个常量，可以用 Uniform 通道传递参数到片元着色器的一个实例。
- **Texture Data（纹理通道）**：专门用来传递纹理数据的通道。

需要注意的是，这 3 个通道中 Uniform 通道和 Texture Data 通道都可以直接向顶点着色器和片元着色器传递参数，但是 Attribute 只能向顶点着色器传递参数，因为 OpenGL 架构在最初设计的时候，Attribute 属性通道就是顶点着色器的专用通道。片元着色器中是不可能有 Attribute 的，但是我们可以使用 GLSL 代码，通过顶点着色器把 Attribute 信息间接传递到片元着色器中。

另外，虽然 Texture Data 通道能直接向顶点着色器传递纹理数据，但是向顶点着色器传递纹理数据本身是没有实质作用的，因为顶点着色器并不处理太多关于纹理的计算，纹理更多是在片元着色器中进行计算。



参考：[了解 OpenGL 渲染架构](https://www.jianshu.com/p/51be4551d36f "了解 OpenGL 渲染架构")

## 4、OpenGL 状态机

在 Client/Server 的渲染架构下，OpenGL 的渲染流程其实是基于一个**状态机**来工作的。

我们先举个例子说明什么是状态机。我们都坐过电梯，一般来说电梯有这样几个状态：`开门`、`关门`、`运行（上升/下降）`、`静止`。

它们有什么特点呢?

电梯只有静止的时候才能开门，只有开门之后才能关门，只有关门之后才可以运动，只有运动之后才可以静止，所以，可以说电梯的各个状态是有依赖关系的，换种更专业的说法，就是各种状态可以通过有向图来表示。

![电梯状态图](assets/resource/av-basic-knowledge/render-opengl-4.png)
_电梯状态图_

电梯不能随意从一个状态跳转到另一个状态，比如：不能在运动过程中开门。


关于 OpenGL 状态机，[Learn OpenGL](https://learnopengl-cn.github.io/01%20Getting%20started/01%20OpenGL/ "Learn OpenGL") 中有概述：

>OpenGL 自身是一个巨大的状态机（State Machine）：一系列的变量描述 OpenGL 此刻应当如何运行。OpenGL 的状态通常被称为 OpenGL 上下文（Context）。我们通常使用如下途径去更改 OpenGL 状态：设置选项，操作缓冲。最后，我们使用当前 OpenGL 上下文来渲染。
>
>假设当我们想告诉 OpenGL 去画线段而不是三角形的时候，我们通过改变一些上下文变量来改变 OpenGL 状态，从而告诉 OpenGL 如何去绘图。一旦我们改变了 OpenGL 的状态为线段绘制模式，下一个绘制命令就会画出线段而不是三角形。
>
>当使用 OpenGL 的时候，我们会遇到一些状态设置函数（State-changing Function），这类函数将会改变上下文。以及状态使用函数（State-using Function），这类函数会根据当前 OpenGL 的状态执行一些操作。只要你记住 OpenGL 本质上是个大状态机，就能更容易理解它的大部分特性。


基于上面的理解，我们来看一段 OpenGL 的代码：

```c
unsigned int VBO, VAO; 
glGenVertexArrays(1, &VAO);
glGenBuffers(1, &VBO); 
glBindVertexArray(VAO); 
glBindBuffer(GL_ARRAY_BUFFER, VBO); 
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW); 
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0); 
glBindBuffer(GL_ARRAY_BUFFER, 0); 
glBindVertexArray(0);
```

初看这段代码，我们最深的印象可能是各种 `glBind...`，字面上是**绑定**的意思，如果从状态机的角度理解，其实 `glBind...` 就意味着**进入了某个状态**。

所以我们可以用状态图来表示上面的代码如下：

![示例代码状态图](assets/resource/av-basic-knowledge/render-opengl-5.png)
_示例代码状态图_

不过 OpenGL 的状态是可以**嵌套**的，所以细看上面的代码，我们还能看到这里状态存在包含关系，因为一个 VBO 会被绑定于一个 VAO 中，所以用下图来看会更加直观：

![状态嵌套示例](assets/resource/av-basic-knowledge/render-opengl-6.png)
_状态嵌套示例_

通俗来说就是，执行了绑定 X 到解绑 X 之间的任何操作，都会影响到 X。

明白了上面的状态机机制后，相信后面学习 OpenGL 的代码就能降低不少难度了。



参考：[OpenGL 工作机制](https://juejin.cn/post/7121525553491869703 "OpenGL 工作机制")




## 5、OpenGL 图形渲染管线

一个一个状态的切换以及在不同状态中的渲染逻辑和数据处理构成了 OpenGL 的**渲染管线**。

什么是管线？其实也可理解为一个**流程**。理解图像渲染管线前，我们可以想象一下如果让你在屏幕上绘制一个三角形，你要怎么做呢？

第一步，可能是先确定三角形三个顶点的位置：

![三角形绘制流程 1](assets/resource/av-basic-knowledge/render-opengl-7.png)
_三角形绘制流程 1_

第二步，自然是将三个点用线段连起来：

![三角形绘制流程 2](assets/resource/av-basic-knowledge/render-opengl-8.png)
_三角形绘制流程 2_

第三步，你可能觉得这样的三角形太过于单调，于是准备给三角形上色，因为是在屏幕上的，而屏幕本质用是一个个像素来显示颜色的，所以上色之前要先确定好哪些像素是属于三角形的，于是你叫计算机把属于三角形内部的像素一个个圈出来：

![三角形绘制流程 3](assets/resource/av-basic-knowledge/render-opengl-9.png)
_三角形绘制流程 3_

第四步，你想画一个带渐变色的炫酷三角形，所以需要给每个像素都上不同的颜色，于是你给一个个像素精心上色：

![三角形绘制流程 4](assets/resource/av-basic-knowledge/render-opengl-10.png)
_三角形绘制流程 4_

这样下来，一个漂亮的三角形就画出来了。回想这个过程，其实就像工厂的流水线一样，将整个工作拆解成一步一步实现即可。

OpenGL 的渲染管线其实也是类似的一个过程，它的工序包括：**顶点着色器 → 图元装配 → 几何着色器 → 光栅化 → 片段着色器 → 测试与混合**。

![OpenGL 渲染管线](assets/resource/av-basic-knowledge/render-opengl-11.png)
_OpenGL 渲染管线_

这些工序是将输入的 3D 的坐标，转化为显示在屏幕上的 2D 的像素的一个处理流程。


早期的 OpenGL 使用**立即渲染模式（Immediate Mode，也就是固定渲染管线）**。这种模式下绘制图形很方便，OpenGL 的大多数功能都被库隐藏起来，是一种**配置化（Configurable）**的管线，开发者很少有控制 OpenGL 如何进行计算的自由。而随着需求场景变的多样和复杂，开发者迫切希望能有更多的灵活性。随着时间推移，规范越来越灵活，开发者对绘图细节有了更多的掌控，现代 OpenGL 转变为**可编程（Programmable）**渲染管线，而这里的编程语言就是 `GLSL 语言`，它是一种类 C 的语言，专为图形计算量身定制，包含了一些针对向量和矩阵操作的有用特性，我们用它编写我们自己的顶点着色器和片段着色器。


上面的介绍中我们多次提到了一个词：**着色器（Shader）**，它是什么呢？

着色器就是一段运行在 GPU 中的程序，这段程序由开发者编写，所以说为开发者提供了很大的灵活度和可掌控度。现在 OpenGL 主要有三种着色器：**顶点着色器**、**几何着色器**、**片段着色器**，其中顶点着色器和片段着色器为开发者必须提供，几何着色器为可选提供。


下面我们介绍一下 OpenGL 渲染管线的几个重要工序：


**1）顶点着色器（Vertex Shader）**

顶点着色器主要用于**确定绘制图形的形状，以及接收开发者传入的数据并传给后面阶段**。接收外部传入的顶点数据，根据需要对顶点数据进行变换处理之后，再将顶点数据传入下一个阶段图元装配。另外顶点着色器也接收外部传进来的颜色值以及纹理采样器，然后再传递给下一个阶段进行图元装配处理。

每个顶点着色器只接收处理一个顶点坐标，有多少个顶点就会执行多少次。


**2）图元装配**

图元装配阶段是接收顶点着色器的输出数据，**将顶点着色器传来的顶点数据组装为图元**。就如上面画三角形中所说的将三角形三个顶点连接起来，具体连接方式需要开发者指定。所谓图元，指的就是点、线、三角形等最基本的几何图形，再复杂的图形也离不开这些基本图形的组成。另外，图元装配阶段还会将超出屏幕的顶点坐标进行裁剪，裁剪之后，顶点坐标被转化为屏幕坐标，之后将图元数据传递给管线的下一个阶段进行光栅化（几何着色器为非必须阶段，这里就暂时不讲了）。

下图是 OpenGL 支持的图元类型：

![OpenGL 图元类型](assets/resource/av-basic-knowledge/render-opengl-12.png)
_OpenGL 图元类型_


**3）光栅化**

拿到图元装配传递过来的图元数据，光栅化要做的就是**将一个图元转化为一张二维的图片**。而这张图片由若干个**片段（fragment）**组成（可以当做将这张图拆解为一个个类似屏幕上像素的小片段），片段可以近似看成像素，但是又略有不同，一个片段包含渲染该片段所需要的位置、颜色和深度的全部信息。光栅化完成之后，就把每个片段传给片段着色器。


**4）片段着色器（Fragment Shader）**

接下来的阶段是片段着色器，这是另外一个必须有的重要着色器，也是最后一个可以通过编程来控制屏幕是上显示颜色的阶段（后面的混合测试阶段还可以改变片段的颜色），在这个阶段主要是**计算片段的颜色**。这里每个片段着色器接收一个片段数据的输入，所以有几个片段就会执行所少次，根据具体需要灵活设置该片段的颜色。然后片段数据就被传递到下一个阶段：测试与混合。


**5）测试和混合**


这个阶段的测试是专门用来**丢弃一些不需要显示的片段**，其中测试主要包含**深度测试**和**模板测试**。


深度测试是在显示 3D 图形的时候，根据片段的深度来防止被阻挡的面渲染到其它面的前面。这里是 OpenGL 内部维护一个**深度缓冲**，保存这一帧中深度最小的片段的深度，然后对屏幕同一个位置的其他片段的深度再进行比较，深度比缓冲中大的片段则丢弃，直到找到深度最小的片段，就将其显示出来。

![深度测试](assets/resource/av-basic-knowledge/render-opengl-13.png)
_深度测试_

上图中每个方格表示一个片段，片段上的数值表示当前片段的深度，R 则表示深度无限，加号表示 2 个图形叠加一起，则由下面部分的图可知，当 2 个图形叠加在一起的时候，同一个位置的片段总是显示深度较小的那一个。


模板缓冲区是用于控制屏幕需要显示的内容，屏幕大小决定了模板缓冲区大小；模板测试基于**模板缓冲区**，从而让我们完成想要的效果。模板测试类似于**与运算**：

![模板测试](assets/resource/av-basic-knowledge/render-opengl-14.png)
_模板测试_


上图可以看出，模板就是每个片段位置有 0 也有 1，然后和缓冲中的图像数据对应片段进行类似与运算，也类似与拿一个遮罩罩住，只留下 1 的对应片段显示出来。



混合则是**计算带有透明度的片段的最终颜色**，在这个阶段会与显示在它背后的片段的颜色按照透明度进行叠加行成新的颜色，通俗讲就是形成透明物体的效果。

![混合](assets/resource/av-basic-knowledge/render-opengl-15.png)
_混合_

由图可以看出，通过混合，右边的窗户既有部分自己的颜色，又有窗户里面物体的部分颜色，就是两者透明度按照比例叠加的结果。

于是走完整个渲染管线流程，我们的渲染工作就算是告一段落了。


我们再来回顾一下这条**渲染管线**做了哪些事情：

首先我们传入了图形的**顶点数据**，然后 OpenGL 内部会按照指定的**图元类型**自动将顶点连成图形，然后再将图形内的区域切成一个个**小片段**，然后给每个小片段自由**上色**，最后把被挡住的或者我们不想显示的区域的下片段**丢弃**，并且对有透明度的片段进行前后片段颜色的**混合**。


参考：[图形渲染管线的那些事](https://juejin.cn/post/7119135465302654984 "图形渲染管线的那些事")


到此，我们基本上就对 OpenGL 有个初步的认识了，至于更细节的知识则需要在实践中去学习和领悟了。


<!-- sync -->



## 6、EGL

了解了 OpenGL 的渲染管线，我们接着来看看它如何在设备上实现渲染。

我们这里只讨论 iOS/Android 设备，所以这里的 OpenGL 也对应的是 OpenGL ES。

如果我们了解了 OpenGL ES 就会知道，虽然它定义了一套移动设备的图像渲染 API，但是并没有定义窗口系统。为了让 GLES 能够适配各种平台，GLES 需要与知道如何通过操作系统创建和访问窗口的库结合使用，这就有了 **EGL**，EGL 是 OpenGL ES 渲染 API 和本地窗口系统之间的一个中间接口层，它主要由系统制造商实现。EGL 提供如下机制：

- 与设备的原生窗口系统通信；
- 查询绘图图层的可用类型和配置；
- 创建绘图图层；
- 在 OpenGL ES 和其他图形渲染 API 之间同步渲染；
- 管理纹理贴图等渲染资源。

**EGL 是 OpenGL ES 与设备的桥梁，以实现让 OpenGL ES 能够在当前设备上进行绘制。**

![EGL 架构](assets/resource/av-basic-knowledge/render-opengl-17.png)
_EGL 架构_


### 6.1、Android EGL

Android 平台自 2.0 版本之后图形系统的底层渲染均由 OpenGL ES 负责，其 EGL 架构实现如下图所示：

![Android EGL 架构](assets/resource/av-basic-knowledge/render-opengl-16.png)
_Android EGL 架构_


- **Display** 是对实际显示设备的抽象。在 Android 上的实现类是 `EGLDisplay`。
- **Surface** 是对用来存储图像的内存区域 FrameBuffer 的抽象，包括 Color Buffer、Stencil Buffer、Depth Buffer。在 Android 上的实现类是 `EGLSurface`。
- **Context** 存储 OpenGL ES 绘图的一些状态信息。在 Android 上的实现类是 `EGLContext`。

本地窗口相关的 API 提供了访问本地窗口系统的接口，而 EGL 可以创建渲染表面 EGLSurface ，同时提供了图形渲染上下文 EGLContext，用来进行状态管理，接下来 OpenGL ES 就可以在这个渲染表面上绘制。

使用 EGL 在平台实现渲染步骤大致如下：

- 1）调用 `eglGetDisplay` 来获得 EGLDisplay 对象，从而**建立与平台窗口系统的联系**，这个 EGLDisplay 将作为 OpenGL ES 的渲染目标；
- 2）调用 `eglInitialize` **初始化 EGL**；
- 3）调用 `eglChooseConfig` 来获得 EGLConfig 对象，从而**确定渲染表面的配置信息**；
- 4）通过 EGLDisplay 和 EGLConfig 对象，调用 `eglCreateWindowSurface` 或 `eglCreatePbufferSurface` 方法得到 EGLSurface，从而**创建渲染表面**，其中 eglCreateWindowSurface 用于创建屏幕上渲染区域，eglCreatePbufferSurface 用于创建离屏渲染区域；
- 5）通过 EGLDisplay 和 EGLConfig，调用 `eglCreateContext` 获得 EGLContext 对象，从而**创建渲染上下文**，OpenGL 的任何一条指令都是必须在自己的 OpenGL 上下文环境中执行；
- 6）调用 `eglMakeCurrent` 将 EGLSurface、EGLContext、EGLDisplay 三者绑定就**完成了上下文绑定**，绑定成功之后 OpenGL ES 的环境就创建好了，接下来就可以开始渲染了；

>通过上面的步骤就做好了 EGL 的准备工作：一方面为 OpenGL ES 渲染提供了目标 EGLDisplay 及上下文环境 EGLContext，可以接收到 OpenGl ES 渲染出来的纹理；另一方面我们连接好了设备显示屏 EGLSurface（这里可能是 SurfaceView 或者 TextureView）。接下来，由于 OpenGL ES 的渲染必须新开一个线程，并为该线程绑定显示设备及上下文环境（EGLContext），所以 `eglMakeCurrent()` 就是来绑定该线程的显示设备及上下文的。

- 7）OpenGL ES 完成绘制后，调用 `eglSwapBuffers` 方法**交换前后缓冲，将绘制内容显示到屏幕上**，而离屏渲染不需要调用此方法；

>这里需要注意的是 EGL 的工作模式是**双缓冲模式**，其内部有两个 FrameBuffer（帧缓冲区）：BackFrameBuffer 和 FrontFrameBuffer，当 EGL 将一个 FrameBuffer 显示到屏幕上的时候，另一个 FrameBuffer 就在后台等待 OpenGL ES 进行渲染输出。
>
>直到调用了 `eglSwapBuffer()` 这条指令的时候，才会把前台的 FrameBuffer 和后台的 FrameBuffer 进行交换，这时界面呈现的就是 OpenGL ES 刚刚渲染的内容了。
>
>这样做的原因是如果应用程序使用单缓冲绘图时可能会存在图像闪烁的问题，因为图像生成不是一下子被绘制出来的，而是按照从左到右、从上到下逐像素绘制的。如果最终图像不是在瞬间全部展示给用户，而是通过把绘制过程也展示出来了，这会导致用户看到的渲染效果出现闪烁。为了规避这个问题，可以使用双缓冲渲染：前缓冲保存着最终输出的图像，它会在屏幕上显示；而所有的的渲染指令都会在后缓冲上绘制，对用户屏蔽从左到右、从上到下逐像素绘制的过程，这样就可以避免闪烁了。

- 8）绘制结束后，不再需要使用 EGL 时，需要调用 `eglMakeCurrent` 取消绑定，调用 `eglDestroyContext`、`eglDestroySurface`、`eglTerminate` 等函数销毁 EGLDisplay、EGLSurface、EGLContext 三个对象。


>在[《RenderDemo（1）：用 OpenGL 画一个三角形》](https://mp.weixin.qq.com/s/b-nFCBMf-oaayyG8a86mgw) Android Demo 的 `KFGLContext` 类中就可以看到上面这套流程。
>
>不过，如果你觉得上述配置 EGL 的流程太麻烦的话，Android 平台提供了 `GLSurfaceView` 类实现了 Display、Surface、Context 的管理，即 GLSurfaceView 内部实现了对 EGL 的封装，可以很方便地利用接口 GLSurfaceView.Renderer 的实现，使用 OpenGL ES API 进行渲染绘制。GLSurfaceView 提升了 OpenGL ES 开发的便利性，当然也相应的失去了一些灵活性。


参考：

- [EGL 作用及其使用](https://blog.csdn.net/u010281924/article/details/105296617 "EGL 作用及其使用")
- [EGL](https://blog.csdn.net/Kennethdroid/article/details/99655635 "EGL")





### 6.2、iOS EAGL

iOS 平台对 EGL 的实现是 **EAGL（Embedded Apple Graphics Library）**。OpenGL ES 系统与本地窗口（UIKit）系统的桥接由 EAGL 上下文系统实现。

与 Android EGL 不同的是，iOS EAGL 不会让应用直接向 BackFrameBuffer 和 FrontFrameBuffer 进行绘制，也不会让应用直接控制双缓冲区的交换（swap），系统自己保留了这些操作权，以便可以随时使用 Core Animation 合成器来控制显示的最终外观。

**Core Animation 是 iOS 上图形渲染和动画的核心基础架构。**可以使用托管多种 iOS 系统内容的图层（UIKit、Quartz 2D、OpenGL ES），来合成应用的用户界面或者其他视觉显示。

OpenGL ES 通过 CAEAGLLayer 与 Core Animation 连接，CAEAGLLayer 是一种特殊类型的 Core Animation 图层，它的内容来自 OpenGL ES 的 RenderBuffer，Core Animation 将 RenderBuffer 的内容与其他图层合成，并在屏幕上显示生成的图像。所以同一时刻可以有任意数量的层。Core Animation 合成器会联合这些层并在后帧缓存中产生最终的像素颜色，然后切换缓存。

![Core Animation 与 OpenGL ES 共享 RenderBuffer](assets/resource/av-basic-knowledge/render-opengl-18.png)
_Core Animation 与 OpenGL ES 共享 RenderBuffer_


一个应用提供的图层与操作系统提供的图层混合起来可以产生最终的显示外观。如下图所示，OpenGL ES 图层显示了一个应用生成的旋转立方体，但是在显示器顶部的显示状态栏图层则是由操作系统生成和控制的，此图显示的是合并两个图层来产生后帧缓存中的颜色数据的过程，交换后，我们看到的就是前帧缓存上的内容。

![iOS 多图层合成](assets/resource/av-basic-knowledge/render-opengl-19.png)
_iOS 多图层合成_


所以，iOS 的 EAGL 配置过程其实就是使用 CoreAnimation 的 layer 来支持 OpenGL ES 渲染的过程，步骤大致如下：

- 1）创建一个 EAGL 图层 `CAEAGLLayer` 对象，并设置好它的属性；
- 2）创建 OpenGL ES 上下文 `EAGLContext`，并设置为当前上下文环境；
- 3）创建一个颜色渲染缓冲区对象 `ColorRenderBuffer`，并调用 `renderbufferStorage:fromDrawable:` 为其分配存储空间，这里其实是将 CAEAGLLayer 的绘制存储区共享为了 ColorRenderBuffer 的绘制缓冲区。分配缓冲区需要的宽、高、像素格式等信息都会从 layer 中取得；

>需要注意的是，如果 CAEAGLLayer 的 bounds 或其他属性变了，需要重新分配 ColorRenderBuffer 的存储空间，否则会出现 ColorRenderBuffer 和 CAEAGLLayer 的尺寸不匹配。

- 4）创建帧缓冲区 `FrameBuffer` 对象，并将 ColorRenderBuffer 绑定为它的附件；
- 5）从颜色渲染缓冲区 ColorRenderBuffer 获取宽高信息；
- 6）根据需要创建一个深度渲染缓冲区 `DepthRenderBuffer` 对象，并绑定为 FrameBuffer 的附件；
- 7）根据需要检测 FrameBuffer 的状态；
- 8）将 CAEAGLLayer 添加到 Core Animation 的图层树中；
- 9）在绘制动作完成后，调用 EAGLContext 的 `presentRenderbuffer:` 方法，就可以将绘制结果显示在屏幕上了。

>在[《RenderDemo（1）：用 OpenGL 画一个三角形》](https://mp.weixin.qq.com/s/b-nFCBMf-oaayyG8a86mgw) iOS Demo 的 `DMTriangleRenderView` 类中可以看到类似的流程，只不过 Demo 中我们是创建了一个 UIView 的子类，并重写它的 `+layerClass` 方法返回 `CAEAGLLayer` 类型来获得了一个 CAEAGLLayer 对象用于 OpenGL ES 渲染。
>
>同样的，如果你觉得上述流程太麻烦，iOS 平台还提供了封装好的 `GLKView` 来简化我们使用 OpenGL ES，GLKView 是对 CAEAGLLayer 的封装，内嵌了配置 Core Animation 支持 OpenGL ES 的流程。


参考：

- [iOS OpenGL ES 应用开发实践指南](https://book.douban.com/subject/24849591/ "iOS OpenGL ES 应用开发实践指南")
- [iOS OpenGL ES Programming Guide](https://developer.apple.com/library/archive/documentation/3DDrawing/Conceptual/OpenGLES_ProgrammingGuide/WorkingwithEAGLContexts/WorkingwithEAGLContexts.html "iOS OpenGL ES Programming Guide")
- [OpenGL ES 在 iOS 中的上下文环境搭建](https://www.jianshu.com/p/c34c14589e0c "OpenGL ES 在 iOS 中的上下文环境搭建")


## 7、VBO、EBO 和 VAO

当我们开始上手写 OpenGL 的程序了，我们就要开始逐渐接触 VBO、EBO、VAO 了。先初步看看概念：

- **VBO（Vertex Buffer Objects）顶点缓冲区对象**，指的是在 GPU 显存里面存储的顶点数据（位置、颜色）。
- **EBO/IBO（Element/Index Buffer Object）索引缓冲区对象**，指的是为了更高效的利用数据，存储索引来达到减少重复数据的索引数据。
- **VAO（Vertex Array Object）顶点数组对象**，主要作用是用于管理 VBO 或 EBO，减少 `glBindBuffer`、`glEnableVertexAttribArray`、`glVertexAttribPointer` 这些调用操作，高效地实现在顶点数组配置之间切换。

### 7.1、VBO 和 EBO

在 OpenGL 开发中，用于绘制的顶点数据首先是存储在 CPU 内存中的，比如我们在[《RenderDemo（1）：用 OpenGL 画一个三角形》](https://mp.weixin.qq.com/s/b-nFCBMf-oaayyG8a86mgw)中的三角形的 3 个顶点数据。而在调用 glDrawArrays 或者 glDrawElements 等接口进行绘制时，OpenGL 需要将顶点数组数据从 CPU 内存拷贝到 GPU 显存。如果我们的程序里需要多次绘制，那就会触发多次内存拷贝从而带来一些性能消耗。

如果我们可以在 GPU 显存中缓存这些顶点数据，就可以大幅减少 CPU 内存到 GPU 显存的数据拷贝的开销，这就是 VBO 和 EBO 出现的原因。**VBO 和 EBO 的作用是在 GPU 显存中开辟一块存储空间来缓存顶点数据或者图元索引数据，避免每次绘制时 CPU 内存到 GPU 显存的数据拷贝，从而提升渲染性能。**

那 VBO 和 EBO 有什么区别呢？

1）我们先来看看 VBO 的使用：

```c
// 顶点数据：
GLfloat vertices[] = {
    -0.5f, -0.5f, 0.0f,
     0.5f, -0.5f, 0.0f,
     0.0f,  0.5f, 0.0f
};

// 使用 VBO：
GLuint VBO;
glGenBuffers(1, &VBO); // 创建 VBO 对象
glBindBuffer(GL_ARRAY_BUFFER, VBO); // 把新创建的 VBO 绑定到 GL_ARRAY_BUFFER 目标上，同时也绑定到了 OpenGL 渲染管线上
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW); // 将顶点数据 (CPU 内存) 拷贝到 VBO（GPU 显存）

// 绘制：
glDrawArrays(GL_TRIANGLES, 0, 3); // 使用 glDrawArrays 来绘制
```

整个过程还是比较浅显易懂的：**做了一次 CPU 到 GPU 的数据拷贝。**

>在[《RenderDemo（1）：用 OpenGL 画一个三角形》](https://mp.weixin.qq.com/s/b-nFCBMf-oaayyG8a86mgw)的 iOS Demo 中我们用到了 VBO。

2）我们接着来看看 EBO 的使用：

假设我们不再绘制一个三角形而是绘制一个矩形。我们可以绘制两个三角形来组成一个矩形（OpenGL 主要处理三角形）。这会生成下面的顶点的集合：

```c
GLfloat vertices[] = {
    // 第一个三角形
     0.5f,  0.5f, 0.0f, // 右上角
     0.5f, -0.5f, 0.0f, // 右下角
    -0.5f,  0.5f, 0.0f, // 左上角
    // 第二个三角形
     0.5f, -0.5f, 0.0f, // 右下角
    -0.5f, -0.5f, 0.0f, // 左下角
    -0.5f,  0.5f, 0.0f  // 左上角
};
```

可以看到，有几个顶点叠加了。我们指定了`右下角`和`左上角`两次！一个矩形只有 4 个而不是 6 个顶点，这样就产生 50% 的额外开销。当我们有包括上千个三角形的模型之后这个问题会更糟糕，这会产生一大堆浪费。更好的解决方案是只储存不同的顶点，并设定绘制这些顶点的顺序。这样子我们只要储存 4 个顶点就能绘制矩形了，之后只要指定绘制的顺序就行了。EBO 就是来优化这个问题的：

```c
// 这次我们只定义了 4 个顶点：
GLfloat vertices[] = {
     0.5f,  0.5f, 0.0f, // 右上角
     0.5f, -0.5f, 0.0f, // 右下角
    -0.5f, -0.5f, 0.0f, // 左下角
    -0.5f,  0.5f, 0.0f  // 左上角
};

// 但是通过索引指定了每个三角形的 3 个顶点：
GLuint indices[] = { // 注意索引从 0 开始! 
    0, 1, 3, // 第一个三角形
    1, 2, 3  // 第二个三角形
};

// 使用 VBO：
GLuint VBO;
glGenBuffers(1, &VBO); // 创建 VBO 对象
glBindBuffer(GL_ARRAY_BUFFER, VBO); // 把新创建的 VBO 绑定到 GL_ARRAY_BUFFER 目标上，同时也绑定到了 OpenGL 渲染管线上
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW); // 将顶点数据 (CPU 内存) 拷贝到 VBO（GPU 显存）

// 使用 EBO：
GLuint EBO;
glGenBuffers(1, &EBO); // 创建 EBO 对象
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO); // 把新创建的 EBO 绑定到 GL_ELEMENT_ARRAY_BUFFER 目标上，同时也绑定到了 OpenGL 渲染管线上
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW); // 将顶点数据 (CPU 内存) 拷贝到 EBO（GPU 显存）

// 绘制：
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0); // 使用 glDrawElements 来绘制
```

整个过程比 VBO 略复杂了一点，但是还是很好理解的：**去掉重复顶点，通过索引指定绘制顶点，创建 VBO 做一次顶点数据拷贝，创建 EBO 做了一次索引数据拷贝。**






### 7.2、VAO

通过对 VBO、EBO 的使用，我们可以减少 CPU 到 GPU 内存拷贝来提高性能，但是如果我们需要绘制大量的顶点和物体时，每次绘制都需要绑定正确的缓冲对象并为每个物体配置所有顶点属性，这样一大堆操作很是麻烦。是否可以用一种对象来储存这些状态配置，使得我们需要的时候直接绑定这个对象就可以切换到正确的状态呢？这就是 **VAO** 要解决的问题。


如果说 VBO、EBO 是通过 GPU 显存的缓存来减少内存拷贝从而提升性能，那么 VAO 则略有不同：**VAO 的主要作用是用于管理 VBO 或 EBO，减少 `glBindBuffer`、`glEnableVertexAttribArray`、`glVertexAttribPointer` 这些调用操作，高效地实现在顶点数组配置之间切换。**VAO 如何实现这些能力呢？

```c
// 顶点数据：
GLfloat vertices[] = {
    -0.5f, -0.5f, 0.0f,
     0.5f, -0.5f, 0.0f,
     0.0f,  0.5f, 0.0f
};

// 创建 VBO：
GLuint VBO;
glGenBuffers(1, &VBO); // 创建 VBO 对象

// 创建 VAO：
GLuint VAO;
glGenVertexArrays(1, &VAO); // 创建 VAO 对象，注意这里用的是 glGenVertexArrays

// 在绑定 VAO 后操作 VBO，当前 VAO 会记录 VBO 的操作，我们下面用缩进表示操作 VBO 的代码：
glBindVertexArray(VAO); // 绑定 VAO，注意这里用的是 glBindVertexArray
    // 绑定 VBO
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    // 把顶点数组复制到缓冲中供 OpenGL 使用
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW); 
    // 设置顶点属性指针
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(GLfloat), (GLvoid *) 0);
    glEnableVertexAttribArray(0);
// 解绑 VAO
glBindVertexArray(0);

// ...省略其他代码...

// 会被调用多次的绘制代码：
glBindVertexArray(VAO); // 绑定使用 VAO 绘制
glDrawArrays(GL_TRIANGLES, 0, 3); // 使用 glDrawArrays 来绘制
glBindVertexArray(0); // 解绑 VAO
```

上面的代码相比我们用 VBO 绘制三角形的代码还是复杂一些的，上面的代码可以理解为：**使用 VAO 记录 VBO 的操作相当于创建了一个快捷方式，后面直接用 VAO 快捷方式绘制。**


### 7.3、VBO、EBO 和 VAO 内存布局

上面我们介绍了 VBO、EBO 和 VAO 的使用，大致知道了它们的作用，我们继续来看看使用它们时的内存布局来加深一下印象：

当我们的 Vertex Shader 如下：

```sh
#version 330 core
layout (location = 0) in vec3 position; // 位置变量的属性位置值为 0 
layout (location = 1) in vec3 color;    // 颜色变量的属性位置值为 1

out vec3 ourColor; // 向片段着色器输出一个颜色

void main()
{
    gl_Position = vec4(position, 1.0);
    ourColor = color; // 将 ourColor 设置为我们从顶点数据那里得到的输入颜色
}
```

Fragment Shader 如下：

```sh
#version 330 core
in vec3 ourColor;
out vec4 color;

void main()
{
    color = vec4(ourColor, 1.0f);
}

```

创建 VBO 的代码如下：

```c
GLfloat vertices[] = {
    // 位置               // 颜色
     0.5f, -0.5f, 0.0f,  1.0f, 0.0f, 0.0f,   // 右下
    -0.5f, -0.5f, 0.0f,  0.0f, 1.0f, 0.0f,   // 左下
     0.0f,  0.5f, 0.0f,  0.0f, 0.0f, 1.0f    // 顶部
};

GLuint VBO;
glGenBuffers(1, &VBO);  
glBindBuffer(GL_ARRAY_BUFFER, VBO);  
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
```

这时候对应的 VBO 布局格式如下图所示：

![VBO 布局格式](assets/resource/av-basic-knowledge/render-opengl-20.png)
_VBO 布局格式_


当 VAO 只管理 VBO 时，布局格式如下图所示：

![VAO 管理 VBO 布局格式](assets/resource/av-basic-knowledge/render-opengl-21.png)
_VAO 管理 VBO 布局格式_

当 VAO 管理 VBO 和 EBO 时，布局格式如下图所示：

![VAO 管理 VBO 和 EBO 布局格式](assets/resource/av-basic-knowledge/render-opengl-22.png)
_VAO 管理 VBO 和 EBO 布局格式_




参考：

- [Learn OpenGL](https://learnopengl-cn.readthedocs.io/zh/latest/01%20Getting%20started/04%20Hello%20Triangle/#_10 "LearnOpenGL")
- [VBO、EBO 和 VAO](https://blog.csdn.net/Kennethdroid/article/details/98088890 "VBO、EBO 和 VAO")





## 8、FBO

上面我们介绍了通过 VBO、EBO 和 VAO 管理渲染过程中的数据来优化渲染性能，接下来我们来介绍另一个重要的 XXO：**帧缓冲区对象 FBO（Frame Buffer Object）**。

FBO 是用来做什么的呢？

在建立了 OpenGL 的渲染环境后，我们相当于有了一只画笔和一块默认的画布，这块画布就是我们的屏幕，是一块默认的帧缓冲区（Default Frame Buffer）。我们渲染的目的地是我们的屏幕，我们画出来的东西会显示在屏幕上。这个默认的帧缓冲区是与一系列缓冲区相关联的，具体有哪些缓冲区，多少位的缓冲区，是建立 OpenGL Context 的时候用户自定义的。一般来讲，必要的是颜色缓冲区和深度缓冲区，模板缓冲区、累加缓冲区是可选的。

后来随着新需求的需要，离屏渲染（Off-screen Render）技术开始出现，相较于直接渲染到屏幕，离屏渲染是先把物体绘制到『其他地方』而非屏幕上，而 OpenGL 则在某个版本引入了 FBO 可以支持离屏渲染。我们可以认为 OpenGL 的 FBO 就相当于是模拟了默认帧缓冲区的功能和结构创建了一种可以作为『画布』使用的 Object。也就是说，你可以把你想渲染的东西渲染到你生成的 FBO 里，而不是直接渲染到屏幕上。上面说的默认帧缓冲区关联的一系列其他缓冲区，FBO 也是可以有的，只是需要我们自己去创建、设置和绑定。



FBO 虽然也叫缓冲区对象，但是它并不是一个真正的缓冲区，因为 OpenGL 并没有为它分配存储空间去存储渲染所需的几何、像素数据，我们可以认为它是一个指针的集合，这些指针指向了颜色缓冲区、深度缓冲区、模板缓冲区、累积缓冲区等这些真正的缓冲区对象，我们把这里的『指向关系』叫做**附着**，而 FBO 中的附着点类型有：**颜色附着**、**深度附着**和**模板附着**。这些附着点指向的缓冲区通常包含在某些对象里，我们把这些对象叫做**附件**，附件的类型有：**纹理（Texture）**或**渲染缓冲区对象（Render Buffer Object，RBO）**。

![FBO 的附件和附着点](assets/resource/av-basic-knowledge/render-opengl-23.jpg)
_FBO 的附件和附着点_


- **纹理（Texture）**是一个可以往上绘制细节的 2D 图片（甚至也有 1D 和 3D 的纹理），你可以想象纹理是一张绘有砖块的纸，无缝折叠贴合到你的 3D 的房子上，这样你的房子看起来就像有砖墙外表一样了。除了图像以外，纹理也可以被用来储存大量的数据，这些数据可以发送到着色器上进行计算和处理。
- **渲染缓冲区对象（Render Buffer Object，RBO）**则是一个由应用程序分配的 2D 图像缓冲区，可以分配和存储颜色、深度或者模板值，可以用作 FBO 中的颜色、深度或者模板附着。


FBO 是 OpenGL 渲染管线的最终目标，但其实 FBO 本身不直接用于渲染，而是要为其绑定好附件后才能作为渲染目标。所以，建构一个完整的 FBO 需要满足下列条件：

- 必须往 FBO 里面加入至少一个附件（颜色、深度、模板缓冲）；
- 其中至少有一个是颜色附件；
- 所有的附件都应该是已经完全做好的（已经存储在内存之中）；
- 每个缓冲都应该有同样数目的样本。





**1）使用纹理附件**

当把一个纹理（Texture）附加到 FBO 上的时候，所有渲染命令会写入到纹理上，就像它是一个普通的颜色/深度或者模板缓冲一样。使用纹理的好处是，所有渲染操作的结果都会被储存为一个纹理图像，这样我们就可以简单的在着色器中使用了。

下面是一个简单的使用纹理附件的例子：

```c
// 创建和绑定 FBO：
GLuint fbo;
glGenFramebuffers(1, &fbo); // 创建 FBO
glBindFramebuffer(GL_FRAMEBUFFER, fbo); // 绑定 FBO，注意：如果这里用 glBindFramebuffer(GL_FRAMEBUFFER, 0) 则是激活默认的帧缓冲区

// 创建纹理：
GLuint texture;
glGenTextures(1, &texture);
glBindTexture(GL_TEXTURE_2D, texture);
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, m_width, m_height, 0, GL_RGBA, GL_UNSIGNED_BYTE, NULL); // 创建纹理和分配存储空间。传入 NULL 作为纹理的 data 参数，不填充数据，填充纹理数据会在渲染到 FBO 时去做。
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR );
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
glBindTexture(GL_TEXTURE_2D, 0);

// 将纹理添加为 FBO 的附件，连接在颜色附着点：
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, texture, 0);

// 检测 FBO：
GLenum status = glCheckFramebufferStatus(GL_FRAMEBUFFER);
if (status != GL_FRAMEBUFFER_COMPLETE)
    printf("Frame buffer incomplete!\n");
else
    printf("Frame buffer complete!\n");

// ...省略其他代码...
```



**2）使用 RBO 附件**

下面是一个简单的使用 RBO 附件的例子：

```c
// 创建和绑定 FBO：
GLuint fbo;
glGenFramebuffers(1, &fbo); // 创建 FBO
glBindFramebuffer(GL_FRAMEBUFFER, fbo); // 绑定 FBO，注意：如果这里用 glBindFramebuffer(GL_FRAMEBUFFER, 0) 则是激活默认的帧缓冲区

// 创建 RBO：
GLuint rbo;
glGenRenderbuffers(1, &rbo); // 创建 RBO
glBindRenderbuffer(GL_RENDERBUFFER, rbo); // 绑定 RBO，所有后续渲染缓冲操作都会影响到当前的渲染缓冲对象
glRenderbufferStorage(GL_RENDERBUFFER, GL_RGBA, m_width, m_height); // 为 RBO 的颜色缓冲区分配存储空间

// 将 RBO 添加为 FBO 的附件，连接在颜色附着点：
glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_RENDERBUFFER, rbo);

// 检测 FBO：
GLenum status = glCheckFramebufferStatus(GL_FRAMEBUFFER);
if (status != GL_FRAMEBUFFER_COMPLETE)
    printf("Frame buffer incomplete!\n");
else
    printf("Frame buffer complete!\n");

// ...省略其他代码...
```



参考：

- [Learn OpenGL 帧缓冲](https://learnopengl-cn.readthedocs.io/zh/latest/04%20Advanced%20OpenGL/05%20Framebuffers/ "Learn OpenGL 帧缓冲")
- [FBO 离屏渲染](https://blog.csdn.net/Kennethdroid/article/details/98883854 "FBO 离屏渲染")
- [关于 FBO 的一些理解](https://www.cnblogs.com/chandler00x/p/3886062.html "关于 FBO 的一些理解")












