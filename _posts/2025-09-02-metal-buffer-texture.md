---
title: 探索 Metal 音视频技术（2）：缓冲区与纹理
description: 本文将介绍 Metal 缓冲区与纹理。
author: Keyframe
date: 2025-09-02 18:08:08 +0800
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



这个系列文章我们来探索 Metal 音视频技术，对于想要开始学习音视频技术的朋友，这些文章是份不错的入门资料，本篇介绍缓冲区与纹理。


本章介绍 Metal 中的资源对象（`MTLResource`），用于存储“无格式”内存和“带格式”的图像数据。

`MTLResource`主要有两种类型：

- **MTLBuffer**：表示一段**无格式**的内存分配，可存放任意类型数据，常用于顶点、着色器以及计算状态数据。
- **MTLTexture**：表示一段**带格式**的图像数据，拥有指定的纹理类型和像素格式。纹理可作为顶点/片元/计算函数的输入，也可作为渲染输出（即附着点）。
    

此外，本章还讨论 `MTLSamplerState`对象。采样器本身并非资源，但在纹理查找计算时被使用。

## 1、缓冲区（Buffer）：无格式的内存分配


### 1.1、创建 Buffer

通过 `MTLDevice`的下列方法创建并返回 `MTLBuffer`：


| 方法      | 说明 |
| ----------- | ----------- |
| `newBufferWithLength:options:`      | 创建新的存储分配      |
| `newBufferWithBytes:length:options:`  | 从 CPU 内存复制数据到新分配        |
| `newBufferWithBytesNoCopy:length:options:deallocator:`    | 复用已有存储，不分配新内存      |



- 所有方法均需提供 `length`（字节数）。
- `options`为 `MTLResourceOptions`，传 `0`则使用默认值。
    

### 1.2、Buffer 常用方法

- `contents`：返回 buffer 在 CPU 的地址指针。
- `newTextureWithDescriptor:offset:bytesPerRow:`：根据 buffer 数据创建纹理（见后文）。
    

## 2、纹理（Texture）：带格式的图像数据


`MTLTexture`表示一段带格式的图像数据，可用作着色器资源或渲染附着点。其结构可为：

- 1D、2D、3D 图像
- 1D/2D 图像数组
- 立方体（6 张 2D 图）
    
像素格式由 `MTLPixelFormat`指定，详见后文。

### 2.1、创建纹理

#### 2.1.1、三种创建方式



| 创建者      | 方法 | 说明 |
| ----------- | ----------- |
| `MTLDevice`      | `newTextureWithDescriptor:`      |   全新分配     |
| `MTLTexture`  | `newTextureViewWithPixelFormat:`        |  与原纹理共享存储，仅重新解释格式    |
| `MTLBuffer`    | `newTextureWithDescriptor:offset:bytesPerRow:`      |   与 buffer 共享存储   |



> 共享存储意味着修改会相互可见，但可能禁用某些优化（如像素 swizzling、tiled）。


#### 2.1.2、使用纹理描述符 MTLTextureDescriptor

创建步骤：

1. 创建 `MTLTextureDescriptor`并设置属性：
	- `textureType`：维度与排列（如数组、立方体）。
	- `width/height/depth`：基础 mipmap 尺寸。
	- `pixelFormat`：像素存储格式。
	- `arrayLength`：1D/2D 数组纹理的元素数。
	- `mipmapLevelCount`：mipmap 级数。
	- `sampleCount`：每像素采样数（多重采样）。
	- `resourceOptions`：内存行为选项。
3.  调用 `newTextureWithDescriptor:`创建纹理。
4.  如需加载数据，使用 `replaceRegion:mipmapLevel:slice:withBytes:bytesPerRow:bytesPerImage:`。
    

示例：创建 64×64×64 的 3D 纹理，无 mipmap。

```objc
MTLTextureDescriptor* txDesc = [[MTLTextureDescriptor alloc] init];
txDesc.textureType = MTLTextureType3D;
txDesc.height = 64;
txDesc.width  = 64;
txDesc.depth  = 64;
txDesc.pixelFormat = MTLPixelFormatBGRA8Unorm;
txDesc.arrayLength = 1;
txDesc.mipmapLevelCount = 1;
id <MTLTexture> aTexture = [device newTextureWithDescriptor:txDesc];
```

#### 2.1.3、便利构造器

