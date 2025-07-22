---
title: 如何正确将 HDR 视频转换成 SDR 视频：一手的实战经验
description: 介绍将 HDR 视频转换成 SDR 视频技术方案和源码。
author: Keyframe
date: 2025-02-25 06:08:08 +0800
categories: [音视频实战经验]
tags: [音视频实战经验, 音视频, 视频编辑, HDR, SDR, 编解码]
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


## 1、什么是 SDR 和 HDR？

**SDR**（Standard Dynamic Range）即标准动态范围，是一种基于亮度、对比度、颜色特性，以及 CRT 显示器的局限性来展示视频的技术。这里说的`动态范围`一般是指亮度范围，更大的亮度范围可以支持更高的对比度。SDR 的支持的亮度范围在 0.1nit 到 100nit 之间，使用 Rec.709/sRGB 色域，并使用 Gamma 曲线来作为它的电光转换函数（Electro-Optical Transfer Function，EOTF）。

**HDR**（High Dynamic Range）即高动态范围，是对 SDR 的升级，是一种提升视频显示质量的技术。HDR 改变了视频和图像的亮度和颜色信息在信号中的表示方式，从而支持**更大的亮度范围（0.0005-10000nit）**、**更宽广的色域（BT.2020）**、**更高精度的量化（10bit 或 12bit）**。因此 HDR 视频画面可以展现出更多的亮部和暗部细节，画面拥有丰富的色彩和生动自然的细节表现，因此画面更接近人眼所见；SDR 视频的色彩饱和度以及画面对比度则不如 HDR 视频，相比 HDR 视频，SDR 视频的画面，给人一种暗淡不自然的观感，同时在亮部以及暗部细节上都有很明显的缺失。



![HDR 和 SDR 的视觉差异](assets/resource/av-experience/transcode-hdr-to-sdr.png)
_HDR 和 SDR 的视觉差异_




![HDR 和 SDR 信息处理区别](assets/resource/av-experience/transcode-hdr-to-sdr-6.jpg)
_HDR 和 SDR 信息处理区别_






HDR 技术有着不同的标准，其中常见的有四个：HDR10、HDR10+、Dolby Vision、HLG。

- `HDR10` 是比较基础的一个版本，也是一个开放的标准，于 2014 年被采用。HDR10 由于其易用性和免许可费而获得广泛的接受。该标准描述了符合 UHDTV Rec.ITU-R BT.2020 标准建议的视频内容。HDR10 采用的是 PQ EOTF 转换曲线，与 SDR 显示器不兼容。HDR10 采用了静态元数据，不能满足不同场景或者不同帧调色的需求，所以 HDR10 的效果展现能力比较有限。
- `HDR10+` 是三星提出的用于对抗 Dolby Vision 的技术，在 HDR10 的基础上，添加了动态元数据的支持。可以针对每一个视频场景或者视频帧进行亮度和色彩的调节，支持动态色调映射，向后也可兼容 HDR10 的格式。
- `Dolby Vision` 是杜比公司具有知识产权的技术，需要授权费用才能使用。也因为需要授权费用，所以内容还不是特别丰富。Dolby Vision 实际是国际上首个推出的商业化版本的 HDR 标准，具备非常强的竞争力。它采用动态元数据，可以最高支持 10000nit 的峰值亮度。为了增强码流播放和显示兼容性，设计了众多的 Profile 支持不同的应用。
- `HLG` 标准出现于 2015 年，是由英国 BBC 公司和日本的 NHK 电视台共同开发，也被广泛采用。该标准描述了符合 BT.2020 要求的内容。如前所述，HLG 广泛地应用到广电系统中，有很好的兼容性。


下面不同格式 HDR 的参数对比：

![不同格式 HDR 的参数对比](assets/resource/av-experience/transcode-hdr-to-sdr-1.png)
_不同格式 HDR 的参数对比_


>上面的介绍中，提到了`元数据`，这里简单介绍一下：
>
>HDR 的元数据是用来描述视频或图像处理过程中的关键信息或者特征，主要有两种：`静态元数据`和`动态元数据`。
>
>静态元数据规定了整个片子像素级别最大亮度上限，在 ST 2086 中有标准化的定义。静态元数据的缺点是必须做全局的色调映射，没有足够的调节空间，兼容性不好。
>
>动态元数据可以很好地解决这个问题。动态元数据主要有两个方面的作用：与静态元数据相比，它可以在每一个场景或者每一帧画面，给调色师一个发挥的空间，以展现更丰富的细节；另一个方面，通过动态元数据，在目标显示亮度上做色调映射，可以最大程度在目标显示器上呈现作者的创作意图。
>
>SMPTE ST 2094 标准中定义了一系列的动态元数据。ST 2094-10、ST 2094-20、ST 2094-30、ST 2094-40 分别给出了杜比、飞利浦、特艺和三星四家公司的动态元数据和色域转换的方案。


参考：

