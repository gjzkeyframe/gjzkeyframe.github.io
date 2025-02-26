---
title: 音视频面试题集锦第 18 期
description: 持续更新的音视频面试题集锦。
author: Keyframe
date: 2025-02-17 03:58:38 +0800
categories: [音视频面试题集锦]
tags: [音视频面试题集锦, 音视频]
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



我们在知识星球上创建的音视频技术社群**关键帧的音视频开发圈**已经运营了一段时间了，在这里大家可以一起交流和分享音视频技术知识和实战方案。我们会不定期整理一些音视频相关的面试题，汇集一份[音视频面试题集锦（可进入免费订阅）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2380776196751425539#wechat_redirect)。也会循序渐进地归纳总结音视频技术知识，绘制一幅[音视频知识图谱（可进入免费订阅）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2349658423078092802#wechat_redirect)。



下面是第 18 期面试题精选：


- **1、聊聊 OpenGL glFlush 和 glFinish 区别？**
- **2、怎么实现 OpenGL 多线程同步？**
- **3、如何实现 OpenGL 资源共享？**
- **4、OpenGL 纹理缓存要如何设计？**



## 1、聊聊 OpenGL glFlush 和 glFinish 区别？



一般来说，我们在使用 OpenGL 的时候，指令不是立即执行的。它们首先被送到指令缓冲区，然后才被送到硬件执行。`glFinish` 和 `glFlush` 都是强制将命令缓冲区的内容提交给硬件执行。

**1）glFlush：**

`glFlush` 清空缓冲区，将指令送往缓硬件立即执行，但是它是将命令传送完毕之后立即返回，不会等待指令执行完毕。这些指令会在有限时间内执行完毕。

**2）glFinish：**

`glFinish` 将缓冲区的指令立即送往硬件执行，但是要一直等到硬件执行完这些指令之后才返回。如果直接绘制到前缓冲，那么在你想保存屏幕截图之前，就需要调用这个函数，确保绘制完毕。如果使用双缓冲，则这个函数不会有太大作用。

如果调用 `glFinish`，通常会带来性能上的损失。因为它会是的 GPU 和 CPU 之间的并行性丧失。一般，我们提交给驱动的任务被分组，然后被送到硬件上（在缓冲区交换的时候）。如果调用 `glFinish`，就强制驱动将命令送到 GPU。然后 CPU 等待直到被传送的命令全部执行完毕。这样在 GPU 工作的整个期间内，CPU 没有工作（至少在这个线程上）。而在 CPU 工作时（通常是在对命令分组），GPU 没有工作。因此造成性能上的下降。

**3）glFlush 和 glFinish 区别：**

一般使用 `glFlush` 的目的是确保在调用之后，CPU 没有 OpenGL 相关的事情需要做，命令会送到硬件执行。调用 `glFinish` 的目的是确保当返回之后，没有相关工作留下需要继续做。






## 2、怎么实现 OpenGL 多线程同步？


**1）OpenGL 为什么需要同步？**

一般情况下我们调用 OpenGL 方法后，并不是马上有效果的，如果在 B 线程使用 A 线程的纹理有概率出现渲染异常，因为 A 纹理还没有渲染完成。

**2）glFinish 同步方案**

如果要某处确保之前的 OpenGL 执行完，通常会用到 `glFinish`，确保效果正常。但 `glFinish` 只能保证本线程对应的命令队列中的命令执行完，这就意味着不能在一个线程中等待另一个线程的 OpenGL 命令执行完，这就有很大的限制。

**3）Fence 同步方案**

回想我们在 CPU 上的同步操作，例如我们在一个线程中 wait，在另一个线程中 notify，这很容易实现在一个线程中等待另一个线程的指定任务执行完成，这也是我们很常用的操作，但在对于 GPU，无法用 `glFinish` 来实现类似的逻辑。到了OpenGL ES 3.0，我们可以用 `fence` 实现，使用越来也很简单，就是在一个线程中插入一个 `fence`，然后在另一个线程中就可以去等待这个 `fence` 。

![](assets/resource/av-interview-qa/fence-sync.webp)


例如我们有这样一种逻辑，在 GLThread 0 中渲染一个纹理，在另一个线程 GLThread 1 中将这个纹理拿去使用，那就需要确保在 GLThread 1 使用这个纹理时，GLThread 0 对这个纹理的渲染已经完成，有了 `fence` 后，我们可以在 GLThread 0 渲染操作之后插入一个 `fence`，然后在 GLThread 1 要使用这个纹理时去等这个 `fence`。

插入 `fence` 的代码，通常线程 A 插入：

```c
GLsync fenceSyncObject = glFenceSync(GL_SYNC_GPU_COMMANDS_COMPLETE, 0);
glFlush();
```

这个方法调用后会往当前线程的命令队列中插入一个 `fence` 并返回一个 long 型变量来代码这个 `fence` 同步对象，以便于其它地方去等待它。glFlush 这里作用保证  `fence`  指令一定可以推入 GPU。

等待 `fence` 的代码，通常线程 B 等待：

```c
glClientWaitSync(fence, 0, GL_TIMEOUT_IGNORED);
glDeleteSync(fence);
```

有 2 个方法可以用于等待， `glWaitSync` 和 `glClientWaitSync`，它们的差别是 `glWaitSync` 是在 GPU 上等待，`glClientWaitSync` 是在 CPU 上等待。这样好处不会阻塞 CPU ，提高渲染效率。





## 3、如何实现 OpenGL 资源共享？

**1）资源共享基础描述**

OpenGL 渲染中有一个线程相关的上下文 (Context), OpenGL 所创建的资源，其实对程序员可见的仅仅是上下文 ID 而已，其内容依赖于这个上下文，有时候为了方便起见，在某个线程中创建了上下文之后，所有的 OpenGL 操作都转到此线程来调用。这样在简单的 2d/3d 渲染中尚可，但是如果涉及复杂的 OpenGL 渲染时，这样就未必足够， 事实上 OpenGL 已经考虑到这一点， 上下文是可以在多个线程间共享的，在使用 eglCreateContext 时， 可以传入一个已创建成功的上下文， 这样就可以得到一个共享的上下文 (Shared Context).

OpenGL 的绘制命令都是作用在当前的 Context 上，这个 Current Context 是一个线程私有（thread-local）的变量，也就是说如果我们在线程中绘制，那么需要为该线程制定一个 Current Context 的，当多个线程参与绘制任务时，需要原线程解绑再重新绑定新的线程。多个线程不能同时指定同一个 Context 为 Current Context，否则会导致崩溃。

![](assets/resource/av-interview-qa/opengl-context.webp)

**2）OpenGL 可以共享哪些资源？**

可以共享的资源：

- 纹理；
- shader；
- program 着色器程序；
- buffer 类对象，如 VBO、 EBO、 RBO 等 。

不可以共享的资源：

- FBO 帧缓冲区对象（不属于 buffer 类）；
- VAO 顶点数组对象（不属于 buffer 类）。

在不可以共享的资源中，FBO 和 VAO 属于资源管理型对象，FBO 负责管理几种缓冲区，本身不占用资源，VAO 负责管理 VBO 或 EBO ，本身也不占用资源。

**3）OpenGL 共享上下文实现？**

iOS ShareContext 实现：

```c
- (EAGLContext *)shareContext:(nullable EAGLContext *)context {
    EAGLContext *shareContext = [[EAGLContext alloc] initWithAPI:context.API sharegroup:context.sharegroup]; 
    return shareContext;
}
```

Android ShareContext 实现：

```c
public EGLContext eglCreateContext(EGLDisplay display, EGLConfig config, EGLContext share_context, int[] attrib_list) {
      EGLContext result = mEgl10.eglCreateContext(display, config,
                share_context, attrib_list);
        return result;
}
```

**4）OpenGL 共享上下文有哪些注意事项？**

- 每个线程同时只能绑定（eglMakeCurrent）一个 Context ，但可以按顺序切换不同 Context。
- 多线程共享 Context 需要注意同步，可以使用 fence 或者 glFinish。
- FBO、VAO 不同 Context 不可以共享。





## 4、OpenGL 纹理缓存要如何设计？


**1）OpenGL 纹理缓存用途？**

- 播放器场景：解码器解码后的纹理上屏，通用情况解码后的纹理立即渲染即可，但如果解码后的纹理添加缓存模块，缓存模块可以大大优化播放器的渲染帧率（4K 模式）。
- 转码场景：编码与解码通常为 2 个不同线程，解码需要有自己的纹理缓存，这样异步编码模块可以最快速度获取解码纹理数据。

**2）OpenGL 纹理缓存如何设计？**

- 需要一个可复用的纹理数组，设置一个最大上限。
- 每个纹理需要忙碌或空闲的状态，当空闲情况下可以进行复用。
- 一个 FBO 频繁更换绑定不同的纹理，将内容数据刷新到指定纹理上。
- 外层纹理使用完成后将纹理状态设置为空闲。

**3）FBO 绑定指定纹理如何实现？**

代码示例如下：

```c
void  AttachTextureToFBO (GLuint texId) {
  glBindFramebuffer(GL_FRAMEBUFFER, fbo);
  glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, texId, 0);
  GLenum status = glCheckFramebufferStatus(GL_FRAMEBUFFER);
  check(status);
  glBindFramebuffer(GL_FRAMEBUFFER, 0);
}
```


---

**更多的音视频知识、面试题、技术方案干货可以进群来看：**

![微信扫码加入](assets/img/keyframe-zsxq.png)
_微信扫码加入_