- `texture2DDescriptorWithPixelFormat:width:height:mipmapped:`：2D 纹理描述符。
- `textureCubeDescriptorWithPixelFormat:size:mipmapped:`：立方体纹理描述符。
    

示例：创建 64×64 无 mipmap 的 2D 纹理。

```
MTLTextureDescriptor *texDesc =
    [MTLTextureDescriptor texture2DDescriptorWithPixelFormat:MTLPixelFormatBGRA8Unorm
                                                     width:64 height:64
                                                 mipmapped:NO];
id <MTLTexture> myTexture = [device newTextureWithDescriptor:texDesc];
```

### 2.2、纹理切片（slice）

- 一个 slice = 一张 1D/2D/3D 纹理及其所有 mipmap。
- 基础级尺寸由 `width/height/depth`决定；第 `i`级尺寸为 `max(1, floor(size / 2^i))`。
- mipmap 级数 = `floor(log2(max(width, height, depth))) + 1`。
- 0 基索引：
	- 普通 1D/2D/3D：`slice = 0`。
	- 立方体：`slice 0..5`。
	- 1D/2D 数组：`slice 0..arrayLength-1`。

### 2.3、拷贝图像数据

同步拷贝方法：

- `replaceRegion:mipmapLevel:slice:withBytes:bytesPerRow:bytesPerImage:` 将 CPU 内存数据写入纹理指定 slice 与 mipmap。
- `getBytes:bytesPerRow:bytesPerImage:fromRegion:mipmapLevel:slice:` 将纹理数据读回 CPU 内存。
    

示例：将 2D 图像数据写入纹理 slice 0、mipmap 0。

```
NSUInteger myRowBytes   = width  * pixelSize;
NSUInteger myImageBytes = myRowBytes * height;
[tex replaceRegion:MTLRegionMake2D(0,0,width,height)
     mipmapLevel:0 slice:0
       withBytes:textureData
     bytesPerRow:myRowBytes
   bytesPerImage:myImageBytes];
```

## 3、像素格式（Pixel Formats）


`MTLPixelFormat`定义单个像素在 `MTLTexture`中的颜色、深度、模板数据布局，分为三类：

1. **普通格式**：8/16/32 位分量，按地址升序排列。  
	- 例：`MTLPixelFormatRGBA8Unorm`红在最前，`MTLPixelFormatBGRA8Unorm`蓝在最前。
2. **打包格式**：多个分量打包为 16/32 位，LSB→MSB。  
	- 例：`MTLPixelFormatRGB10A2Uint`三通道 10 位 + 2 位 Alpha。
3. **压缩格式**：按块压缩，仅支持 2D、2D Array、Cube，不支持 1D/2DMS/3D。
    

特殊格式：

- `MTLPixelFormatGBGR422`、`BGRG422`：YUV 采样格式，仅 2D、无 mipmap、宽度为偶数。
- sRGB 格式（如 `RGBA8Unorm_sRGB`）：采样时自动从 sRGB 转线性；渲染时自动线性转 sRGB。
    

## 4、采样器状态（Sampler State）


`MTLSamplerState`定义纹理采样时的寻址、过滤、各向异性等参数。

创建步骤：

1. 创建 `MTLSamplerDescriptor`。
2. 设置过滤、寻址等属性。
3. 通过 `newSamplerStateWithDescriptor:`创建采样器。

示例：

```
MTLSamplerDescriptor *desc = [[MTLSamplerDescriptor alloc] init];
desc.minFilter = MTLSamplerMinMagFilterLinear;
desc.magFilter = MTLSamplerMinMagFilterLinear;
desc.sAddressMode = MTLSamplerAddressModeRepeat;
desc.tAddressMode = MTLSamplerAddressModeRepeat;
desc.mipFilter        = MTLSamplerMipFilterNotMipmapped;
desc.maxAnisotropy    = 1U;
desc.normalizedCoords = YES;
desc.lodMinClamp      = 0.0f;
desc.lodMaxClamp      = FLT_MAX;

id <MTLSamplerState> sampler = [device newSamplerStateWithDescriptor:desc];
```

## 5、CPU 与 GPU 内存一致性


- **CPU → GPU**：CPU 在 `MTLCommandBuffer`提交前修改资源，GPU 才能看到。
- **GPU → CPU**：命令缓冲执行完成（`status == MTLCommandBufferStatusCompleted`）后，CPU 才能看到 GPU 对资源的修改。




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

