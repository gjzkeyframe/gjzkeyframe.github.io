---
title: 探索 Metal 音视频技术（7）：使用 AVFoundation 与 Metal 渲染 HDR 视频
description: 本文将介绍使用 AVFoundation 与 Metal 渲染 HDR 视频。
author: Keyframe
date: 2025-09-07 18:08:08 +0800
categories: [音视频基础知识]
tags: [音视频基础知识, 音视频, 视频, 渲染, Metal]
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

---


这个系列文章我们来探索 Metal 音视频技术，对于想要开始学习音视频技术的朋友，这些文章是份不错的入门资料，本篇介绍使用 AVFoundation 与 Metal 渲染 HDR 视频。


- **HDR（High Dynamic Range）**指动态范围与亮度超出传统 SDR（≈100 nits）的内容、格式与显示设备。
- 2010 年代中期起，HDR 电视与内容同步普及，如今大多数智能手机亦能拍摄 HDR 照片/视频。
- 早期 HDR 渲染需将高动态范围值“**色调映射（tonemapping）**”到 SDR 范围，此过程会丢失高光细节。
- 现代 HDR 显示虽大幅进步，但仍无法完全还原太阳等高亮光源与极黑之间的对比，因此色调映射依旧是管线必要环节。
    



## 1、什么是 EDR？


- **EDR（Extended Dynamic Range）**并非格式，而是 Apple 平台用于**自适应展示 HDR 内容**的一套技术与色彩管理方式。
- 类似于色彩管理系统：
    - 通过设置 `CAMetalLayer.colorspace`告知系统内容的真实色域；
    - EDR 进一步根据**当前显示能力与亮度设置**实时调整 HDR 值的呈现。
- 支持从 100 nits 的传统 SDR 到 1600 nits 的 Pro Display XDR 等各种显示器。
- **EDR headroom（余量）**\= 当前显示器可呈现的最大亮度 / SDR 参考白（1.0）。该值随环境光与用户调节动态变化。
- 在 EDR 语境下：
    - 1.0 表示 SDR 纯白, > 1.0 表示高光/发光体，可远高于 1.0。



## 2、色彩空间


- 色彩空间 = { 白点、RGB 三原色、传递函数（EOTF） }。
- **sRGB**：1999 定标准，覆盖 <40% 人眼可见色域。
- **Display P3**：Apple 将 DCI-P3 原色与 D65 白点、sRGB 传递函数结合，成为 Apple 设备首选。
- **Rec. 2020**：HDR 视频常用，色域远大于 P3。
- HDR 需要新的传递函数：
    - **HLG（Hybrid Log-Gamma）**：兼容 SDR，高光部分平滑滚降；
    - **PQ（Perceptual Quantizer）**：HDR10、Dolby Vision 使用，绝对亮度编码。
    
![色彩空间](assets/resource/av-basic-knowledge/metal-7-1.webp)
_色彩空间_


## 3、使用 AVPlayer 播放视频


- 高阶播放：
    
```
let asset = AVURLAsset(url: url)
let playerItem = AVPlayerItem(asset: asset)
let player = AVPlayer(playerItem: playerItem)
player.play()
```


- 检测 HDR：
    
```
let hdrTracks = try await asset.loadTracks(withMediaCharacteristic: .containsHDRVideo)
```



## 4、获取视频帧：AVPlayerItemVideoOutput


- 通过 `AVPlayerItemVideoOutput`在播放时“截取”视频帧。
- 目标 Metal 纹理格式：`.rgba16Float`（半浮点，线性 HDR）。
    
- 配置示例：

```
let videoColorProperties: [String: Any] = [
    AVVideoColorPrimariesKey: AVVideoColorPrimaries_ITU_R_2020,
    AVVideoTransferFunctionKey: AVVideoTransferFunction_Linear,
    AVVideoYCbCrMatrixKey: AVVideoYCbCrMatrix_ITU_R_2020
]
let outputSettings: [String: Any] = [
    AVVideoAllowWideColorKey: true,
    AVVideoColorPropertiesKey: videoColorProperties,
    kCVPixelBufferPixelFormatTypeKey as String: NSNumber(value: kCVPixelFormatType_64RGBAHalf)
]
let videoOutput = AVPlayerItemVideoOutput(outputSettings: outputSettings)
playerItem.add(videoOutput)
```

- 在 `MTKView.draw()`中定时拉取：
    
