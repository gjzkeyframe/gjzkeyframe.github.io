---
title: 音视频面试题集锦第 30 期
description: 持续更新的音视频面试题集锦。
author: Keyframe
date: 2025-02-17 05:08:18 +0800
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


我们在知识星球上创建的音视频技术社群**关键帧的音视频开发圈**已经运营了一段时间了，在这里大家可以一起交流和分享音视频技术知识和实战方案。我们会不定期整理一些音视频相关的面试题，汇集一份[音视频面试题集锦（可进入免费订阅）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2380776196751425539#wechat_redirect)。也会循序渐进地归纳总结音视频技术知识，绘制一幅[音视频知识图谱（可进入免费订阅）](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2349658423078092802#wechat_redirect)。


下面是第 30 期面试题精选：


- **1、为什么自制的动态图片导出到相册无法识别成动态图片？**
- **2、iOS 如何实现音频内录，录制当前所有手机的声音集合？**
- **3、应该选择哪种复杂字体渲染的方案 (非 UIView 的能力)？**
- **4、iOS 如何在不解码的情况下给视频添加 Metadata？**



## 1、为什么自制的动态图片导出到相册无法识别成动态图片？

Live Photo 需要有一个特殊的 Metadata Key `[kCGImagePropertyMakerAppleDictionary : [17 : <Identifier>]]`。

因此需要先将此 Metadata 导入到图片中，然后再导出动态图片，这样图片才会被识别成动态图片。

以下是将此 Metadata 导入到图片的 swift 代码示例：

```java
guard let imageSource = CGImageSourceCreateWithURL(photoURL as CFURL, nil),
let imageRef = CGImageSourceCreateImageAtIndex(imageSource, 0, nil),
var imageProperties = CGImageSourceCopyPropertiesAtIndex(imageSource, 0, nil) as? [AnyHashable : Any] else {
    throw LivePhotosAssembleError.addPhotoIdentifierFailed
}
let identifierInfo = ["17" : identifier]
imageProperties[kCGImagePropertyMakerAppleDictionary] = identifierInfo
guard let imageDestination = CGImageDestinationCreateWithURL(destinationURL as CFURL, UTType.jpeg.identifier as CFString, 1, nil) else {
    throw LivePhotosAssembleError.createDestinationImageFailed
}
CGImageDestinationAddImage(imageDestination, imageRef, imageProperties as CFDictionary)
if CGImageDestinationFinalize(imageDestination) {
    return destinationURL
} else {
    throw LivePhotosAssembleError.createDestinationImageFailed
}
```



## 2、iOS 如何实现音频内录，录制当前所有手机的声音集合？

在 iOS 11 中，ReplayKit 提供了强大的能力：将系统作为一个整体进行直播。用户在控制中心内开启屏幕录制后，ReplayKit2 Extension 可以获取到整个系统级的屏幕画面、以及设备所产生的所有音频，实现跨应用录屏(iOS System Boardcast)，同时 ReplayKit 也提供了麦克风采集，这种系统级的直播在应用间切换时也不会停止。可以看下图了解 ReplayKit 的流程。： 

![ReplayKit 数据流](assets/resource/av-interview-qa/ReplayKit.webp)   

用户通过手动点击录制按钮然后选择你的应用，则你的 Extension 即可接收到音视频数据，此时你可以在 Extension 里面做一些简单处理或者上传。

更多的内容可以参考：[iOS ReplayKit 与 屏幕录制](https://segmentfault.com/a/1190000043619243 "iOS ReplayKit 与 屏幕录制")


## 3、应该选择哪种复杂字体渲染的方案 (非 UIView 的能力)？

看过多个字体渲染的三方库发现有部分平台上还是使用了系统的能力。因为系统的字体渲染更稳定，性能更高，支持的功能也足够丰富。例如 iOS 使用了 CoreText，高性能，支持高级文本布局和多种字体，并且可以轻松的输出图片。

以下是 CoreText 实现文字渲染然后输出到图片的示例：

```objc
CTFontRef font = CTFontCreateWithName(CFSTR("Helvetica-Bold"), 16, NULL);
NSDictionary *attributes = @{(id)kCTFontAttributeName: (__bridge id)font};
NSMutableAttributedString *attributedString = [[NSMutableAttributedString alloc] initWithString:@"Hello, World!" attributes:attributes];

CTFramesetterRef framesetter = CTFramesetterCreateWithAttributedString((__bridge CFAttributedStringRef)attributedString);
CGMutablePathRef path = CGPathCreateMutable();
CGPathAddRect(path, NULL, CGRectMake(0, 0, 280, 50)); // 定义绘制区域
CTFrameRef frame = CTFramesetterCreateFrame(framesetter, CFRangeMake(0, 0), path, NULL);

CGContextRef context = UIGraphicsGetCurrentContext(); // 获取当前图形上下文
int width = 280;
int height = 50;
GLubyte *imageData = (GLubyte *) calloc(1, (int)width * (int)height * 4);
if (!context){
    context = CGBitmapContextCreate(imageData, (size_t)width, (size_t)height, 8, (size_t)width * 4, CGColorSpaceCreateDeviceRGB(),  kCGBitmapByteOrder32Little | kCGImageAlphaPremultipliedFirst);
}
CGContextSetGrayFillColor(context, 1.0f, 1.0f);
CGContextSetTextMatrix(context, CGAffineTransformIdentity); // 设置文本矩阵
UIGraphicsPushContext(context);
[attributedString drawAtPoint:CGPointMake(0 , 0)];
CGImageRef a = CGBitmapContextCreateImage(context);
UIImage *newImage = [UIImage imageWithCGImage:a]; // 输出结果

CGContextRelease(context);
CFRelease(frame);
CGPathRelease(path);
CFRelease(fontName);
CFRelease(font);
```


## 4、iOS 如何在不解码的情况下给视频添加 Metadata？


- 1、初始化 AVAssetReader，创建对应的 AVAssetReaderTrackOutput，包括 videoReaderOutput、audioReaderOutput。注意 videoReaderOutput 吐出的数据应该是 h264/h265 非解码的数据，audioReaderOutput 也应该是 aac 等非解码格式。
- 2、初始化 AVAssetWriter，创建及对应的 AVAssetWriterInput，包括 videoWriterInput、audioWriterInput。
- 3、初始化要添加的 AVMetaDataItem。
- 4、AVAssetWriter 添加来自 AVAssetWriterInputMetadataAdaptor 的 assetWriterInput。
- 5、AVAssetWriter 进入写状态。
- 5、使用 AVAssetReader 和 AVAssetWriterInputMetadataAdaptor 写入准备好的 Metadata。
- 6、videoReaderOutput、audioReaderOutput、videoWriterInput、audioWriterInput 进入读写写状态。
- 8、读取完所有数据后，让 AVAssetReader 停止读取。使所有 AVAssetWriterInput 标记完成。
- 9、等待 AVAssetWriter 变为完成状态，视频创建完成。





---

**更多的音视频知识、面试题、技术方案干货可以进群来看：**

![限时优惠，扫码加入](assets/img/keyframe-zsxq.png)





<!-- 




新人报道~顺便向向星球的大佬请教一下，想了解一下目前应该如何增添一个项目，然后也想问一点简历优化的建议，目前有以下主要问题bg是非科班单211硕

（1）项目上：主要是想新添一个项目把webserver 踢掉，找实习的时候做了一个RTSP服务器，整体框架其实都是那一套epoll+socket+线程池，只是多了一点音视频比如rtsp的知识。最近找了很久很久，实在是找不到合适的项目了，之前想做一个视频会议项目，但是好像很难找到，之前也花钱买了某某教育机构的音视频网课，但感觉项目实在是很难评...目前最后就是两个计划，一个是做一个简单kv存储项目主要目的就是换掉webserver，或者说做一个推流器相关的项目，但是我找了很久很久，都没有找到一个感觉可以写到简历的程度的项目...目前找了一个比较水的推流器项目。所以有点纠结，最大的问题在于希望赶上九月的秋招，希望在九月份之前把这个项目找到并完成写到简历上。

（2）简历上：目前可能第一个视频播放器还算是比较熟悉，两个服务器整体框架也还能说的出来，就是细节可能需要再复习一下，主要是实习感觉都有点水，其实虽然多段实习了，但是感觉去实习并没有让我真的去学到什么技术...感觉还是打杂居多，怕面试被问到直接被戳穿了...所以不知道是不是可以优化一下...

（3）复习方面：很纠结不知道优先级怎么安排，因为之前也面过音视频的实习，感觉主要还是C++基础居多，不知道音视频对校招是不是会考察更多业务上的知识，但是音视频业务知识也太广了，后面不知道是不是简历上那些业务知识能不能应付过来？所以不太清楚到底应该平时优先时间花在什么上面...

我目标也只是想在杭州（或江浙沪）找到C++开发，并没有给自己很高预期，希望前辈可以给我一点建议，非常非常感谢。



1）项目和简历

现在这个时间点，如果要继续找音视频方向 C++ 的工作，我觉得可以把播放器和 RTSP 这两个项目再打磨打磨吧。

在音视频 SDK 相关的实际工作中，大家通常会关注：1）解决需求的具体技术方案；2）技术指标的定义、量化、优化。因此，对于项目具体的改进方向，我建议你可以好好想想这几个问题：

- 你的项目是要解决什么需求？
- 你选用什么技术方案、通过怎么的技术架构来实现项目？
- 在项目中你遇到的难点是什么？如何解决？
- 你如何量化定义你在项目中的指标？如何优化指标？取得了什么样的效果？

如果你简历里的项目能把这几点表达清楚，在面试时能讲得好，我觉得就还好。


2）复习

应届毕业生找工作时，面试考察主要包括：1）大学课程相关的计算机基础知识（包括：数据结构与算法、计算机网络、编程语言特性及语法、设计模式、操作系统基础原理、数据库基础等）；2）算法编程题；3）项目开发经验。

所以在基础知识上，可以多注意数据结构与算法、编程语言特性及语法；此外需要多刷刷 leetcode 算法题；后面加分项就是上面说的针对你的项目开发经验做好准备。



3）你还可以看看几个群友求职相关的问题参考参考：

https://t.zsxq.com/lLDl3

https://t.zsxq.com/lKQeI

https://t.zsxq.com/OhyEF

https://t.zsxq.com/0y9sv




---



新人报道 提问🙋：
- 个人情况介绍：

 - 双非本 中等211硕 开学研2，非科班研究方向cv，目前无论文

 - 之前有学过一些音视频基础OpenGL基础ffmpeg等，CPP水平一般，有些遗忘，leetcode 正在刷题中，自己写过Qt和Android平台的视频播放器「快进快退seek重播音量调节等基础功能， rtmp推流正在完善中」

- 问题：

 - 小目标想在寒假暑假找到一些音视频的实习，但感觉项目不够，八股文基础知识等感觉可以自行学习，但在项目方面又些迷茫，不知道做一个什么样的项目不会烂大街，不知道应该去做一个什么样的项目，个人实力不强，想尽可能找一些有开源类似的项目来参考着写，希望能得到您的建议！

 - 对于大方向来说也有些迷茫，感觉形式越来越严峻，自己学历也不高，不知道音视频方向是否可以继续，流媒体呀图像处理呀WebRtc呀不知道该从何入手，不知道应该着重或者先学习什么，对于之后的学习规划很迷茫，希望您能为我未来学习方向提供一些建议！

谢谢！



1）学习开源项目

你有过 Android 平台播放器开发的经验，可以在这个基础上继续摸索一下客户端播放器的项目。很多大厂的播放器项目最初都是基于 ijkplayer（https://github.com/bilibili/ijkplayer） 这个开源播放器项目来做的，所以我建议你可以把这个项目跑起来，然后研究熟悉一下，你可以搜一搜讲解 ijkplayer 源码的文章，对其有个大概的了解，比如这篇：

https://www.jianshu.com/p/daf0a61cc1e0

2）在开源项目的基础上增加指标统计代码，并做优化对照

然后，我建议你可以看看这几篇文章里对于播放器指标的定义和优化方案，来优化一下 ijkplayer 播放器的参数配置和代码，来优化指标，这些优化经验和效果你都可以放到简历里，这个在面试中是能加分的。

- 播放秒开优化：https://mp.weixin.qq.com/s?__biz=MjM5MTkxOTQyMQ==&mid=2257487092&idx=1&sn=8585840d39805b43fb7cab28a66d9d42&chksm=a5d418a692a391b048eb1d07d7d066f21b40bdb2feed8a9bb99367ccef3bad62feacf07628e3&scene=178&cur_album_id=2140155659911233539#rd
- 播放卡顿优化：https://mp.weixin.qq.com/s?__biz=MjM5MTkxOTQyMQ==&mid=2257487093&idx=1&sn=7a81b9ba1e3f192eb8e888b5b3ceeb13&chksm=a5d418a792a391b1735e0eea124856228b170444fa2a17db1dd4a5c06a817151b77e03ad73bf&scene=178&cur_album_id=2140155659911233539#rd

3）大方向上，目前就业形势不太好，只能尽量去做好准备了。应届生找实习或找工作，面试考察主要包括：1）大学课程相关的计算机基础知识（包括：数据结构与算法、计算机网络、编程语言特性及语法、设计模式、操作系统基础原理、数据库基础等）；2）算法编程题；3）项目开发经验。

在基础知识上，可以多注意数据结构与算法、编程语言特性及语法；此外需要多刷刷 leetcode 算法题；后面加分项就是上面说的针对你的项目开发经验做好准备。

然后，就是多投简历、多面试、多总结沉淀，不管什么样的公司、岗位，都去投简历和面试，增加自己的经验，如果能拿到一些 offer，就算不是心仪的公司，你心里也会安稳很多。

4）你还可以看看几个群友求职相关的问题参考参考：

https://t.zsxq.com/LEcMT
https://t.zsxq.com/lLDl3
https://t.zsxq.com/lKQeI
https://t.zsxq.com/OhyEF
https://t.zsxq.com/0y9sv




 -->