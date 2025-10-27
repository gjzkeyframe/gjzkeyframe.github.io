---
title: 宅男秒变性感美女？阿里 Wan2.2 让你 1 条视频收割百万流量
description: 一段你在说话的视频，一张相同分辨率的其他人物的图片，就能通过 AI 把视频中的你换成图片中的人物。
author: Keyframe
date: 2025-10-27 18:08:08 +0800
categories: [AI 产品]
tags: [AI 产品, 换脸, AI, AIGC, 视频, 照片, GIF, Wan]
pin: false
math: true
mermaid: true
image:
  path: assets/resource/aigc-product/replace-characters-5.jpeg
  alt: Replace Characters
---



阿里在大模型上的发力真的很猛，不停的在开源各种各样的新模型，他们前一段时间开源的 Wan2.2 有点东西！

![视频换人](assets/resource/aigc-product/replace-characters-2.mp4)
_Wan2.2 视频换人_

一段你在说话的视频，一张相同分辨率的其他人物的图片，就能通过 AI 把视频中的你换成图片中的人物，让你不露脸就能换人出镜，这真是 i 人主播和博主的福音！这套技术理论上直接开直播问题也不大，再把音色换一换，死肥宅摇身一变性感美女，也可以去收割百万流量了。


上面这个视频是一位博主发布在 Twitter/X 上的一条帖子，这条帖子目前已经有 100 万的浏览量了，可见大家对这个效果还是很认可的。

![视频换人](assets/resource/aigc-product/replace-characters-1.png)
_Wan2.2 视频换人_

那他是怎么实现的呢？对 AIGC 新技术感兴趣欢迎加群讨论：


>想要学习 AI 技术的朋友，快来加入我们的<a href="https://t.zsxq.com/nd3Wj" target="_blank" rel="noopener noreferrer">【AI 赚钱社群】</a>，加入后你就能：
>
>- 1）获得全套 Stable Diffusion WebUI AI 图片和视频制作教程
>- 2）获得全套 ComfyUI AI 图片和视频制作教程
>- 3）获得自动化批量生产 AI 套图的程序
>- 4）获得 <a href="https://aigchub.ai" target="_blank" rel="noopener noreferrer">AIGCHub</a> 创作者资格，用 AI 生产图片和视频来赚钱
>
>现在加入，送你一张 20 元优惠券：<a href="https://t.zsxq.com/nd3Wj" target="_blank" rel="noopener noreferrer">点击领取优惠券</a>
>
>![知识星球新人优惠券](assets/img/aigc-zsxq-coupon.png){: w="300" }
>_微信扫码也可领取优惠券_
{: .prompt-tip }





步骤如下：


- 1、准备一个你自己面对镜头讲话的视频 V，以及一张跟视频同样镜头构图的照片 A（可以直接从视频 V 中截图）
- 2、再准备一张你想要替换的人物照片 B，尺寸跟镜头角度和构图尽量跟 A 接近
- 3、使用 Nano Banana，用下面提示词将 A、B 两张图片上传，生成图片 C

```
Prompt：
把第一张图片的人物换成第二张的人物，同时保持第一张图片的背景环境不变。简单说就是用第二张的人物去替换第一张的人物。同时需要注意融合进去后的环境光线打在人物上不能太违和。需要跟环境协调。
```

![使用 Nano Banana 生成新图](assets/resource/aigc-product/replace-characters-3.jpeg)
_Nano Banana 生成新图_

![生成新图 C](assets/resource/aigc-product/replace-characters-4.jpeg)
_生成新图_


- 4、将生成的图片 C 跟视频 V 上传到 [https://create.wan.video](https://create.wan.video) ，设置如下参数来生成视频：

```
参数设置👇
Media：Avatar
Function：Photo Animate
```

![使用 Wan 生成新视频](assets/resource/aigc-product/replace-characters-5.jpeg)
_Wan 生成新视频_

---

如果你想要换脸生成新的图片、GIF、视频，你还可以使用 FaceXSwap：

![在 AppStore 搜索 'facexswap'](resource/aigc-influence/aigc-influence-70/facexswap-2.png)
_在 AppStore 搜索 'facexswap'_


- FaceXSwap 官网：[https://facexswap.com](https://facexswap.com)
- FaceXSwap iOS App 下载：[https://apps.apple.com/app/id6752116909](https://apps.apple.com/app/id6752116909)
