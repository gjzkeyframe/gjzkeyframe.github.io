---
title: 音视频面试题集锦第 3 期
description: 持续更新的音视频面试题集锦。
author: Keyframe
date: 2025-02-17 01:38:08 +0800
categories: [音视频面试题集锦]
tags: [音视频面试题集锦,  面试, 音视频]
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


我们在知识星球上创建的音视频技术社群**关键帧的音视频开发圈**已经运营了一段时间了，在这里群友们会一起做一些打卡任务。比如：周期性地整理音视频相关的面试题，汇集一份**音视频面试题集锦**，你可以看看这个合集：[音视频面试题集锦](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2380776196751425539#wechat_redirect)。再比如：循序渐进地归纳总结音视频技术知识，绘制一幅**音视频知识图谱**，你可以看看这个合集：[音视频知识图谱](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2349658423078092802#wechat_redirect)。

下面是 2022.09 月音视频面试题集锦内容的节选：

**1）OpenGL 是按照什么架构设计的？**

>OpenGL 的渲染架构是 **Client/Server 模式**：Client（客户端）指的是我们在 CPU 上运行的一些代码，比如我们会编写 OC/C++/Java 代码调用 OpenGL 的一些 API；而 Server（服务端）则对应的是图形渲染管线，会调用 GPU 芯片。我们开发的过程就是不断用 Client 通过 OpenGL 提供的通道去向 Server 端传输渲染指令，来间接的操作 GPU 芯片。
>
>![OpenGL 渲染架构](assets/resource/av-interview-qa/opengl-render-structure.webp)


**2）什么是渲染上下文（Context）？**

>OpenGL 自身是一个巨大的状态机（State Machine）：一系列的变量描述 OpenGL 此刻应当如何运行。OpenGL 的状态通常被称为 OpenGL 上下文（Context）。我们通过改变上下文中的状态来改变接下来绘画的属性和操作的缓冲对象，然后 OpenGL 利用当前的上下文（Context）的状态去渲染。因此状态的改变要非常小心，因为是状态是全局，会影响接下来的所有渲染操作。

**3）什么是离屏渲染？**

>GPU 渲染机制：CPU 计算好显示内容提交到 GPU，GPU 渲染完成后将渲染结果放入帧缓冲区，随后屏幕控制器会按照 VSync 信号逐行读取帧缓冲区的数据，经过可能的数模转换传递给显示器显示。
>
>当前屏幕渲染，指的是 GPU 的渲染操作是在当前用于显示的屏幕缓冲区中进行。
>
>离屏渲染，指的是 GPU 在当前屏幕缓冲区以外新开辟一个缓冲区进行渲染操作。
>
>特殊的离屏渲染：如果将不在 GPU 的当前屏幕缓冲区中进行的渲染都称为离屏渲染，那么就还有另一种特殊的离屏渲染方式：CPU 渲染。


**4）为什么离屏渲染会造成性能损耗？**

>当使用离屏渲染的时候会很容易造成性能消耗，因为离屏渲染会单独在内存中创建一个屏幕外缓冲区并进行渲染，而屏幕外缓冲区跟当前屏幕缓冲区上下文切换是很耗性能的。
>
>由于垂直同步的机制，如果在一个 VSync 时间内，CPU 或者 GPU 没有完成内容提交，则那一帧就会被丢弃，等待下一次机会再显示，而这时显示屏会保留之前的内容不变。这就是界面卡顿的原因。


**5）什么是 OpenGL 渲染管线（Pipeline）？**

>OpenGL 渲染管线就是 OpenGL 的工作流程，指的是一堆原始图形数据途经一个输送管道，期间经过各种变化处理最终出现在屏幕的过程。   
>
>图形渲染管线可以大致被划分为两个主要部分：第一部分把你的 3D 坐标转换为 2D 坐标；第二部分是把 2D 坐标转变为实际的有颜色的像素。 


**6）OpenGL 渲染管线主要包含哪些部分？**

>OpenGL 的渲染管线其实也是类似的一个过程，它的工序包括：顶点着色器 → 图元装配 → 几何着色器 → 光栅化 → 片段着色器 → 测试与混合。
>
>![OpenGL 渲染管线](assets/resource/av-interview-qa/opengl-render-pipeline.webp)


**7）为什么说 OpenGL 渲染管线中的着色器（Shader）是可编程管线？**

>OpenGL 渲染管线中着色器允许开发者自己配置，这样我们就可以使用 GLSL（OpenGL Shading Language）来编写自己的着色器替换默认的着色器，从而更细致地控制图形渲染管线中的特定部分。

**8）有哪些着色器可以由程序员进行编程？**

>可编程的着色器有：顶点着色器（Vertex Shader）、几何着色器（Geometry Shader）、片段着色器（Fragment Shader）。常用的是顶点着色器和片段着色器。

**9）什么是 VBO、EBO 和 VAO？**

>可以认为它们是在 OpenGL 中处理数据的三大类缓冲内存对象。  
>
>VBO（Vertex Buffer Objects）顶点缓冲区对象，指的是在 GPU 显存里面存储的顶点数据（位置、颜色）。
>
>EBO（Element Buffer Object）图元索引缓冲区对象，指的是为了更高效的利用数据，存储索引来达到减少重复数据的索引数据。
>  
>VAO（Vertex Array Object）顶点数组对象，主要作用是用于管理 VBO 或 EBO，减少 `glBindBuffer`、`glEnableVertexAttribArray`、`glVertexAttribPointer` 这些调用操作，高效地实现在顶点数组配置之间切换。


**10）Vertex Buffer Object 的布局格式是怎样的？**

>当我们的 Vertex Shader 如下：
>
>```sh
>#version 330 core
>layout (location = 0) in vec3 position; // 位置变量的属性位置值为 0 
>layout (location = 1) in vec3 color;    // 颜色变量的属性位置值为 1
>
>out vec3 ourColor; // 向片段着色器输出一个颜色
>
>void main()
>{
>    gl_Position = vec4(position, 1.0);
>    ourColor = color; // 将ourColor设置为我们从顶点数据那里得到的输入颜色
>}
>```
>
>Fragment Shader 如下：
>
>```sh
>#version 330 core
>in vec3 ourColor;
>out vec4 color;
>
>void main()
>{
>    color = vec4(ourColor, 1.0f);
>}
>
>```
>
>创建 VBO 的代码如下：
>
>```c
>GLfloat vertices[] = {
>    // 位置              // 颜色
>     0.5f, -0.5f, 0.0f,  1.0f, 0.0f, 0.0f,   // 右下
>    -0.5f, -0.5f, 0.0f,  0.0f, 1.0f, 0.0f,   // 左下
>     0.0f,  0.5f, 0.0f,  0.0f, 0.0f, 1.0f    // 顶部
>};
>
>GLuint VBO;
>glGenBuffers(1, &VBO);  
>glBindBuffer(GL_ARRAY_BUFFER, VBO);  
>glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
>```
>
>这时候对应的 VBO 布局格式如下图所示：
>
>![VBO 布局格式](assets/resource/av-interview-qa/vbo.png)


**11）Vertex Array Object 的布局格式是怎样的？**

>当 VAO 只管理 VBO 时，布局格式如下图所示：
>
>![VAO 管理 VBO 布局格式](assets/resource/av-interview-qa/vao-vbo.png)
>
>当 VAO 管理 VBO 和 EBO 时，布局格式如下图所示：
>
>![VAO 管理 VBO 和 EBO 布局格式](assets/resource/av-interview-qa/vao-vbo-ebo.png)






