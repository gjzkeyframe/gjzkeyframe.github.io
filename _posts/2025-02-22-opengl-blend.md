---
title: OpenGL 颜色混合浅析
description: 介绍 OpenGL 颜色混合的基础概念和知识。
author: Keyframe
date: 2025-02-22 08:40:08 +0800
categories: [音视频基础知识]
tags: [音视频基础知识, 音视频, 渲染, OpenGL]
pin: false
math: true
mermaid: true
---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频**、**AI** 领域的最新技术和产品信息：
>
>![微信公众号](assets/img/keyframe-mp.jpg)
_微信扫码关注我们_
{: .prompt-tip }


混合是什么呢？混合就是把两种颜色混在一起。具体一点，就是把某一像素位置当前存储在颜色缓冲区的颜色和将要画上去的颜色，通过某种方式混在一起，从而实现特殊的效果。

OpenGL 一次渲染过程包含了多个阶段，包括顶点着色器、图元组装、栅格化、片元着色器、测试和混合等，最后将结果输出到 `FrameBuffer` 上。渲染管线最后一个阶段就是混合。

![OpenGL 渲染管线](assets/resource/av-basic-knowledge/blend.jpeg)
_OpenGL 渲染管线_

要正确的渲染出预期的颜色效果，需要对混合的几个概念有一些了解，否则很可能会发现最后出来的颜色跟自己想要的是不一样的。这篇文章我们就讲一讲相关的概念和实践。


## 1、源色与目标色

前面我们已经提到，混合需要把存储在颜色缓冲区当前位置的颜色和将要画上去的颜色找出来，经过混合处理后得到一种新的颜色。这里把将要画上去的颜色称为`源颜色（Source Color）`，把颜色缓冲区中当前的颜色称为`目标颜色（Destination Color）`。

针对 `OpenGL` 渲染场景：

- 源颜色：Shader 中 `gl_FragColor` 的颜色。
- 目标颜色：`glClearColor` 的颜色。

## 2、关闭颜色混合

当调用了 `glDisable(GL_BLEND)` 就表示关闭颜色混合。默认情况下 OpenGL 的颜色混合就是关闭的，这时候需要注意：颜色透明通道这个参数，即颜色的 alpha 值，是不起作用的。

```c
// 缓冲区颜色
glClearColor(1, 0, 0, 1); // rgba，红色
glClear(GL_COLOR_BUFFER_BIT);

// Shader
void main()
{
	gl_FragColor = vec4(0.0, 1.0, 0.0, 0.0); // rgba，绿色
}
```

比如，上面 `glClearColor` 设置了窗口颜色为红色，而 Shader 中设置 `gl_FragColor` 为绿色，虽然透明通道 apha 值为 `0`，但因为没有开启颜色混合，透明通道值不会影响颜色的渲染，所以最终显示的颜色为绿色。





## 3、开启颜色混合

当调用了 `glEnable(GL_BLEND)` 就表示开启颜色混合。这时候就会对源颜色 `gl_FragColor` 与目标色 `glClearColor ` 进行混合。

```c
glEnable(GL_BLEND); // 开启颜色混合
glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA); // 设置颜色混合模式
glDisable(GL_BLEND); // 关闭颜色混合
```

### 3.1、颜色混合模式

在开启了颜色混合后，还需要通过 `glBlendFunc` 函数指定混合模式。`glBlendFunc` 函数的参数可以理解为混合因子，第一个参数代表源颜色混合因子，第二个参数代表目标颜色混合因子。混合因子可选值如下：

![混合因子可选值](assets/resource/av-basic-knowledge/blend-mode.png)
_混合因子可选值_

根据上面的混合因子的可选值，可以组合出来的混合模式有很多种。不过，在我们开发中常用的混合模式有 2 种，我们举例分析：

```c
// 缓冲区颜色
glClearColor(1.0, 0.0, 0.0, 1.0); // rgba，红色
glClear(GL_COLOR_BUFFER_BIT);
glEnable(GL_BLEND);

// Shader
void main()
{
	gl_FragColor = vec4(0.0, 1.0, 0.0, 0.0); // rgba，绿色
}
```

对于不开启颜色混合最终输出是绿色，那么开启颜色混合后最终渲染出来是什么颜色呢？


1）如果这样设置颜色混合模式：`glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA)`

- `GL_SRC_ALPHA` 表示源颜色混合因子为源颜色的 alpha 值，`GL_ONE_MINUS_SRC_ALPHA` 表示目标颜色混合因子为 1 减去源颜色的 alpha 值。
- 计算方法：源色为 `(0.0, 1.0, 0.0, 0.0)`，目标色为 `(1.0, 0.0, 0.0, 1.0)`，源颜色 alpha 值为 `0.0`，那么目标颜色混合因子为 `1 - 0.0 = 1.0`，通过`源颜色 * 源颜色混合因子 + 目标色 * 目标色混合因子`，最终得到混合后颜色：`(0.0*0.0+1.0*1.0, 1.0*0.0+0.0*1.0, 0.0*0.0+0.0*1.0)`为红色。


2）如果这样设置颜色混合模式：`glBlendFunc(GL_ONE, GL_ONE_MINUS_SRC_ALPHA)`

- `GL_ONE` 表示源颜色混合因子为 1，`GL_ONE_MINUS_SRC_ALPHA` 表示目标颜色混合因子为 1 减去源颜色的 alpha 值。
- 计算方法：源色为 `(0.0, 1.0, 0.0, 0.0)`，目标色为 `(1.0, 0.0, 0.0, 1.0)`，源颜色混合因子 `1`，alpha 值为 `0.0`，所以目标透明因子为 `1 - 0.0 = 1.0`，通过`源颜色 * 源颜色混合因子 + 目标色 * 目标色混合因子`，最终混合后颜色 `(0.0*1+1.0*1.0, 1.0*1+0.0*1.0, 0.0*1+0.0*1.0)` 为黄色。

### 3.2、颜色预乘

另外在渲染图像时还需要注意一下`颜色预乘`的概念，如果处理不当，最终渲染出来的颜色也可能出现不对的情况。

什么是颜色预乘呢，针对于解码后的图片颜色为 `(r, g, b, a)`，颜色预乘则是 rgb 乘以 alpha 值，预乘后的结果为 `(r*a, g*a, b*a, a)`，现在 iOS 和 Android 平台系统解码后的图片均是预乘后的结果。

- 对于预乘后的图片，我们做混合情况下需要使用 `glBlendFunc(GL_ONE, GL_ONE_MINUS_SRC_ALPHA)`，因为源颜色已经乘过了。
- 对于没有预乘的图片，我们做混合情况下需要使用 `glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA)`，这种方式会进行源颜色乘法。




