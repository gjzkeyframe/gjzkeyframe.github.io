---
title: 音视频编辑组件
description: 介绍音视频编辑组件的架构和模块设计思路。
author: Keyframe
date: 2025-02-25 16:08:08 +0800
categories: [音视频实战经验]
tags: [音视频实战经验, 音视频, 视频编辑]
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


## 1、音视频编辑组件框架

![](assets/resource/av-experience/edit-kit-1.png)


### 1.1、功能模块层

编辑 SDK 从大的功能模块上来看可以分为：


- 编辑工程管理模块：支持草稿工程资源包的导入/导出，实现非破坏性编辑与状态持久化。
- 抽帧模块：支持带特效的按需抽帧（缩略图/封面），时间精度达毫秒级。
- 预览播放器：实时渲染多轨道特效（贴纸/滤镜/转场），支持逐帧调试。
- 转码导出模块：基于硬件加速的多轨道合成导出，支持 H.264/HEVC 编码。





### 1.2、能力封装层

支持上述模块的能力封装层主要包括：

<!--  
- DecoderService：组装基础的 Demuxer、Decoder 等能力，在 Composition 层级提供支持多 Track、多 Segment 的解码能力，并支持应用于视频的贴纸、滤镜、特效等效果。封装这些综合能力给上层功能模块使用。
- EncoderService：组装基础的 Encoder、Muxer、Mixer 等能力。在 Composition 层级提供支持多 Track、多 Segment 的编码能力，并支持应用于视频的贴纸、滤镜、特效等效果。封装这些综合能力给上层功能模块使用。
- RenderService：在 Composition 层级提供支持多 Track、多 Segment 的渲染能力，并支持应用于视频的贴纸、滤镜、特效等效果。封装这些综合能力给上层功能模块使用。
- Effect：特效能力，可应用于 Composition 中的各种实体，包括 Track、Segment 等。
-->

- DecoderService：多轨道分段解码、特效预处理、硬件解码加速。
- EncoderService：多轨道混合编码、码率控制、色彩空间转换。
- RenderService：OpenGL/Metal 渲染管线、特效合成引擎、多线程调度。
- EffectSystem：插件式特效框架，支持 Track/Segment 级特效应用。


### 1.3、基础数据结构层



能力封装层依赖的基础数据结构包括：

- Composition：素材容器。用于管理视频编辑中各种音视频资源；管理剪辑操作上下文；管理贴纸、滤镜、特效应用上下文。
- Timeline：基于 Composition 计算出来的音视频时间轴。
- Track：音频、视频等轨道。在编辑场景中，图片、文字、滤镜、转场、贴纸、特效等概念也可以封装成 Track 的概念。
- Segment：音频、视频等轨道中某一段时间段落对应的段落。


```
Composition
├── Timeline (时间轴元数据)
├── Tracks
│   ├── VideoTrack
│   ├── AudioTrack
│   ├── EffectTrack
│   └── ...
└── Segments
    ├── VideoSegment (CMTimeRange)
    ├── AudioSegment
    └── TransitionSegment
```


时间轴、轨道、段落的概念如下图所示：


![](assets/resource/av-experience/edit-kit-2.png)




### 1.4、基础能力模块层

能力封装层依赖的基础能力模块包括：

- Demuxer：解封装器。
- Muxer：封装器。
- Decoder：解码器。
- Encoder：编码器。
- AssetReader：用于资源读取。
- AssetWriter：用于资源写入。
- Mixer：混合器。
- FrameBuffer：底层音视频数据结构。用于封装、编解码、混合等模块的数据流转。



## 2、实战解析

### 2.1、iOS 平台的编辑基础能力

### 2.1.1、基础数据结构

视频编辑模块的基础数据结构层次示意图如下：

![](assets/resource/av-experience/edit-kit-7.png)


#### 2.1.1.1、素材容器（AVComposition）

AVComposition 的能力可以概括为以下几点：

- 多轨道容器：支持最大 32 条异构轨道。
- 时间轴映射：通过 CMTimeRange 实现精确时序控制。
- 资源引用：采用 AVURLAsset 实现零拷贝资源管理。