- [Standard-dynamic-range](https://en.wikipedia.org/wiki/Standard-dynamic-range_video "Standard-dynamic-range video")
- [High-dynamic-range](https://en.wikipedia.org/wiki/High-dynamic-range_television "High-dynamic-range")
- [HDR 技术趋势浅析](https://mp.weixin.qq.com/s?__biz=MzU1NTEzOTM5Mw==&mid=2247521774&idx=1&sn=d2b1a7a705e742c65697760f7641ec9b&scene=21#wechat_redirect "HDR 技术趋势浅析")

## 2、HDR 在应用中可能遇到什么问题？

因为 HDR 是一套涉及到颜色空间和设备显示特性的技术方案，所以要实现对 HDR 的支持，需要满足：

- 视频资源满足 HDR 标准
- 显示设备支持 HDR 显示

由于 HDR 技术方案涉及到颜色空间，使得在相机采集、编码、解码、渲染到屏幕上这一整个流程里面，凡是涉及到要对颜色信息进行理解和处理的节点，都需要实现对 HDR 的支持才能保证最终正确地呈现出它的特性。这就很容易出现由于某一个环节缺失对 HDR 的支持而造成最终的呈现问题。

所以，作为一种新的技术标准，HDR 在应用中最大的问题是**兼容性问题**，这里最大的兼容性问题是 HDR 与 SDR 新旧技术之间的兼容，此外还有不同 HDR 标准之间的兼容。

HDR 在应用中最常见的问题有：

- **视频播放黑屏。**
- **视频播放色彩异常。**
- **视频画面较暗。**
- **视频画面发灰。**

**这种兼容性问题是怎么造成的呢？**

**1）颜色位深**

核心原因之一是颜色位深的差别。SDR 使用的颜色空间是使用 8bit 的位深，而 HDR 则使用的颜色空间是使用 10bit 或 12bit 的位深。这样一来在表示信息时就有容量差异了。在视频处理的流程中，如果从 HDR 向 SDR 转换时，如果处理不合理就会出现破坏性的信息丢失，导致最终视频展示效果的异常。

![8bit vs. 10bit](assets/resource/av-experience/transcode-hdr-to-sdr-3.png)
_8bit vs. 10bit_





**2）色域**

核心原因之二是色域的差别。色域是指一个颜色空间所能表示的所有颜色的集合，色域越广，所能表示的色彩越丰富。

SDR 使用 BT.709 颜色空间标的色域，而 HDR 则使用 BT.2020 颜色空间的更广的色域。宽色域向窄色域兼容时，同样也有信息丢失的问题，不合理的处理也会导致最终视频展示效果的异常。

![BT.709 vs. BT.2020](assets/resource/av-experience/transcode-hdr-to-sdr-2.png)
_BT.709 vs. BT.2020_






**3）转换函数（Transfer Function）**

另外一个带来兼容性问题的原因是转换函数的差别。

这里的转换函数是指光电转换函数（Optical-Electro Transfer Function）或电光转换函数（Electro-Optical Transfer Function）。

我们为什么要使用转换函数呢？人眼对于物理世界的感知是非线性的，对于中等亮度和暗部区间的敏感程度远高于高亮度区间。为了讨好人眼这种对不同亮度的非线性的敏感度，我们在设备的采集电路中采集到光信号向电信号转换时，通常会将其转换为非线性信号，这里用到了 OETF，这样对非线性信号进行编码时，可以用更多的码率来编码人眼敏感的中等亮度或暗部细节，从而使得编码在讨好人眼上有更好的 ROI。而在显示时，我们要再将非线性信号还原为线性光展示给人眼，这时候则要用到 EOTF。


![SDR 和 HDR 的非线性编码](assets/resource/av-experience/transcode-hdr-to-sdr-8.jpg)
_SDR 和 HDR 的非线性编码_




传统的 SDR 使用 BT.709 Gamma 曲线作为转换函数，对高亮部分进行了截断，可以表达的亮度动态范围有限，最大亮度只有 100nit。而在 HDR 技术中，增加了高亮部分细节的表达，大大扩展了亮度的动态范围，Gamma 曲线已经不能满足最大亮度的需求，HDR 则使用 PQ（Perceptual Quantizer，感知量化）或 HLG（Hybrid Log Gamma，混合对数伽马）曲线作为转换函数。

![SDR 和 HDR 的转换函数](assets/resource/av-experience/transcode-hdr-to-sdr-7.png)
_SDR 和 HDR 的转换函数_




不同 HDR 转换函数的设计初衷不同，下面是 PQ 和 HLG 的区别：


- `PQ（Perceptual Quantizer，感知量化）曲线`的设计更接近人眼的特点，亮度表达更准确。基于人眼的对比敏感度函数（Contrast Sensitivity Function，CSF），在 SMPTE ST 2084 标准中规定了 EOTF 曲线。亮度范围可从最暗 0.00005nit 到最亮 10000nit。PQ 曲线最早是由 Dolby 公司开发的，并且在 ST 2084 中进行了标准化。
- `HLG（Hybrid Log Gamma，混合对数伽马）曲线`是另外一个重要的 HDR 转换函数曲线，由 BBC 和 NHK 公司开发。这个曲线与 PQ 曲线不同，HLG 规定的是 OETF 曲线，因为在低亮度区域基本与 Gamma 曲线重合，所以提供了与 SDR 显示设备很好的兼容性，在广播电视系统里有着广泛的应用。HLG 曲线最早在 ARIB STD-B67 中进行了标准化，后面也进入了 ITU-R BT.2100。

![OETF 和 EOTF](assets/resource/av-experience/transcode-hdr-to-sdr-5.png)
_OETF 和 EOTF_



如果使用的转换函数不匹配，就会出现信息错误而影响最终的视频展示效果。





**4）设备显示亮度**

其他的原因还有设备显示亮度支持的问题。HDR 的使用也需要硬件设备的支持，电脑或手机的屏幕是否能支持更高的亮度和对比度也会影响最终呈现视频的效果。

下图是人眼能感受的亮度范围，以及 SDR 和 HDR 所能支持的亮度范围的对比：

![人眼、SDR、HDR 的亮度范围](assets/resource/av-experience/transcode-hdr-to-sdr-4.png)
_人眼、SDR、HDR 的亮度范围_



一般台式电脑显示器的持续亮度在 350 尼特左右，有些专业显示器的会高一点，但大部分持续不了较长时间。做的比较好的，比如苹果的 Pro Display XDR 显示器则可以达到 1000 尼特全屏持续亮度，峰值亮度达到 1600 尼特。

此外，由于 HDR 有多种格式，不同格式之间的参数区别也可能是影响视频最终呈现效果的原因。




我们再回头看前面提到的 HDR 在应用中常见的问题，可以知道原因大致如下：

- **视频播放黑屏。**可能是在播放 HDR 视频时，解码器不支持 BT.2020 颜色空间（定义了 10bit 颜色位深），出现解码错误造成视频黑屏。
- **视频播放色彩异常。**可能是播放器渲染模块不支持 BT.2020 颜色空间导致渲染色彩异常的问题。
- **视频画面较暗。**可能是显示器不支持 HDR 的亮度范围，无法识别视频数据中的亮度信息导致。
- **视频画面发灰。**可能是显示器不支持 HDR 颜色空间 BT.2020 的宽色域，导致显示器上的色彩饱和度不足。



参考：

- [HDR 技术趋势浅析](https://mp.weixin.qq.com/s?__biz=MzU1NTEzOTM5Mw==&mid=2247521774&idx=1&sn=d2b1a7a705e742c65697760f7641ec9b&scene=21#wechat_redirect)
- [微博 HDR 视频的落地实践](https://mp.weixin.qq.com/s?__biz=MzU1NTEzOTM5Mw==&mid=2247520083&idx=1&sn=bc1768947ce8671a43a912b2a6917c3b&scene=21#wechat_redirect)
- [高动态范围电视的理论原理](https://transaudiovideo.com/it/news/principi-teorici-di-televisione-high-dynamic-range)


## 3、如何正确将 HDR 视频转换成 SDR 视频？

要解决 HDR 在应用中的问题，最好的体验当然是全链路支持 HDR 技术标准来呈现最佳的视频图像视觉。但现实是，我们还是有很多现存的显示设备是不支持 HDR 的，对于这种情况，我们则需要在这些设备上将 HDR 视频转换成 SDR 视频，并保证转换过程对信息的合理处理从而尽量降低视频视觉体验的损失。这就是我们接下来要讲的：**如何正确将 HDR 视频转换成 SDR 视频？**

简单来讲，HDR 视频转 SDR 视频需要下面几步：

- 1、HDR 非线性电信号转为 HDR 线性光信号（EOTF）
- 2、HDR 线性光信号做颜色空间转换（Color Space Converting），通常是从 BT.2020 转换到 BT.709
- 3、HDR 线性光信号色调映射为 SDR 线性光信号（Tone Mapping）
- 4、SDR 线性光信号转 SDR 非线性电信号（OETF）

HDR 到 SDR 视频的转换，经历了亮度动态范围和色彩空间的压缩，亮度范围从 `[0.0005, 10000]nit` 压缩到 `[0.1, 100]nit`，颜色空间从 BT.2020 转换到 BT.709；同时颜色位深也由 10bit 降低到 8bit；视频信号可用的色阶数量从 1024 降低到 256 个，减少了 75%；同时光电转换函数 EOTF 也会变化，从 PQ 或 HLG 变为 BT.709 Gamma。

实现 HDR 转 SDR 视频的方案有下面几种可供参考：


### 3.1、使用 FFmpeg filter 实现转换

利用 FFmpeg 命令行实现 HDR 转 SDR，主要是应用了 FFmpeg 中 `zscale`（依赖 zimg）以及 `tonemap` 这两个 filter，要使用 zscale，必须确认 FFmpeg 编译时有开启 `--enable-libzimg`。其中 `zimg` 是一个实现颜色空间转换的三方库：[https://github.com/sekrit-twc/zimg](https://github.com/sekrit-twc/zimg "zimg")。


FFmpeg 实现 HDR 转 SDR 的命令如下：

```sh
ffmpeg -i <input> -vf \  # -vf 后面表示是 video filter 的一系列命令 
zscale=t=linear:npl=100, \  # 1）非线性转线性。指定 zscale 模块的转换函数为 linear，输入参数为 npl=100，npl 表示标称峰值亮度（nominal peak luminance）
format=gbrpf32le, \  # 转换格式为 gbr，用 little end 32 位浮点类型存储 10bit 颜色通道
zscale=p=bt709, \  # 2）颜色空间转换。指定 zscale 模块的色域为 bt709
tonemap=tonemap=hable:desat=0, \  # 3）色调映射。指定 tonemapping 转换算法为 hable，输入参数 desat=0
zscale=t=bt709:m=bt709:r=tv, \  # 4）线性转非线性。指定 zscale 模块的转换函数为 bt709，转换矩阵为 bt709，range 为 tv.limited
format=yuv420p \  # 转换格式为 yuv420p
<output>
```

上面的命令是一个串联执行流程，顺序也对应上面我们提到的 HDR 和 SDR 转换流程。下面我们以一个使用了 PQ 标准的 HDR 视频为例来介绍一下几个关键步骤的相关代码：

**1）非线性颜色数字信号经过 EOTF 转换为线性的模拟光信号。**

第一个步骤对应上的命令参数：`zscale=t=linear:npl=100`，表示目标是要转换为 linear 线性模拟光信号，标称峰值亮度（nominal peak luminance）为 100。

我们可以从 [zimg](https://github.com/sekrit-twc/zimg "zimg") 源代码中找到关键函数：


```c++
// src/zimg/colorspace/graph.cpp

std::vector<ColorspaceNode> get_neighboring_colorspaces(const ColorspaceDefinition &csp)
{
  zassert_d(is_valid_csp(csp), "invalid colorspace");

  std::vector<ColorspaceNode> edges;

  auto add_edge = [&](const ColorspaceDefinition &out_csp, auto func)
  {
    edges.emplace_back(out_csp, std::bind(func, csp, out_csp, std::placeholders::_1, std::placeholders::_2));
  };

  if (csp.matrix == MatrixCoefficients::RGB) {
    constexpr MatrixCoefficients special_matrices[] = {
      MatrixCoefficients::UNSPECIFIED,
      MatrixCoefficients::RGB,
      MatrixCoefficients::REC_2020_CL,
      MatrixCoefficients::CHROMATICITY_DERIVED_NCL,
      MatrixCoefficients::CHROMATICITY_DERIVED_CL,
      MatrixCoefficients::REC_2100_LMS,
      MatrixCoefficients::REC_2100_ICTCP,
    };

    // RGB can be converted to conventional YUV.
    for (auto matrix : all_matrix()) {
      if (std::find(std::begin(special_matrices), std::end(special_matrices), matrix) == std::end(special_matrices))
        add_edge(csp.to(matrix), create_ncl_rgb_to_yuv_operation);
    }
    if (csp.primaries != ColorPrimaries::UNSPECIFIED)
      add_edge(csp.to(MatrixCoefficients::CHROMATICITY_DERIVED_NCL), create_ncl_rgb_to_yuv_operation);

    // Linear RGB can be converted to other transfer functions and primaries; also to combined matrix-transfer systems.
    if (csp.transfer == TransferCharacteristics::LINEAR) {
      for (auto transfer : all_transfer()) {
        if (transfer != csp.transfer && transfer != TransferCharacteristics::UNSPECIFIED) {
          add_edge(csp.to(transfer), create_linear_to_gamma_operation);
          if (csp.primaries != ColorPrimaries::UNSPECIFIED)
            add_edge(csp.to(transfer).to(MatrixCoefficients::CHROMATICITY_DERIVED_CL), create_cl_rgb_to_yuv_operation);
        }
      }
      if (csp.primaries != ColorPrimaries::UNSPECIFIED) {
        for (auto primaries : all_primaries()) {
          if (primaries != csp.primaries && primaries != ColorPrimaries::UNSPECIFIED)
            add_edge(csp.to(primaries), create_gamut_operation);
        }
      }

      add_edge(csp.to(MatrixCoefficients::REC_2020_CL).to(TransferCharacteristics::REC_709), create_cl_rgb_to_yuv_operation);

      if (csp.primaries == ColorPrimaries::REC_2020)
        add_edge(csp.to(MatrixCoefficients::REC_2100_LMS), create_ncl_rgb_to_yuv_operation);
    } else if (csp.transfer != TransferCharacteristics::UNSPECIFIED) {
      // Gamma RGB can be converted to linear RGB.
      add_edge(csp.to_linear(), create_gamma_to_linear_operation);
    }
  } else if (csp.matrix == MatrixCoefficients::REC_2020_CL || csp.matrix == MatrixCoefficients::CHROMATICITY_DERIVED_CL) {
    add_edge(csp.to_rgb().to_linear(), create_cl_yuv_to_rgb_operation);
  } else if (csp.matrix == MatrixCoefficients::REC_2100_LMS) {
    // LMS with ST_2084 or ARIB_B67 transfer functions can be converted to ICtCp and also to linear transfer function.
    if (csp.transfer == TransferCharacteristics::ST_2084 || csp.transfer == TransferCharacteristics::ARIB_B67) {
      add_edge(csp.to(MatrixCoefficients::REC_2100_ICTCP), create_lms_to_ictcp_operation);
      add_edge(csp.to(TransferCharacteristics::LINEAR), create_gamma_to_linear_operation);
    }
    // LMS with linear transfer function can be converted to RGB matrix and to ARIB_B67 and ST_2084 transfer functions.
    if (csp.transfer == TransferCharacteristics::LINEAR) {
      add_edge(csp.to_rgb(), create_ncl_yuv_to_rgb_operation);
      add_edge(csp.to(TransferCharacteristics::ST_2084), create_linear_to_gamma_operation);
      add_edge(csp.to(TransferCharacteristics::ARIB_B67), create_linear_to_gamma_operation);
    }
  } else if (csp.matrix == MatrixCoefficients::REC_2100_ICTCP) {
    // ICtCp with ST_2084 or ARIB_B67 transfer functions can be converted to LMS.
    if (csp.transfer == TransferCharacteristics::ST_2084 || csp.transfer == TransferCharacteristics::ARIB_B67)
      add_edge(csp.to(MatrixCoefficients::REC_2100_LMS), create_ictcp_to_lms_operation);
  } else if (csp.matrix != MatrixCoefficients::UNSPECIFIED) {
    // YUV can be converted to RGB.
    add_edge(csp.to_rgb(), create_ncl_yuv_to_rgb_operation);
  }

  return edges;
}
```

EOTF 转换会先调用 `get_neighboring_colorspaces` 函数，并创建 Gamma RGB → Linear RGB 的转换操作，即调用 `create_gamma_to_linear_operation` 函数。



```c++
// src/zimg/colorspace/operation.cpp

std::unique_ptr<Operation> create_gamma_to_linear_operation(const ColorspaceDefinition &in, const ColorspaceDefinition &out, const OperationParams &params, CPUClass cpu)
{
  zassert_d(in.primaries == out.primaries, "primaries mismatch");
  zassert_d((in.matrix == MatrixCoefficients::RGB || in.matrix == MatrixCoefficients::REC_2100_LMS) &&
            (out.matrix == MatrixCoefficients::RGB || out.matrix == MatrixCoefficients::REC_2100_LMS), "must be RGB or LMS");
  zassert_d(in.transfer != TransferCharacteristics::LINEAR && out.transfer == TransferCharacteristics::LINEAR, "wrong transfer characteristics");
 
  if (in.transfer == TransferCharacteristics::ARIB_B67 && use_display_referred_b67(in.primaries, params))
    return create_inverse_arib_b67_operation(ncl_rgb_to_yuv_matrix_from_primaries(in.primaries), params);
  else
    return create_inverse_gamma_operation(select_transfer_function(in.transfer, params.peak_luminance, params.scene_referred), params, cpu);
}
```

由于现在是按照 PQ 标准做转换，所以 `in.transfer` 不等于 `ARIB_B67`(这是 HLG 标准)，会接着调用 `create_inverse_gamma_operation` 函数。


```c++
// src/zimg/colorspace/operation_impl.cpp

// GammaOperationC 类：
class GammaOperationC final : public Operation {
  gamma_func m_func;
  float m_prescale;
  float m_postscale;
public:
  GammaOperationC(gamma_func func, float prescale, float postscale) :
    m_func{ func },
    m_prescale{ prescale },
    m_postscale{ postscale }
  {}

  void process(const float * const *src, float * const *dst, unsigned left, unsigned right) const override
  {
    EnsureSinglePrecision x87;

    for (unsigned p = 0; p < 3; ++p) {
      const float *src_p = src[p];
      float *dst_p = dst[p];

      for (unsigned i = left; i < right; ++i) {
        dst_p[i] = m_postscale * m_func(src_p[i] * m_prescale);
      }
    }
  }
};

// ...

// create_inverse_gamma_operation 函数：
std::unique_ptr<Operation> create_inverse_gamma_operation(const TransferFunction &transfer, const OperationParams &params, CPUClass cpu)
{
  std::unique_ptr<Operation> ret;
 
#if defined(ZIMG_X86)
  ret = create_inverse_gamma_operation_x86(transfer, params, cpu);
#elif defined(ZIMG_ARM)
  ret = create_inverse_gamma_operation_arm(transfer, params, cpu);
#endif
  if (!ret)
    ret = std::make_unique<GammaOperationC>(transfer.to_linear, 1.0f, transfer.to_linear_scale);
 
  return ret;
}
```

暂时不考虑平台加速的代码，这里则是要构建了一个 `GammaOperationC`。构建 `GammaOperationC` 最重要的参数是转换函数对象：`TransferFunction`。由于这里是要转为线性光信号，所以是取的是 `TransferFunction` 对象的 `to_linear` 和 `to_linear_scale` 属性。这个对象是之前调用 `select_transfer_function` 函数来获得的，代码如下：

```c++
// src/zimg/colorspace/gamma.cpp

TransferFunction select_transfer_function(TransferCharacteristics transfer, double peak_luminance, bool scene_referred)
{
  zassert_d(!std::isnan(peak_luminance), "nan detected");
 
  TransferFunction func{};
 
  func.to_linear_scale = 1.0f;
  func.to_gamma_scale = 1.0f;
 
  switch (transfer) {
  // ... 
    case TransferCharacteristics::REC_709:
    func.to_linear = scene_referred ? rec_709_inverse_oetf : rec_1886_eotf;
    func.to_gamma = scene_referred ? rec_709_oetf : rec_1886_inverse_eotf;
    break;
  case TransferCharacteristics::ST_2084:
    func.to_linear = scene_referred ? st_2084_inverse_oetf : st_2084_eotf;
    func.to_gamma = scene_referred ? st_2084_oetf : st_2084_inverse_eotf;
    func.to_linear_scale = static_cast<float>(ST2084_PEAK_LUMINANCE / peak_luminance);
    func.to_gamma_scale = static_cast<float>(peak_luminance / ST2084_PEAK_LUMINANCE);
    break;
  case TransferCharacteristics::ARIB_B67:
    func.to_linear = scene_referred ? arib_b67_inverse_oetf : arib_b67_eotf;
    func.to_gamma = scene_referred ? arib_b67_oetf : arib_b67_inverse_eotf;
    func.to_linear_scale = scene_referred ? 12.0f : static_cast<float>(1000.0 / peak_luminance);
    func.to_gamma_scale = scene_referred ? 1.0f / 12.0f : static_cast<float>(peak_luminance / 1000.0);
    break;
  default:
    error::throw_<error::InternalError>("invalid transfer characteristics");
    break;
  }
 
  return func;
}
```

从上面的代码中我们可以知道，因为我们这里 PQ 曲线对应的是 SMPTE ST 2084 标准，转换函数 `to_linear` 即 `st_2084_eotf`，`to_linear_scale` 则为 `ST2084_PEAK_LUMINANCE / peak_luminance`。


`st_2084_eotf` 函数的实现如下：


```c++
// src/zimg/colorspace/gamma.cpp

constexpr float ST2084_M1 = 0.1593017578125f;
constexpr float ST2084_M2 = 78.84375f;
constexpr float ST2084_C1 = 0.8359375f;
constexpr float ST2084_C2 = 18.8515625f;
constexpr float ST2084_C3 = 18.6875f;   
constexpr float FLT_MIN 1.17549435082228750797e-38F

float st_2084_eotf(float x) noexcept
{
  // Filter negative values to avoid NAN.
  if (x > 0.0f) {
    float xpow = zimg_x_powf(x, 1.0f / ST2084_M2);
    float num = std::max(xpow - ST2084_C1, 0.0f);
    float den = std::max(ST2084_C2 - ST2084_C3 * xpow, FLT_MIN);
    x = zimg_x_powf(num / den, 1.0f / ST2084_M1);
  } else {
    x = 0.0f;
  }

  return x;
}
```

到这里，处理一个使用了 PQ 标准的 HDR 视频，去获取对应 EOTF 转换函数及参数的核心步骤就介绍完了。总结起来主要步骤如下：

```
get_neighboring_colorspaces
    -> create_gamma_to_linear_operation
        -> create_inverse_gamma_operation
            -> select_transfer_function
                case TransferCharacteristics::ST_2084:
                    func.to_linear = scene_referred ? st_2084_inverse_oetf : st_2084_eotf;
                    func.to_linear_scale = static_cast<float>(ST2084_PEAK_LUMINANCE / peak_luminance);
                    break;
```


**2）HDR 线性光信号做颜色空间转换。**

由于 HDR 和 SDR 使用的颜色空间是不同的，HDR 通常使用 BT.2020，SDR 通常用 BT.709，所以要做一下颜色空间转换。

这个步骤对应的命令参数：`zscale=p=bt709`，表示转换的目标颜色空间是 bt709。


这里主要是根据颜色空间转换矩阵来做一下转换即可。我们还是可以从 [zimg](https://github.com/sekrit-twc/zimg "zimg") 源代码中找到关键函数：


延时空间的转换也会先调用 `get_neighboring_colorspaces` 函数。


```c++
// src/zimg/colorspace/graph.cpp

// 创建颜色空间转换函数
std::vector<ColorspaceNode> get_neighboring_colorspaces(const ColorspaceDefinition &csp)
{
  zassert_d(is_valid_csp(csp), "invalid colorspace");

  std::vector<ColorspaceNode> edges;

  auto add_edge = [&](const ColorspaceDefinition &out_csp, auto func)
  {
    edges.emplace_back(out_csp, std::bind(func, csp, out_csp, std::placeholders::_1, std::placeholders::_2));
  };

  if (csp.matrix == MatrixCoefficients::RGB) {
    constexpr MatrixCoefficients special_matrices[] = {
      MatrixCoefficients::UNSPECIFIED,
      MatrixCoefficients::RGB,
      MatrixCoefficients::REC_2020_CL,
      MatrixCoefficients::CHROMATICITY_DERIVED_NCL,
      MatrixCoefficients::CHROMATICITY_DERIVED_CL,
      MatrixCoefficients::REC_2100_LMS,
      MatrixCoefficients::REC_2100_ICTCP,
    };

    // RGB can be converted to conventional YUV.
    for (auto matrix : all_matrix()) {
      if (std::find(std::begin(special_matrices), std::end(special_matrices), matrix) == std::end(special_matrices))
        add_edge(csp.to(matrix), create_ncl_rgb_to_yuv_operation);
    }
    if (csp.primaries != ColorPrimaries::UNSPECIFIED)
      add_edge(csp.to(MatrixCoefficients::CHROMATICITY_DERIVED_NCL), create_ncl_rgb_to_yuv_operation);

    // Linear RGB can be converted to other transfer functions and primaries; also to combined matrix-transfer systems.
    if (csp.transfer == TransferCharacteristics::LINEAR) {
      for (auto transfer : all_transfer()) {
        if (transfer != csp.transfer && transfer != TransferCharacteristics::UNSPECIFIED) {
          add_edge(csp.to(transfer), create_linear_to_gamma_operation);
          if (csp.primaries != ColorPrimaries::UNSPECIFIED)
            add_edge(csp.to(transfer).to(MatrixCoefficients::CHROMATICITY_DERIVED_CL), create_cl_rgb_to_yuv_operation);
        }
      }
      if (csp.primaries != ColorPrimaries::UNSPECIFIED) {
        for (auto primaries : all_primaries()) {
          if (primaries != csp.primaries && primaries != ColorPrimaries::UNSPECIFIED)
            add_edge(csp.to(primaries), create_gamut_operation);
        }
      }

      add_edge(csp.to(MatrixCoefficients::REC_2020_CL).to(TransferCharacteristics::REC_709), create_cl_rgb_to_yuv_operation);

      if (csp.primaries == ColorPrimaries::REC_2020)
        add_edge(csp.to(MatrixCoefficients::REC_2100_LMS), create_ncl_rgb_to_yuv_operation);
    } else if (csp.transfer != TransferCharacteristics::UNSPECIFIED) {
      // Gamma RGB can be converted to linear RGB.
      add_edge(csp.to_linear(), create_gamma_to_linear_operation);
    }
  } else if (csp.matrix == MatrixCoefficients::REC_2020_CL || csp.matrix == MatrixCoefficients::CHROMATICITY_DERIVED_CL) {
    add_edge(csp.to_rgb().to_linear(), create_cl_yuv_to_rgb_operation);
  } else if (csp.matrix == MatrixCoefficients::REC_2100_LMS) {
    // LMS with ST_2084 or ARIB_B67 transfer functions can be converted to ICtCp and also to linear transfer function.
    if (csp.transfer == TransferCharacteristics::ST_2084 || csp.transfer == TransferCharacteristics::ARIB_B67) {
      add_edge(csp.to(MatrixCoefficients::REC_2100_ICTCP), create_lms_to_ictcp_operation);
      add_edge(csp.to(TransferCharacteristics::LINEAR), create_gamma_to_linear_operation);
    }
    // LMS with linear transfer function can be converted to RGB matrix and to ARIB_B67 and ST_2084 transfer functions.
    if (csp.transfer == TransferCharacteristics::LINEAR) {
      add_edge(csp.to_rgb(), create_ncl_yuv_to_rgb_operation);
      add_edge(csp.to(TransferCharacteristics::ST_2084), create_linear_to_gamma_operation);
      add_edge(csp.to(TransferCharacteristics::ARIB_B67), create_linear_to_gamma_operation);
    }
  } else if (csp.matrix == MatrixCoefficients::REC_2100_ICTCP) {
    // ICtCp with ST_2084 or ARIB_B67 transfer functions can be converted to LMS.
    if (csp.transfer == TransferCharacteristics::ST_2084 || csp.transfer == TransferCharacteristics::ARIB_B67)
      add_edge(csp.to(MatrixCoefficients::REC_2100_LMS), create_ictcp_to_lms_operation);
  } else if (csp.matrix != MatrixCoefficients::UNSPECIFIED) {
    // YUV can be converted to RGB.
    add_edge(csp.to_rgb(), create_ncl_yuv_to_rgb_operation);
  }

  return edges;
}
```

在 `get_neighboring_colorspaces` 函数中会根据 ColorPrimaries 的差异来将 `ZIMG_PRIMARIES_BT2020` 转换为 `ZIMG_PRIMARIES_BT709`，这个过程会调用 `create_gamut_operation` 进行 rgb 与 xyz 对应颜色空间的转换。


```c++
// src/zimg/colorspace/operation.cpp

// 创建 rgb 与 xyz 颜色空间转换矩阵
std::unique_ptr<Operation> create_gamut_operation(const ColorspaceDefinition &in, const ColorspaceDefinition &out, const OperationParams &params, CPUClass cpu)
{
  zassert_d(in.matrix == MatrixCoefficients::RGB && in.transfer == TransferCharacteristics::LINEAR, "must be linear RGB");
  zassert_d(out.matrix == MatrixCoefficients::RGB && out.transfer == TransferCharacteristics::LINEAR, "must be linear RGB");

  Matrix3x3 m = gamut_xyz_to_rgb_matrix(out.primaries) * white_point_adaptation_matrix(in.primaries, out.primaries) * gamut_rgb_to_xyz_matrix(in.primaries);
  return create_matrix_operation(m, cpu);
}
```

这里的实现是用 CIE XYZ 颜色空间作为中转，先把原颜色空间转为 XYZ 颜色空间，再把 XYZ 颜色空间转为目标颜色空间，整个步骤可以理解为像素直接和两个转换矩阵相乘。其中对应的 BT.2020 RGB 转 XYZ 颜色空间、XYZ 转 BT.709 颜色空间的矩阵如下：

```
// BT2020
RGB2XYZ Matrix:
0.6370, 0.1446, 0.1689
0.2627, 0.6780, 0.0593
0.0000, 0.0281, 1.0610

// BT709
XYZ2RGB Matrix:
3.2410, -1.5374, -0.4986
-0.9692, 1.8760, 0.0416
0.0556, -0.2040, 1.0570
```

总结一下上述过程大致如下：

```
get_neighboring_colorspaces
    -> create_gamut_operation
        -> gamut_xyz_to_rgb_matrix && gamut_rgb_to_xyz_matrix
```





**3）HDR 的线性模拟光信号 ToneMapping 转换到 SDR 的线性模拟光信号。**

通过上一步获得的 EOTF 转换函数，完成颜色数字信号转换为线性的模拟光信号后，接下来我们要做的是将 HDR 的线性模拟光信号 ToneMapping 转换到 SDR 的线性模拟光信号。

这个步骤对应的命令参数：`tonemap=tonemap=hable:desat=0`，表示 tonemap 的算法用 hable，减饱和强度（desaturation strength）为 0。


这里用到的核心代码是 [FFmpeg](https://github.com/FFmpeg/FFmpeg "FFmpeg") 的视频滤镜模块中的 [ffmpeg/libavfilter/vf_tonemap.c](https://github.com/FFmpeg/FFmpeg/blob/master/libavfilter/vf_tonemap.c "vf_tonemap.c")。具体代码如下：


```c
// ffmpeg/libavfilter/vf_tonemap.c

#define MIX(x,y,a) (x) * (1 - (a)) + (y) * (a)
static void tonemap(TonemapContext *s, AVFrame *out, const AVFrame *in,
                    const AVPixFmtDescriptor *desc, int x, int y, double peak)
{
    int map[3] = { desc->comp[0].plane, desc->comp[1].plane, desc->comp[2].plane };
    const float *r_in = (const float *)(in->data[map[0]] + x * desc->comp[map[0]].step + y * in->linesize[map[0]]);
    const float *g_in = (const float *)(in->data[map[1]] + x * desc->comp[map[1]].step + y * in->linesize[map[1]]);
    const float *b_in = (const float *)(in->data[map[2]] + x * desc->comp[map[2]].step + y * in->linesize[map[2]]);
    float *r_out = (float *)(out->data[map[0]] + x * desc->comp[map[0]].step + y * out->linesize[map[0]]);
    float *g_out = (float *)(out->data[map[1]] + x * desc->comp[map[1]].step + y * out->linesize[map[1]]);
    float *b_out = (float *)(out->data[map[2]] + x * desc->comp[map[2]].step + y * out->linesize[map[2]]);
    float sig, sig_orig;
 
    /* load values */
    *r_out = *r_in;
    *g_out = *g_in;
    *b_out = *b_in;
 
    /* desaturate to prevent unnatural colors */
    if (s->desat > 0) {
        float luma = s->coeffs->cr * *r_in + s->coeffs->cg * *g_in + s->coeffs->cb * *b_in;
        float overbright = FFMAX(luma - s->desat, 1e-6) / FFMAX(luma, 1e-6);
        *r_out = MIX(*r_in, luma, overbright);
        *g_out = MIX(*g_in, luma, overbright);
        *b_out = MIX(*b_in, luma, overbright);
    }
 
    /* pick the brightest component, reducing the value range as necessary
     * to keep the entire signal in range and preventing discoloration due to
     * out-of-bounds clipping */
    sig = FFMAX(FFMAX3(*r_out, *g_out, *b_out), 1e-6);
    sig_orig = sig;
 
    switch(s->tonemap) {
    default:
    case TONEMAP_NONE:
        // do nothing
        break;
    case TONEMAP_LINEAR:
        sig = sig * s->param / peak;
        break;
    case TONEMAP_GAMMA:
        sig = sig > 0.05f ? pow(sig / peak, 1.0f / s->param)
                          : sig * pow(0.05f / peak, 1.0f / s->param) / 0.05f;
        break;
    case TONEMAP_CLIP:
        sig = av_clipf(sig * s->param, 0, 1.0f);
        break;
    case TONEMAP_HABLE:
        sig = hable(sig) / hable(peak);
        break;
    case TONEMAP_REINHARD:
        sig = sig / (sig + s->param) * (peak + s->param) / peak;
        break;
    case TONEMAP_MOBIUS:
        sig = mobius(sig, s->param, peak);
        break;
    }
 
    /* apply the computed scale factor to the color,
     * linearly to prevent discoloration */
    *r_out *= sig / sig_orig;
    *g_out *= sig / sig_orig;
    *b_out *= sig / sig_orig;
}
```

这里使用的 tonemap 算法是 hable，代码如下：


```c
// ffmpeg/libavfilter/vf_tonemap.c

static float hable(float in)
{
    float a = 0.15f, b = 0.50f, c = 0.10f, d = 0.20f, e = 0.02f, f = 0.30f;
    return (in * (in * a + b * c) + d * e) / (in * (in * a + b) + d * f) - e / f;
}

static float mobius(float in, float j, double peak)
{
    float a, b;

    if (in <= j)
        return in;

    a = -j * j * (peak - 1.0f) / (j * j - 2.0f * j + peak);
    b = (j * j - 2.0f * j * peak + peak) / FFMAX(peak - 1.0f, 1e-6);

    return (b * b + 2.0f * b * j + j * j) / (b - a) * (in + a) / (in + b);
}
```

tonemap 本质上是一个编码压缩曲线，可以简单的理解其目的是为了把 `[0, 1024]` 的空间范围如何较好的压缩映射到 `[0, 255]` 的空间范围。hable 是 tonemap 的一种算法，其他算法还有上面贴出来的 reinhard、mobius。更多的算法可以参考：[Tone mapping 进化论 ](https://zhuanlan.zhihu.com/p/21983679 "Tone mapping 进化论")。




**4）线性的模拟光信号经过 OETF 转换为非线性颜色数字信号。**

做完 ToneMapping 后，我们就得到了符合 SDR 数据范围的线性模拟光信号。接下来我们再将其转换为颜色数字信号。不过，因为是 SDR，这里要使用的 OETF 是 BT.709。

这个步骤对应的命令参数：`zscale=t=bt709:m=bt709:r=tv`，表示使用的 OETF 转换函数为 bt709，转换矩阵也是 bt709，YUV 的 range 为 tv.limited。

代码调用流程和第一步类似，这里只贴一下流程：


```
get_neighboring_colorspaces
    -> reate_linear_to_gamma_operation
        -> create_gamma_operation
            -> select_transfer_function
                case TransferCharacteristics::REC_709:
                func.to_gamma = scene_referred ? rec_709_oetf : rec_1886_inverse_eotf;
                break;  
```

最后对应的转换函数为 `rec_709_oetf`，代码如下：


```c++
// src/zimg/colorspace/gamma.cpp

constexpr float REC709_ALPHA = 1.09929682680944f;
constexpr float REC709_BETA = 0.018053968510807f;
float rec_709_oetf(float x) noexcept
{
  x = std::max(x, 0.0f);
 
  if (x < REC709_BETA)
    x = x * 4.5f;
  else
    x = REC709_ALPHA * zimg_x_powf(x, 0.45f) - (REC709_ALPHA - 1.0f);
 
  return x;
}
```




参考：

- [仿照 FFmpeg 在 GLSL 中处理 HDR.ToneMapping](https://blog.csdn.net/a360940265a/article/details/124457544 "仿照 FFmpeg 在 GLSL 中处理 HDR.ToneMapping")
- [HDR in Android](https://blog.csdn.net/a360940265a/category_11625435.html "HDR in Android")
- [HDR 片源压制成 BT.709 色彩空间的 SDR 视频](https://www.bilibili.com/read/cv3936575 "HDR 片源压制成 BT.709 色彩空间的 SDR 视频")





### 3.2、使用 FFmpeg 软解 + OpenGL 实现转换

上面讲了使用 FFmpeg filter 完成转换的方案，这里有两个问题：一个是 FFmpeg 软解性能的问题，另外一个是使用 CPU 做 filter 性能的问题。我们这里先解决一下用 CPU 做 filter 的性能问题：将 EOTF、颜色空间转换、ToneMapping、OETF 使用 OpenGL 实现，从而将这些操作用 GPU 来完成。同时这里我们也可以同时支持 PQ 和 HLG 标准。


这里需要注意的是，FFmpeg 软解实现中解码出来的数据格式一般为 `AV_PIX_FMT_YUV420P10LE`，小端序，低 10 位有效，高 6 位均为 0，所以可以直接被 OpenGL 读取，不需要做移位操作。但是，要把这样的 YUV 10bit 的数据转为 Texture 纹理则需要做一下兼容处理，使用 16bit 的数据结构来存储 YUV 10bit。

![YUV 10bit](assets/resource/av-experience/transcode-hdr-to-sdr-9.png)
_YUV 10bit_


下面是将 EOTF、颜色空间转换、ToneMapping、OETF 流程用 OpenGL ES Fragment Shader 实现的代码：


```c
precision highp float;
uniform sampler2D inputImageTexture;
uniform mediump mat3 colorConversionMatrix;
uniform mediump int isSt2084;
uniform mediump int isAribB67;
varying highp vec2 textureCoordinate;

#define FFMAX(a,b) ((a) > (b) ? (a) : (b))
#define FFMAX3(a,b,c) FFMAX(FFMAX(a,b),c)

highp vec3 YuvConvertRGB_BT2020(highp vec3 yuv, int normalize) {
    highp vec3 rgb;
    // [64, 960]
    float r = float(yuv.x - 64.) * 1.164384                                  - float(yuv.z - 512.) * -1.67867;
    float g = float(yuv.x - 64.) * 1.164384 - float(yuv.y - 512.) * 0.187326 - float(yuv.z - 512.) * 0.65042;
    float b = float(yuv.x - 64.) * 1.164384 - float(yuv.y - 512.) * -2.14177;
    rgb.r = r;
    rgb.g = g;
    rgb.b = b;
    if (normalize == 1) { 
        rgb /= 1024.0; 
    }
    return rgb;
}

// [arib b67 eotf
const highp float ARIB_B67_A = 0.17883277;
const highp float ARIB_B67_B = 0.28466892;
const highp float ARIB_B67_C = 0.55991073;
highp float arib_b67_inverse_oetf(highp float x)
{
    // Prevent negative pixels expanding into positive values.
    x = max(x, 0.0);
    if (x <= 0.5)
    x = (x * x) * (1.0 / 3.0);
    else
    x = (exp((x - ARIB_B67_C) / ARIB_B67_A) + ARIB_B67_B) / 12.0;
    return x;
}
highp float ootf_1_2(highp float x)
{
    return x < 0.0 ? x : pow(x, 1.2);
}
highp float arib_b67_eotf(highp float x)
{
    return ootf_1_2(arib_b67_inverse_oetf(x));
}
// arib b67 eotf]


// [st 2084 eotf
highp float ST2084_M1 = 0.1593017578125;
const float ST2084_M2 = 78.84375;
const float ST2084_C1 = 0.8359375;
const float ST2084_C2 = 18.8515625;
const float ST2084_C3 = 18.6875;
highp float FLT_MIN = 1.17549435082228750797e-38;
highp float st_2084_eotf(highp float x)
{
    highp float xpow = pow(x, float(1.0 / ST2084_M2));
    highp float num = max(xpow - ST2084_C1, 0.0);
    highp float den = max(ST2084_C2 - ST2084_C3 * xpow, FLT_MIN);
    return pow(num/den, 1.0 / ST2084_M1);
}
// st 2084 eotf]

// [tonemap hable
highp float hableF(highp float inVal)
{
    highp float a = 0.15, b = 0.50, c = 0.10, d = 0.20, e = 0.02, f = 0.30;
    return (inVal * (inVal * a + b * c) + d * e) / (inVal * (inVal * a + b) + d * f) - e / f;
}
// tonemap hable]

// [bt709 
highp float rec_1886_inverse_eotf(highp float x)
{
    return x < 0.0 ? 0.0 : pow(x, 1.0 / 2.4);
}

highp float rec_1886_eotf(float x)
{
    return x < 0.0 ? 0.0 : pow(x, 2.4);
}
// bt709]

void main() {
    highp vec3 rgb10bit = texture2D(inputImageTexture, textureCoordinate).rgb;

    // 1、HDR 非线性电信号转为 HDR 线性光信号（EOTF）
    float peak_luminance = 100.0;
    float ST2084_PEAK_LUMINANCE = 10000.0;
    float to_linear_scale;
    highp vec3 fragColor;
    if (isSt2084 == 1) {
        to_linear_scale = 10000.0 / peak_luminance;
        fragColor = to_linear_scale * vec3(st_2084_eotf(rgb10bit.r), st_2084_eotf(rgb10bit.g), st_2084_eotf(rgb10bit.b));
    } else if (isAribB67 == 1) {
        to_linear_scale = 1000.0 / peak_luminance;
        fragColor = to_linear_scale * vec3(arib_b67_eotf(rgb10bit.r), arib_b67_eotf(rgb10bit.g), arib_b67_eotf(rgb10bit.b));
    } else {
        fragColor = vec3(rec_1886_eotf(rgb10bit.r), rec_1886_eotf(rgb10bit.g), rec_1886_eotf(rgb10bit.b));
    }

    // 2、HDR 线性光信号做颜色空间转换（Color Space Converting）
    // color-primaries REC_2020 to REC_709
    mat3 rgb2xyz2020 = mat3(0.6370, 0.1446, 0.1689,
                            0.2627, 0.6780, 0.0593,
                            0.0000, 0.0281, 1.0610);
    mat3 xyz2rgb709 = mat3(3.2410, -1.5374, -0.4986,
                           -0.9692, 1.8760, 0.0416,
                           0.0556, -0.2040, 1.0570);                                   
    fragColor *= rgb2xyz2020 * xyz2rgb709;

    // 3、HDR 线性光信号色调映射为 SDR 线性光信号（Tone Mapping）
    highp float sig = FFMAX(FFMAX3(fragColor.r, fragColor.g, fragColor.b), 1e-6);
    highp float sig_orig = sig;
    float peak = 10.0;
    sig = hableF(sig) / hableF(peak);
    fragColor *= sig / sig_orig;

    // 4、SDR 线性光信号转 SDR 非线性电信号（OETF）
    fragColor = vec3(rec_1886_inverse_eotf(fragColor.r), rec_1886_inverse_eotf(fragColor.g), rec_1886_inverse_eotf(fragColor.b));
    gl_FragColor = vec4(fragColor, 1.0);
}
```

到这里，我们就把本来使用 FFmpeg `zscale` 和 `tonemap` 两个 Filter 的逻辑就迁移为 OpenGL 实现了。


### 3.3、使用 Android 硬解 + OpenGL 实现转换

接下来，我们接着解决 FFmpeg 软解的性能问题，在硬解支持较好的机型上使用硬解来实现。Android 硬解可以将视频解码为 Texture 纹理，所以相对于软解只要实现从纹理中读出 YUV10bit 数据，然后完成后续的：EOTF、颜色空间转换、ToneMapping、OETF 过程，就可以实现 HDR 转 SDR 了。

实现这个过程的 OpenGL ES Fragment Shader 代码和上一节中一样即可，这里就不再重复了。