```
let itemTime = videoOutput.itemTime(forHostTime: CACurrentMediaTime())
guard videoOutput.hasNewPixelBuffer(forItemTime: itemTime),
      let pixelBuffer = videoOutput.copyPixelBuffer(forItemTime: itemTime, itemTimeForDisplay: nil)
else { return }
```



## 5、从 CVPixelBuffer 到 Metal Texture


- 使用 `CVMetalTextureCache`避免手动拷贝。

- 创建缓存：
    
```
var textureCache: CVMetalTextureCache!
CVMetalTextureCacheCreate(nil, nil, device, nil, &textureCache)
```

- 获取纹理：
    
```
let width = CVPixelBufferGetWidth(pixelBuffer)
let height = CVPixelBufferGetHeight(pixelBuffer)
var cvTexture: CVMetalTexture?
CVMetalTextureCacheCreateTextureFromImage(
    nil, textureCache, pixelBuffer, nil,
    .rgba16Float, width, height, 0, &cvTexture)
let frameTexture = CVMetalTextureGetTexture(cvTexture!)
```

- 需保持 `CVMetalTexture`强引用直至 Metal 命令完成，防止资源过早回收。
    



## 6、配置 CAMetalLayer 以支持 EDR


```
metalLayer.wantsExtendedDynamicRangeContent = true
metalLayer.colorspace = CGColorSpace(name: CGColorSpace.extendedLinearITUR_2020)
metalLayer.pixelFormat = .rgba16Float
```



## 7、色调映射（Tonemapping）


### 7.1、使用系统 EDR 色调映射

- 若视频包含 HDR10/HLG 元数据，可通过 `CVBufferCopyAttachment`提取：

```
let mastering = CVBufferCopyAttachment(imageBuffer, kCVImageBufferMasteringDisplayColorVolumeKey, nil)
let cll = CVBufferCopyAttachment(imageBuffer, kCVImageBufferContentLightLevelInfoKey, nil)
let ambient = CVBufferCopyAttachment(imageBuffer, kCVImageBufferAmbientViewingEnvironmentKey, nil)
```


- 生成 `CAEDRMetadata`并赋值：
    
```
if let mastering, let cll {
    let metadata = CAEDRMetadata.hdr10(displayInfo: mastering,
                                       contentInfo: cll,
                                       opticalOutputScale: 100.0)
    metalLayer.edrMetadata = metadata
} else if let ambient {
    let metadata = CAEDRMetadata.hlg(ambientViewingEnvironment: ambient)
    metalLayer.edrMetadata = metadata
}
```


- 若无元数据，可设定默认 EDR 元数据避免过曝。
    

### 7.2、自定义 Metal Shader 色调映射

- 场景：离屏渲染或需要自定义管线时。
- 步骤：
    1. 将 PQ/HLG 解码到线性 Rec.2020；
    2. 应用 Reinhard/MaxRGB 曲线映射到 EDR headroom；
    3. 输出线性值供后续处理或直接显示。
    

#### 7.2.1、PQ → EDR 示例（Metal 着色器）

```
// PQ EOTF -> nits
float3 pq_eotf(float3 v) {
    const float m1 = 0.1593017578125f;
    const float m2 = 78.84375f;
    const float c1 = 0.8359375f;
    const float c2 = 18.8515625f;
    const float c3 = 18.6875f;
    float3 vp = pow(abs(v), float3(1.0f / m2));
    return 10000.0f * pow(max(vp - c1, 0.0f) / (c2 - c3 * vp), float3(1.0f / m1));
}

// 简单 Reinhard tonemapping
float3 tonemap_maxrgb(float3 rgb, float maxLum, float headroom) {
    float maxC = max(rgb.r, max(rgb.g, rgb.b));
    if (maxC <= maxLum) return rgb * (headroom / maxLum);
    float scale = headroom * maxC / (maxC + headroom * (maxC - maxLum));
    return rgb * scale / maxC;
}
```


#### 7.2.2、HLG → EDR 示例

1.  反 OETF → 相对线性光；
2.  OOTF → 名义亮度；
3.  按 HLG 峰值白与 SDR 参考白比例缩放；
4.  使用同一 maxRGB 曲线 tonemapping。
    



## 8、总结


本文展示了如何在 Metal 中摄取并正确渲染 HDR 视频：

- 利用 **AVFoundation + Core Video**解码并获取帧；
- 借助 **CVMetalTextureCache**零拷贝生成 Metal 纹理；
- 通过 **EDR 元数据**或 **自定义着色器**完成色调映射；
- 适配 macOS/iOS 不同显示能力与亮度设置。
    

希望对你的 HDR 视频项目有所帮助！





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