AVComposition 是一个或多个轨道（AVCompositionTrack）的集合，每个轨道会根据时间线存储源媒体的文件信息，比如音频、视频等。

每个轨道由一系列轨道段（AVCompositionTrackSegment）组成，每个轨道段存储源文件的一部分媒体数据，比如 URL、轨道 ID（CMPersistentTrackID）、时间映射（CMTimeRange）等。

其中 URL 指定文件的源容器，轨道 ID 指定需要用到的源文件轨道，而时间映射指定的是源轨道的时间范围，并且还指定其在合成轨道上的时间范围。

![](assets/resource/av-experience/edit-kit-3.png)



#### 2.1.1.2、轨道（AVCompositionTrack）

视频类型轨道，已经支持的比较好，对应的轨道类型（AVMediaType）是 AVMediaTypeVideo。

音频类型轨道，已经支持的比较好，对应的轨道类型（AVMediaType）是 AVMediaTypeAudio。

图片类型轨道，在视频类型的轨道基础上扩展出图片类型轨道，即通过空的视频文件生成视频轨道，将所选的图片作为帧图提供给混合器。

其他类型轨道：AVFoundation 提供了 AVVideoCompositionCoreAnimationTool 这个工具，能够方便的将 CoreAnimation 框架内的内容应用到视频帧图上。所以添加文字、贴纸等功能，在编辑时可以通过 UIKit 创建了一系列预览视图来实现，在导出时则转换到该工具的 CALayer 上来导出。AVVideoCompositionCoreAnimationTool 是对 CoreAnimation 的支持，可以将 CALayer 及其支持的动画引入到视频画面的合成当中来。使用它有两种方式，一是将 Layer 内容绘制到独立的轨道中参与画面合成；二是将视频帧绘制到 Layer 上，再使用 Layer 的内容生成视频帧。需要注意的是：这种方式只适用于导出，播放时需要使用 AVSynchronizedLayer。

```
// AVComposition 中包含一组 AVCompositionTrack：
@property (nonatomic, readonly) NSArray<AVCompositionTrack *> *tracks;
```

```
// AVVideoComposition 中包含 AVVideoCompositionCoreAnimationTool：
@property (nonatomic, readonly, retain, nullable) AVVideoCompositionCoreAnimationTool *animationTool;
```

#### 2.1.1.3、段落（AVCompositionTrackSegment）


使用 AVCompositionTrackSegment 即可。表示轨道中的某个时间段，作为段落编辑的对象。

```
// AVCompositionTrack 中包含一组 AVCompositionTrackSegment：
@property (nonatomic, readonly, copy) NSArray<AVCompositionTrackSegment *> *segments;
```

#### 2.1.1.4、指令（AVVideoCompositionInstruction/AVVideoCompositionLayerInstruction）

AVVideoCompositionInstruction 代表了对一个时间段内的视频做进一步调节的指令，通过 timeRange 进行关联。

```
// AVVideoComposition 中包含一组 AVVideoCompositionInstruction：
@property (nonatomic, readonly, copy) NSArray<id <AVVideoCompositionInstruction>> *instructions;
```

比如设置该时间段内画布的背景色；对于一个时间段，可能需要处理多个视频轨，这里通过 AVVideoCompositionLayerInstruction 代表对每一个视频轨的处理指令，AVMutableVideoCompositionInstruction 中包含了一组 AVVideoCompositionLayerInstruction，通过 trackID 进行关联。可通过其可变子类 AVMutableVideoCompositionLayerInstruction 设置 transform、opacity 或者 crop。




```
// AVMutableVideoCompositionInstruction 中包含一组 AVVideoCompositionLayerInstruction：
@property (nonatomic, readonly, copy) NSArray<AVVideoCompositionLayerInstruction *> *layerInstructions;
```

指令表示关联到指定的视频段落，进行图像处理，绘制每一帧画面。一个指令可以关联多条视频轨道，从这些视频轨道的指定时间段内获取帧图，作为画面编辑的对象。


指令中画面编辑的具体实现方案，可以采用 CoreImage 框架。CoreImage 本身提供了一些内置的图片实时处理功能。CoreImage 不支持的特效，我们通过自定义 CIKernel 的方式来实现。











