---
title: 音视频面试题集锦第 29 期
description: 持续更新的音视频面试题集锦。
author: Keyframe
date: 2025-02-17 05:08:08 +0800
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


下面是第 29 期面试题精选：

- **1、调试 OpenGL 特效的时候图像不对，有什么调试技巧能快速排查原因？**
- **2、在实现类似 OBS 的实时的图片、GIF 贴片叠加和替换效果时遇到了性能瓶颈，请问如何实现快速的 GIF 贴片叠加和替换？**
- **3、iOS 动态图片如何获取原始视频？**
- **4、自己实现播放器时利用 FFmpeg 拿到解码后数据封装成 CVPixelbuffer 缓存用于渲染，但是缓存后数据只有几帧，但为什么内存占用有时候会有几百兆？**



## 1、调试 OpenGL 特效的时候图像不对，有什么调试技巧能快速排查原因？


1. 如果是大面积画面异常(比如黑屏或者绿屏)，先看是否在渲染时出现报错，可以使用 `glGetError()` 函数来调试。
2. 如果没有报错或者少许图像错误，就在关键节点编写调试代码将纹理转换为 yuv 数据查看具体是哪一个节点出现问题。这些调试技巧可以方便快读定位问题：
	- iOS 可以转换成 `CVPixelBuffer`，调试模式下打开小眼睛即可看到图像。
	- Android 则需要把数据写入到文件，可用 FFmpeg 打开文件查看。
	- 如果 iOS 想看二进制的数据也可以将二进制数据转换为 `NSData`，调试模式下打开小眼睛即可看到二进制内容(右下角也有 export 按钮)。
3. 查看节点的顶点坐标、纹理坐标，`vertex Shader` 和 `fragment Shader`。然后修改调试 shader 内容和坐标数据看哪个环节出现问题。


## 2、在实现类似 OBS 的实时的图片、GIF 贴片叠加和替换效果时遇到了性能瓶颈，请问如何实现快速的 GIF 贴片叠加和替换？


下面是遇到性能问题所采用的方案：

- 使用 FFmpeg 的 overlay 滤镜和 SSE 像素计算方法在推流前实时叠加图片。  
- 考虑到用户可能会传入多张图片，不能每一张都实时叠加，因此单开了一个进程利用 FFmpeg 命令先将这些贴片叠加并输出一张 PNG 图片。这样在实时叠加时，只需读取并叠加这张 PNG 图片，速度能够保证在 40ms 内，满足一秒 25 帧的帧率。  

遇到的瓶颈：

- 用户可能会传入十几张静态图或 GIF，FFmpeg 处理成 1 分钟的 MP4 并解析为 PNG 图片大约需要十几分钟，如果直播间并发进行该操作，则会更慢。    


解决方案：

想要快速的将这些滤镜和贴纸叠加到视频上需要结合 OpenGL 的能力。  

FFmpeg 提供解码原视频和贴纸图片的能力，OpenGL 特效则将所有图片渲染在一起。 

首先 OpengGL 将所有贴纸合为一张透明背景且大小与视频大小相同的纹理，然后 OpenGL 将解码后的视频数据转换成另一张纹理，最后视频纹理与贴纸纹理合二为一即可拿到叠加后的纹理。然后再将纹理转换为裸数据进行推流即可。

OpenGL 处理纹理和特效的效率很高，数据与纹理的转换耗时也不多，可以满足对性能的需求，只不过开发成本较高。


## 3、iOS 动态图片如何获取原始视频？

如果想要获取相册中实况照片对应的视频，可以使用 Photos 框架中的 `PHAsset` 类来实现。

以下是使用 Objective-C 获取实况照片视频的步骤：   

- 首先，确保你的应用有权限访问用户的相册。您需要在 `Info.plist` 文件中添加 `NSPhotoLibraryUsageDescription` 键，并提供一个描述为什么应用需要访问相册的字符串。  
- 使用相册选择页（例如 `UIImagePickerController`）来拿到你想要的实况照片，获取一个 `PHAsset`。使用 `PHImageManager` 的 `requestLivePhotoForAsset:(PHAsset *)asset...` 方法来获取每个实况照片的资源，获取一个 `PHLivePhoto` 实例。
- 调用 `[PHAssetResource assetResourcesForLivePhoto:livePhoto]` 方法即可获得两个 `PHAssetResource`。一个是对应的是图片，类型对应 `PHAssetResourceTypePhoto`，一个对应的是视频，类型对应 `PHAssetResourceTypePairedVideo`。

示例代码如下:

```java
- (void)requestLivePhotoForAsset:(PHAsset *)asset {
    PHLivePhotoRequestOptions *options = [[PHLivePhotoRequestOptions alloc] init];
    options.deliveryMode = PHImageRequestOptionsDeliveryModeHighQualityFormat;
    options.networkAccessAllowed = YES;
    
    [[PHImageManager defaultManager] requestLivePhotoForAsset:asset
                                                       targetSize:PHImageManagerMaximumSize
                                                   contentMode:PHImageContentModeAspectFill
                                                      options:options
                                               resultHandler:^(PHLivePhoto * _Nullable livePhoto, NSDictionary * _Nullable info) {
        if (livePhoto) {
            [self processLivePhoto:livePhoto ];
        } else {
            NSLog(@"Failed to fetch live photo");
        }
    }];
}
- (void)processLivePhoto:(PHLivePhoto *)livePhoto  {
    NSArray<PHAssetResource *> * result = [PHAssetResource assetResourcesForLivePhoto:livePhoto];
    [result enumerateObjectsUsingBlock:^(PHAssetResource * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        if (obj.type == PHAssetResourceTypePhoto){
            //
        } else if (obj.type == PHAssetResourceTypePairedVideo){
            //
        }
    }];
}
```

## 4、自己实现播放器时利用 FFmpeg 拿到解码后数据封装成 CVPixelbuffer 缓存用于渲染，但是缓存后数据只有几帧，但为什么内存占用有时候会有几百兆？


缓存的 `CVPixelBuffer` 需要使用 `CVPixelBufferPool` 来进行创建和释放。例如你自己创建 `CVPixelBuffer` 缓存即使你在程序中及时释放但是系统真正释放的时机可能会延迟，导致占用内存过高。

`CVPixelBufferPool` 的作用是提供一个缓冲区池，用于缓存一定数量的 `CVPixelBuffer` 对象，以便重复使用。这样，当你需要一个 `CVPixelBuffer` 时，你可以从池中获取一个已经分配好的实例，而不是每次都从头开始分配内存。当一个 `CVPixelBuffer` 不再需要时，它会被返回到池中，而不是被销毁，这样它就可以被再次使用。


---

**更多的音视频知识、面试题、技术方案干货可以进群来看：**

![限时优惠，扫码加入](assets/img/keyframe-zsxq.png)







---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
>
>你还可以加入我们的微信群和更多同行朋友来交流和讨论：
>
>![关键帧的音视频开发群](assets/img/av-wechat-group.jpg){: w="600" }
>_微信扫码进群_
{: .prompt-tip }

