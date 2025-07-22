---
title: 音视频面试题集锦第 27 期
description: 持续更新的音视频面试题集锦。
author: Keyframe
date: 2025-02-17 04:58:08 +0800
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

下面是第 27 期面试题精选，我们来看看在跨平台音视频 SDK 开发常用到的 C++ 语言的相关基础知识：


- **1、delete this 合法吗？**
- **2、extern "C" 作用？**
- **3、C++ 中有哪些引用？**
- **4、C++ 内存泄漏怎么产生的？如何避免？**
- **5、如何写出高效的 SDK？**

## 1、delete this 合法吗？


合法，但是需要满足以下条件：

- 必须保证 this 对象是通过 `new`（不是 `new[]`、不是 `placement new`、不是栈上、不是全局、不是其他对象成员）分配的；
- 必须保证调用 `delete this` 的成员函数是最后一个调用 `this` 的成员函数；
- 必须保证成员函数的 `delete this ` 后面没有调用 `this` 了；
- 必须保证 `delete this` 后没有人使用了。


参考：[Is it legal (and moral) for a member function to say delete this?](https://isocpp.org/wiki/faq/freestore-mgmt#delete-this "delete this")



## 2、extern "C" 作用？

被 `extern "C"` 修饰的变量和函数是按照 C 语言方式编译和链接的。

`extern "C"` 的作用是让 C++ 编译器将 `extern "C"` 声明的代码当作 C 语言代码处理，可以避免 C++ 因符号修饰导致代码不能和C语言库中的符号进行链接的问题。


使用示例：

```c
#ifdef __cplusplus
extern "C" {
#endif

void *memset(void *, int, size_t);

#ifdef __cplusplus
}
#endif
```


## 3、C++ 中有哪些引用？

**1）左值引用**

常规引用，一般表示对象的身份。

**2）右值引用**

右值引用就是必须绑定到右值（一个临时对象、将要销毁的对象）的引用，一般表示对象的值。

右值引用可实现转移语义（Move Sementics）和精确传递（Perfect Forwarding），它的主要目的有两个方面：

- 消除两个对象交互时不必要的对象拷贝，节省运算存储资源，提高效率。
- 能够更简洁明确地定义泛型函数。

**3）引用折叠**

- `X& &`、`X& &&`、`X&& &` 可折叠成 `X&`
- `X&& &&` 可折叠成 `X&&`


## 4、C++ 内存泄漏怎么产生的？如何避免？

C++ 内存泄露是如何产生的：

- 内存泄漏一般是指**堆内存**的泄漏，也就是程序在运行过程中动态申请的内存空间不再使用后没有及时释放，导致那块内存不能被再次使用。
- 更广义的内存泄漏还包括未对**系统资源**的及时释放，比如句柄、socket 等没有使用相应的函数释放掉，导致系统资源的浪费。

如何避免：

- 养成良好的编码习惯和规范，记得及时释放掉内存或系统资源；
- 重载 new 和 delete，以链表的形式自动管理分配的内存；
- 使用智能指针：`share_ptr`、`auto_ptr`、`weak_ptr`。


## 5、如何写出高效的 SDK？

高效意味着：稳定、简单易懂、易扩展、有良好的反馈机制。

- 基础：
    - 代码风格、命名规范
    - 日志、基础工具、上报等等
- 稳定：
    - 写模块时，要先想清楚在动手
    - 先把模块的设计思路、生命周期、调用流程等在文档中梳理清楚
    - 单元测试、压力测试、多线程测试等，如：TDD 的方式进行开发
    - 最终输出模块时，附加上占用内存(静态、动态内存范围)、执行耗时、CPU/GPU 消耗等
    - 异常处理：异常日志、异常回调等
- 简单易懂
    - SDK 的背景、框架、时序、调用流程等，用图的方式呈现出来，并输出文档
    - 良好的命名规范、统一风格
    - 统一调用，屏蔽一些细节，减少使用方的调用成本，集中配置
    - 结构清晰
    - 编写注释、说明文档
- 易扩展
    - 减少依赖（减少库冲突）
    - 外部可以自定义实现，比如：定义协议接口，依赖注入的方式
- 反馈机制
    - SDK 中的日常、性能、异常等告警和监控，不断的通过监控，发现问题与优化




---

**更多的音视频知识、面试题、技术方案干货可以进群来看：**

![限时优惠，扫码加入](assets/img/keyframe-zsxq.png)





---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
{: .prompt-tip }