### 2.1.2、音频处理器（AVAudioMix）

针对添加音乐、混音的功能，我们使用 AVAudioMix 完成。

AVAudioMix 可以通过 AVAudioMixInputParameters 指定任意轨道的任意时间段音量。

![](assets/resource/av-experience/edit-kit-4.png)


### 2.1.3、视频处理器（AVVideoComposition）

我们还可以使用 AVVideoComposition 来直接处理 composition 中的视频轨道。处理一个单独的 video composition 时，你可以指定它的渲染尺寸、缩放比例、帧率等参数并输出最终的视频文件。通过一些针对 video composition 的指令（AVVideoCompositionInstruction 等），我们可以修改视频的背景颜色、应用 layer instructions。

这些 layer instructions（AVVideoCompositionLayerInstruction 等）可以用来对 composition 中的视频轨道实施图形变换、添加图形渐变、透明度变换、增加透明度渐变。此外，你还能通过设置 video composition 的 animationTool 属性来应用 Core Animation Framework 框架中的动画效果。

![](assets/resource/av-experience/edit-kit-5.png)



### 2.1.4、抽帧能力（AVAssetImageGenerator）

AVAssetImageGenerator 是一个视频快照生成器，用于截取视频中特定时间（序列）的快照（序列）。


### 2.1.5、预览能力（AVPlayer）

AVPlayer 播放器对应的播放资源对象是 AVPlayerItem。

```
+ (AVPlayer *)playerWithPlayerItem:(nullable AVPlayerItem *)item;
```

AVPlayerItem 是从 AVAsset 中获得。


```
+ (AVPlayerItem *)playerItemWithAsset:(AVAsset *)asset;
```


AVComposition 是继承自 AVAsset。

所以，AVPlayer 可以把 AVComposition 作为播放资源来进行预览播放。



### 2.1.6、导出能力（AVAssetExportSession）


导出的步骤比较简单，只需要把上面几步创建的处理对象赋值给导出类对象就可以导出最终的产品。

![](assets/resource/av-experience/edit-kit-6.png)




## 3、典型问题集锦

1、轨道数量限制会导致什么问题？


问题：导出时出现 AVErrorExceededMaxDecoders 错误。

原因：iOS硬件解码器最大并发数为 4。

解决方案：

- 轨道预处理阶段合并相似轨道。
- 使用 AVVideoComposition 进行轨道虚拟化。
- 采用分时复用解码策略。



2、倒播功能的实现怎么优化性能？

问题：最初的实现方案为导出一个新的视频文件，帧序列是原视频文件中的倒序。如果原视频文件很大，用户只裁剪了一种的一段，对这一段进行倒播时，仍然会将原视频文件进行倒序处理，导出一份新的视频，是一个十分耗时的操作。

解决方案：解码器保持正向读取，渲染层进行时间映射。


3、缩略图的实现怎么优化性能？

可以尝试以下优化策略：

- 在不影响用户体验的情况下，使用低分辨率的图片进行缩略图预览，导出时再使用原图。
- 抽帧接口异步逐帧回调。
- 精准抽帧与非精准抽帧。
- 数据转换和缩放优化。
- 解码丢弃非参考帧。
- 解码器复用池。
- 抽帧缩略图缓存。



4、怎么实现合拍？


- 1）录制过程中，始终是「播放器的 texture」和「相机采集的 texture」进行 mix。
- 2）录制过程中，如果开启倍速 n，音频和视频录制都按照 1 倍速录制，合拍对应的视频和音频按照 1/n 倍速播放。
- 3）录制的结果 mix，视频部分，按照两个 texture 的 mix 出来的效果乘以 n 倍速处理成 video_track。
- 4）录制的结果 mix，音频部分，录制的音频乘以 n 倍速来处理成 audio_track_1，播放的音频按照 1 倍速处理成 audio_track_2。
- 5）将上面的 video_track、audio_track_1、audio_track_2 处理为一个 composition 交给后续的编辑流程。








参考：

- [视频编辑框架设计](https://juejin.cn/post/6844903927876599815)
- [理解 Cabbage 框架的基础设计](https://waguan.cc/ios/2020/08/21/cabbage-arch/)
