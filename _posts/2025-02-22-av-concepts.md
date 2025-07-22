---
title: 音视频基础概念合集：148 个问题让你快速了解音视频的基础概念
description: 通过问答的方式介绍音视频编解码、封装格式、传输协议相关的基础概念。
author: Keyframe
date: 2025-02-22 08:28:08 +0800
categories: [音视频基础知识]
tags: [音视频基础知识, 音视频, 编解码, 封装格式, 传输协议]
pin: true
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


音视频是一个有一定技术门槛的垂直领域，对于前端、iOS/Android 客户端、服务端开发同学来说，这都是一个不错的职业发展方向。对于刚开始接触音视频开发的同学，最头疼的问题应该是音视频纷繁庞杂的概念，如果删繁就简，音视频生产及消费的核心环节其实只有：**采集（声音和图像的数字化） → 编码（压缩数据便于存储和传输） → 封装（按格式封装便于控制音视频的展现） → 传输（用于网络） → 解封装（封装的逆过程） → 解码（编码的逆过程） → 渲染（声音和图像的展现）**。

因此，在前面的文章中，我们按照『**声音和图像基础 → 音视频编码 → 音视频格式 → 音视频协议**』路线图介绍了很多基础的音视频知识。这里面内容较多，其中大部分细节在日常开发中不见得会立即用到，所以暂时不深入了解也问题不大。但是，对很多音视频基础的概念最好还是留个印象，这样在遇到的时候就能快速知道大概的意思，理清思路。如果还想进一步了解其中的细节，则可以回过头来深入查看我们前面发的文章。

这篇文章就是为这些基础概念建一个索引，方便记忆和查阅。如果你是对音视频方向感兴趣的开发者，强烈建议**点赞**、**收藏**、**分享**。


## 1、声音和图像基础


1. `为什么要声音和图像进行数字化？`
	- 将物理世界的模拟信号转换为数字世界的数字信号，用于后续的数字处理。
1. `声音三要素是什么？`
	- 响度、音调、音色。
	- 参见：[《声音的表示（1）》第 2 节](https://mp.weixin.qq.com/s/b4XkNaZWnSLx8KxqV0Ul1A)
1. `波形图是什么？`
	- 横坐标是时间，纵坐标是振幅，表示所有频率叠加的正弦波振幅的总大小随时间的变化规律。
	- 参见：[《声音的表示（1）》第 2 节](https://mp.weixin.qq.com/s/b4XkNaZWnSLx8KxqV0Ul1A)
1. `频谱图是什么？`
	- 横坐标为频率，纵坐标为幅值，表示一个静态的时间点上各频率正弦波的幅值大小的分布状况。
	- 参见：[《声音的表示（1）》第 2 节](https://mp.weixin.qq.com/s/b4XkNaZWnSLx8KxqV0Ul1A)
1. `反应声音大小的物理量有哪些？`
	- 声能、声强、声压、声强级、声压级、响度级。
	- 参见：[《声音的表示（2）》第 3.1 节](https://mp.weixin.qq.com/s/LOUnbvYSNBONE9TgCxjLvw)
1. `日常所说的声音大小的分贝是指哪个物理量？`
	- 声压级。
	- 参见：[《声音的表示（2）》第 3.1 节](https://mp.weixin.qq.com/s/LOUnbvYSNBONE9TgCxjLvw)
1. `音调和频率是什么关系？`
	- 频率是音调的客观评价尺度，单位是赫兹(Hz)。
	- 参见：[《声音的表示（2）》第 3.2 节](https://mp.weixin.qq.com/s/LOUnbvYSNBONE9TgCxjLvw)
1. `音调的计量单位是什么？`
	- 美(mel)。
	- 参见：[《声音的表示（2）》第 3.2 节](https://mp.weixin.qq.com/s/LOUnbvYSNBONE9TgCxjLvw)
1. `乐音的音调由什么决定？`
	- 基音的频率。
	- 参见：[《声音的表示（2）》第 3.2 节](https://mp.weixin.qq.com/s/LOUnbvYSNBONE9TgCxjLvw)
1. `科学音调记号法是什么？`
	- 一种利用字母及一个用来表示所在八度的阿拉伯数字来明确指出音符的位置的记号法。
	- 参见：[《声音的表示（2）》第 3.2 节](https://mp.weixin.qq.com/s/LOUnbvYSNBONE9TgCxjLvw)
1. `十二平均律是什么？`
	- 一种将一个八度平均分成十二等份的音乐律式。
	- 参见：[《声音的表示（2）》第 3.2 节](https://mp.weixin.qq.com/s/LOUnbvYSNBONE9TgCxjLvw)
1. `声音的复杂波形可以分解成什么？`
	- 基频和谐波。
	- 参见：[《声音的表示（2）》第 3.3 节](https://mp.weixin.qq.com/s/LOUnbvYSNBONE9TgCxjLvw)
1. `音色由什么决定？`
	- 谐波频谱。
	- 参见：[《声音的表示（2）》第 3.3 节](https://mp.weixin.qq.com/s/LOUnbvYSNBONE9TgCxjLvw)
1. `音频数字化包括哪些步骤？`
	- 采样、量化、编码。
	- 参见：[《声音的表示（3）》第 4 节](https://mp.weixin.qq.com/s/lexAVx_O3Kz3-51OZ3TlLw)
1. `数字音频三要素是什么？`
	- 采样率、量化位深、声道数。
	- 参见：[《声音的表示（3）》第 4 节](https://mp.weixin.qq.com/s/lexAVx_O3Kz3-51OZ3TlLw)
1. `奈奎斯特采样定理是什么？`
	- 为了不失真地恢复模拟信号，采样频率应该不小于模拟信号频谱中最高频率的 2 倍。
	- 参见：[《声音的表示（3）》第 4 节](https://mp.weixin.qq.com/s/lexAVx_O3Kz3-51OZ3TlLw)
1. `44100 Hz 这个奇葩数字从何而来？`
	- 最早的数字录音由一台 PAL 制式的录像机加上一部 PCM 编码器制作的，场频 50 Hz，可用扫描线数 294 条，一条视频扫描线的磁迹中记录 3 个音频数据块，把它们相乘，就得到了 44100。
	- 参见：[《声音的表示（3）》第 4 节](https://mp.weixin.qq.com/s/lexAVx_O3Kz3-51OZ3TlLw)
1. `常见的声道类型有哪些？`
	- 单声道（Mono）、立体声（Stereo）、5.1 声道、7.1 声道。
	- 参见：[《声音的表示（3）》第 4 节](https://mp.weixin.qq.com/s/lexAVx_O3Kz3-51OZ3TlLw)
1. `最常见的数字音频数据是什么？`
	- PCM 数据。
	- 参见：[《声音的表示（3）》第 5 节](https://mp.weixin.qq.com/s/lexAVx_O3Kz3-51OZ3TlLw)
1. `人眼是如何感知到颜色的？`
	- 人眼视觉感知三原色理论。
	- 参见：[《图像的表示（1）》第 2.2 节](https://mp.weixin.qq.com/s/fOiV32SZb7UcN6KLUCI61g)
1. `人眼对颜色的感受可以分为哪些特征？`
	- 色调（hue）、亮度（brightness）、饱和度（saturation）。还有一个色度（chromaticity）通常是说明『饱和度』和『色调』这两种特征的综合表现。
	- 参见：[《图像的表示（1）》第 2.2 节](https://mp.weixin.qq.com/s/fOiV32SZb7UcN6KLUCI61g)
1. `日常音视频开发处理的视频数据对应的颜色模型是什么？`
	- ITU-R YCbCr。
	- 参见：[《图像的表示（2）》开篇简介](https://mp.weixin.qq.com/s/7QMpCe3YBgKxtHM260SZYA)
1. `YCbCr 颜色模型是如何发展而来？`
	- 大致路径：CIE RGB → CIE XYZ → NTSC YIQ → PAL YUV → ITU-R YCbCr。
	- 参见：[《图像的表示（2）》开篇简介](https://mp.weixin.qq.com/s/7QMpCe3YBgKxtHM260SZYA)
1. `什么是 CIE RGB 颜色模型？`
	- 基于人眼视觉感知三原色理论，CIE 通过大量实验数据建立了 RGB 颜色模型，标准化了 RGB 表示。
	- 参见：[《图像的表示（2）》开篇简介](https://mp.weixin.qq.com/s/7QMpCe3YBgKxtHM260SZYA)
1. `什么是 CIE XYZ 颜色模型？`
	- 为了解决 RGB 模型中与负光混合所带来的种种问题，CIE 从数学上定义了三种标准基色 XYZ，形成了 CIE XYZ 颜色模型。
	- 参见：[《图像的表示（2）》开篇简介](https://mp.weixin.qq.com/s/7QMpCe3YBgKxtHM260SZYA)
1. `什么是 NTSC YIQ 颜色模型？`
	- 在模拟电视时代，RGB 工业显示器要求一幅彩色图像由分开的 R、G、B 信号组成，而电视显示器则需要混合信号输入，为了实现对这两种标准的兼容，NTSC 基于 XYZ 模型制定了 YIQ 颜色模型，实现了彩色电视和黑白电视的信号兼容。
	- 参见：[《图像的表示（2）》开篇简介](https://mp.weixin.qq.com/s/7QMpCe3YBgKxtHM260SZYA)
1. `什么是 PAL YUV 颜色模型？`
	- 为了解决 NTSC YIQ 的组合模拟视频信号中分配给色度信息的带宽较低而影响了图像颜色质量的问题，PAL 引入了 YUV 颜色模型，支持用不同的采样格式来调整传输的色度信息量。
	- 参见：[《图像的表示（2）》开篇简介](https://mp.weixin.qq.com/s/7QMpCe3YBgKxtHM260SZYA)
1. `什么是 ITU-R YCbCr 颜色模型？`
	- 进入数字电视时代，ITU-R 为数字视频转换制定了 YCbCr 颜色模型，成为我们现在最常使用的颜色模型。
	- 参见：[《图像的表示（2）》开篇简介](https://mp.weixin.qq.com/s/7QMpCe3YBgKxtHM260SZYA)
1. `什么是加色模式？`
	- 当光由两个或多个具有不同主频率的光源混合而成时，改变各个光源的强度来构造一系列其他颜色的光的方法。如：RGB。
	- 参见：[《图像的表示（2）》第 3.2 节](https://mp.weixin.qq.com/s/7QMpCe3YBgKxtHM260SZYA)
1. `什么是减色模式？`
	- 通过从光源中减去某些颜色后得到其他一些列的颜色的方法。如：CMYK。
	- 参见：[《图像的表示（2）》第 3.2 节](https://mp.weixin.qq.com/s/7QMpCe3YBgKxtHM260SZYA)
1. `什么是色域、基色、互补色？`
	- 通过基色可以产生的所有颜色的集合称为该颜色模型的色域（颜色范围）；
	- 其中用来生成其他颜色光源的色彩称为基色；
	- 如果两种基色混合则生成白色光，就称它们为互补色。如：红色和青色、绿色和品红、蓝色和黄色都是互补色。
	- 参见：[《图像的表示（2）》第 3.2 节](https://mp.weixin.qq.com/s/7QMpCe3YBgKxtHM260SZYA)
1. `什么是色度图？`
	- 忽略颜色的亮度（brightness）特征，只关注色度（chromaticity）时，使用二维的颜色坐标系来表示颜色模型平面图示法。
	- 参见：[《图像的表示（2）》第 3.2 节](https://mp.weixin.qq.com/s/7QMpCe3YBgKxtHM260SZYA)
1. `色度图的基准是什么？`
	- CIE 1931 年的 x-y 色度图。
	- 参见：[《图像的表示（2）》第 3.3 节](https://mp.weixin.qq.com/s/7QMpCe3YBgKxtHM260SZYA)
1. `色度图的作用是什么？`
	- 为不同基色组比较整个颜色范围、标识互补色、确定指定颜色的主波长、确定指定颜色的饱和度。
	- 参见：[《图像的表示（2）》第 3.3 节](https://mp.weixin.qq.com/s/7QMpCe3YBgKxtHM260SZYA)
1. `三基色的色域如何确定？`
	- 三基色组成的三角形在色度图对应的区域。
	- 参见：[《图像的表示（2）》第 3.3 节](https://mp.weixin.qq.com/s/7QMpCe3YBgKxtHM260SZYA)
1. `ITU-R BT.601、BT.709、BT.2020 是什么？`
	- ITU-R（国际电信联盟无线电通信部门）制定的彩色视频转换成数字图像时使用的系列标准，分别面向标清电视（SDTV）、高清电视（HDTV）、超清电视（UDTV）应用场景。
	- 参见：[《图像的表示（2）》第 3.6 节](https://mp.weixin.qq.com/s/7QMpCe3YBgKxtHM260SZYA)
1. `YCbCr 有哪些采样格式？`
	- 4:4:4、4:2:2、4:1:1、4:2:0。
	- 参见：[《图像的表示（2）》第 3.6 节](https://mp.weixin.qq.com/s/7QMpCe3YBgKxtHM260SZYA)
1. `YCbCr 有哪些存储格式？`
	- 平面格式（planar），如：I420、YV12；
	- 紧缩格式（packed）；
	- 混合模式，如：NV12、NV21。
	- 参见：[《图像的表示（2）》第 3.6 节](https://mp.weixin.qq.com/s/7QMpCe3YBgKxtHM260SZYA)
1. `什么是显示伽马？`
	- 典型的 CRT 显示器产生的亮度约为输入电压的 2.2 次幂的效应。
	- 参见：[《图像的表示（2）》第 3.7 节](https://mp.weixin.qq.com/s/7QMpCe3YBgKxtHM260SZYA)
1. `什么是伽马校正？`
	- 由于显示伽马的存在，为使最终显示出来的图像亮度与真实场景亮度成线性比例关系，而在将图像输入到显示器之前进行的校正过程。
	- 参见：[《图像的表示（2）》第 3.7 节](https://mp.weixin.qq.com/s/7QMpCe3YBgKxtHM260SZYA)
1. `什么是编码伽马？`
	- 伽马校正过程中引入的伽马处理。
	- 参见：[《图像的表示（2）》第 3.7 节](https://mp.weixin.qq.com/s/7QMpCe3YBgKxtHM260SZYA)
1. `图像数字化包括哪些步骤？`
	- 采样、量化。
	- 参见：[《图像的表示（3）》第 4 节](https://mp.weixin.qq.com/s/RhMLfOoDdf6TvRoGVQaNHA)
1. `数字图像包括哪些基本属性？`
	- 图像分辨率、像素深度。
	- 参见：[《图像的表示（3）》第 4 节](https://mp.weixin.qq.com/s/RhMLfOoDdf6TvRoGVQaNHA)
1. `最常见的数字图像数据是什么？`
	- RGB 数据和 YCbCr 数据。
	- 参见：[《图像的表示（3）》第 5 节](https://mp.weixin.qq.com/s/RhMLfOoDdf6TvRoGVQaNHA)





## 2、音视频编码


1. `为什么要对音视频数据进行编码？`
	- 为了进行数据压缩，以此来降低数据传输和存储的成本。
	- 参见：[《音频编码》开篇简介](https://mp.weixin.qq.com/s/tVUgH17HZe6PWj09hYzkbg)
1. `对信息进行压缩的理论基础是什么？`
	- 信源包含的符号出现概率的非均匀性、信源的相关性、人的感知对不同信源的敏感度不一样，使得信源是可以被压缩的。
	- 参见：[《音频编码》开篇简介](https://mp.weixin.qq.com/s/tVUgH17HZe6PWj09hYzkbg)
1. `对音频数据进行压缩的理论基础是什么？`
	- 时域冗余：幅度分布的非均匀性、样值之间的相关性、信号周期之间的相关性、静止系数、长时自相关性；
	- 频域冗余：长时功率谱密度的非均匀性、语音特有的短时功率谱密度；
	- 听觉冗余：最小可闻阈值、频率掩蔽效应、时域掩蔽效应等。
	- 参见：[《音频编码》开篇简介](https://mp.weixin.qq.com/s/tVUgH17HZe6PWj09hYzkbg)
1. `什么是频率掩蔽效应？`
	- 当有能量较大的声音出现的时候，该声音频率附近的其它频率的人耳听不到。
	- 参见：[《音频编码》开篇简介](https://mp.weixin.qq.com/s/tVUgH17HZe6PWj09hYzkbg)
1. `什么是时域掩蔽效应？`
	- 当强音信号和弱音信号同时出现时，可能会发生前掩蔽、同时掩蔽、后掩蔽，被掩蔽的信号人耳听不到。
	- 参见：[《音频编码》开篇简介](https://mp.weixin.qq.com/s/tVUgH17HZe6PWj09hYzkbg)
1. `常见的音频编码格式有哪些？`
	- PCM、WAV、MP3、AAC、OPUS 等。
	- 参见：[《音频编码》开篇简介](https://mp.weixin.qq.com/s/tVUgH17HZe6PWj09hYzkbg)
1. `MPEG-2 AAC 编码工具有哪些？`
	- 码流解析模块、无噪编解码模块、量化和反量化模块、缩放因子处理模块、Mid/Side 立体声编解码模块、预测模块、强度立体声编解码模块、非独立交换耦合模块、瞬时噪音整形模块（TNS）、滤波器组/块切换模块、增益控制模块、独立交换耦合模块。
	- 参见：[《音频编码》第 2.2 节](https://mp.weixin.qq.com/s/tVUgH17HZe6PWj09hYzkbg)
1. `MPEG－4 AAC 相对 MPEG－2 AAC 增加了哪些编码工具？`
	- 长时预测模块（LTP）、知觉噪声替换模块（PNS）、频段复制技术（SBR）、参数立体声技术（PS）。
	- 参见：[《音频编码》第 2.2 节](https://mp.weixin.qq.com/s/tVUgH17HZe6PWj09hYzkbg)
1. `MPEG-2 AAC 有哪些编码规格？`
	- LC，低复杂度规格；Main，主规格；SSR，可变采样率规格。
	- 参见：[《音频编码》第 2.3 节](https://mp.weixin.qq.com/s/tVUgH17HZe6PWj09hYzkbg)
1. `MPEG－4 AAC 相对 MPEG－2 AAC 增加了哪些编码规格？`
	- LD，低延迟规格；LTP，长时预测规格；HE，高效率规格。
	- 参见：[《音频编码》第 2.3 节](https://mp.weixin.qq.com/s/tVUgH17HZe6PWj09hYzkbg)
1. `HE AAC、HE AAC v2 是指什么？`
	- HE AAC 是 AAC + SBR 编码技术。
	- HE AAC v2 是 AAC + SBR + PS 编码技术。
	- 参见：[《音频编码》第 2.3 节](https://mp.weixin.qq.com/s/tVUgH17HZe6PWj09hYzkbg)
1. `ASC 是什么？`
	- AudioSpecificConfig，存储音频编码相关的基础信息，比如采样率、位深、声道。
	- 参见：[《音频编码》第 2.4.2 节](https://mp.weixin.qq.com/s/tVUgH17HZe6PWj09hYzkbg)
1. `MPEG-2 AAC 的音频编码数据格式哪些？`
	- ADIF，音频数据交换格式；ADTS，音频数据传输流格式。
	- 参见：[《音频编码》第 2.4.3 节](https://mp.weixin.qq.com/s/tVUgH17HZe6PWj09hYzkbg)
1. `MPEG－4 AAC 相对 MPEG－2 AAC 增加了哪些编码数据格式？`
	- LATM 和 LOAS。
	- 参见：[《音频编码》第 2.4.3 节](https://mp.weixin.qq.com/s/tVUgH17HZe6PWj09hYzkbg)
1. `对视频数据进行压缩的理论基础是什么？`
	- 相邻像素之间的空间冗余、相邻帧之间的时间冗余、编码冗余、视觉冗余。
	- 参见：[《视频编码（1）》开篇简介](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)
1. `常见的视频编码格式有哪些？`
	- ISO-MPEG/ITU-T 系列：H.264（AVC）、H.265（HEVC）、H.266（VVC）；
	- AOM 系列：VP8、VP9、AV1；
	- AVS 系列：AVS2、AVS3。
	- 参见：[《视频编码（1）》开篇简介](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)
1. `H.264 的句法元素有哪些？`
	- 序列、图像、片、宏块、子宏块。
	- 参见：[《视频编码（1）》第 1.1.1 节](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)
1. `I 帧、B 帧与 P 帧有什么区别？`
	- I 帧，即帧内编码帧，不参考其他图像帧，只利用本帧的信息进行编码。
	- P 帧，即预测编码帧，参考之前的 I 帧或 P 帧，采用运动预测的方式进行帧间预测编码。
	- B 帧，即双向预测编码帧，提供最高的压缩比，它既参考之前的图像帧（I 帧或 P 帧）和后来的图像帧（P 帧），采用运动预测的方式进行帧间双向预测编码。
	- 参见：[《视频编码（1）》第 1.1.4 节](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)
1. `什么是率失真？`
	- 在允许一定程度失真的条件下，能够把信源信息压缩到什么程度。即最少需要多少比特数才能描述信源。
	- 参见：[《视频编码（1）》第 1.1.4 节](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)
1. `视频编码怎么进行率失真优化？`
	- 遍历所有的参数候选模式对视频进行编码，满足码率限制的失真最小的一组参数集作为最优的视频编码参数。
	- 参见：[《视频编码（1）》第 1.1.4 节](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)
1. `什么是 DTS 和 PTS？`
	- DTS 是解码时间戳；PTS 是显示时间戳。
	- 参见：[《视频编码（1）》第 1.1.5 节](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)
1. `什么是 GOP？`
	- GOP 是视频编码序列中两个 I 帧之间的距离。
	- 参见：[《视频编码（1）》第 1.1.6 节](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)
1. `什么是 IDR 帧？`
	- IDR 帧全称叫做 Instantaneous Decoder Refresh，是 I 帧的一种。IDR 帧的作用是立刻刷新，重新算一个新的序列开始编码，使错误不致传播。
	- 参见：[《视频编码（1）》第 1.1.7 节](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)
1. `H.264 有哪些压缩方式？`
	- 帧内压缩和帧间压缩。
	- 参见：[《视频编码（1）》第 1.1.8 节](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)
1. `H.264 编码框架是如何分层的？`
	- 分为两个层面：视频编码层面（VCL）和网络抽象层面（NAL）。
	- 参见：[《视频编码（1）》第 1.2 节](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)
1. `视频编码层（VCL）是什么？`
	- 对视频编码核心算法过程、子宏块、宏块、片等概念的定义。这层主要是为了尽可能的独立于网络来高效的对视频内容进行编码。
	- 参见：[《视频编码（1）》第 1.2 节](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)
1. `网络适配层（NAL）是什么？`
	- 对图像序列、图像等片级别以上的概念的定义。这层负责将 VCL 产生的比特字符串适配到各种各样的网络和多元环境中。
	- 参见：[《视频编码（1）》第 1.2 节](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)
1. `什么是 SODB、RBSP、NAL？`
	- SODB（String Of Data Bits）是 VCL 层编码后输出的原始数据；
	- RBSP（Raw Byte Sequence Payload）是 NAL 层在 SODB 数据后面添加了结尾比特进行字节对齐后的数据；
	- NAL 是在 RBSP 头部加上 NAL Header 来组成一个一个的 NAL 数据单元。
	- 参见：[《视频编码（1）》第 1.2 节](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)
1. `H.264 的编码工具有哪些？`
	- 帧内预测、帧间预测、离散余弦变换（DCT）、量化、上下文自适应的变长编码（CAVLC）、上下文自适应的二进制算术编码（CABAC）。
	- 参见：[《视频编码（1）》第 1.3 节](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)
1. `H.264 的编码流程是怎样的？`
	- 预测 → 变换 → 量化 → 熵编码。
	- 参见：[《视频编码（1）》第 1.3 节](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)
1. `什么是 Annex-b 码流格式？`
	- 当数据流是存储在介质上时，在每个 NAL 前添加起始码 0x000001 的码率格式。
	- 参见：[《视频编码（1）》第 1.4.1 节](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)
1. `Annex-b 码流格式如何防止竞争？`
	- 通过对特定序列插入 0x03：0x000000 → 0x00000300、0x000001 → 0x00000301、0x000002 → 0x00000302、0x000003 → 0x00000303。
	- 参见：[《视频编码（1）》第 1.4.1 节](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)
1. `什么是 SPS、PPS？`
	- SPS 是序列参数集，保存了一组编码后的图像序列的依赖的全局参数。
	- PPS 是图像参数集，保存了每一帧编码后的图像所依赖的参数。
	- 参见：[《视频编码（1）》第 1.4.4 节](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)
1. `什么是 SEI？`
	- SEI 是补充增强信息，属于码流范畴，它提供了向视频码流中加入额外信息的方法。
	- 参见：[《视频编码（1）》第 1.4.4 节](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)
1. `什么是 Slice？`
	- 一帧图像可编码成一个或者多个片，每片包含整数个宏块，分片的目的是为了限制错误码的扩散和传输，使编码片相互间保持独立。
	- 参见：[《视频编码（1）》第 1.4.4 节](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)
1. `H.265 有哪些编码工具？`
	- 帧内预测、帧间预测、变换和量化、去方块滤波（DBF）、样点自适应补偿滤波（SAO）、熵编码。
	- 参见：[《视频编码（2）》开篇简介](https://mp.weixin.qq.com/s/CAgnEIaQQ-6lT1KjESRApA)
1. `H.265 相比 H.264 有特色编码技术？`
	- 新的编码单元划分方式、改进的帧内预测技术、先进的帧间预测技术、RQT 技术、ACS 技术、SAO 技术、IBDI 技术。
	- 参见：[《视频编码（2）》开篇简介](https://mp.weixin.qq.com/s/CAgnEIaQQ-6lT1KjESRApA)
1. `什么是 RQT 技术？`
	- 一种基于四叉树结构的自适应变换技术。
	- 参见：[《视频编码（2）》第 2.2.4 节](https://mp.weixin.qq.com/s/CAgnEIaQQ-6lT1KjESRApA)
1. `什么是 ACS 技术？`
	- 基于 4x4 块单元，支持对角、水平和垂直方向的帧内预测扫描技术。
	- 参见：[《视频编码（2）》第 2.2.5 节](https://mp.weixin.qq.com/s/CAgnEIaQQ-6lT1KjESRApA)
1. `什么是去方块滤波（DBF）技术？`
	- 在 TU/PU 块边界根据 MV、QP 等进行不同强度滤波，以减少基于块的视频编码中重构图像会出现的方块效应。
	- 参见：[《视频编码（2）》第 2.1.4 节](https://mp.weixin.qq.com/s/CAgnEIaQQ-6lT1KjESRApA)
1. `什么是 SAO 技术？`
	- 通过补偿重构像素值，以减少振铃效应。
	- 参见：[《视频编码（2）》第 2.2.6 节](https://mp.weixin.qq.com/s/CAgnEIaQQ-6lT1KjESRApA)
1. `什么是 IBDI 技术？`
	- 在编码器的输入端将未压缩图像像素深度由 P 比特增加到 Q 比特（Q > P），在解码器的输出端又将解压缩图像像素深度从 Q 比特恢复到 P 比特，从而提高了编码器编码精度，降低了帧内/帧间预测误差。
	- 参见：[《视频编码（2）》第 2.2.7 节](https://mp.weixin.qq.com/s/CAgnEIaQQ-6lT1KjESRApA)
1. `H.266 有哪些编码工具？`
	- 块划分、帧内预测、帧间预测、变换和量化、熵编码、环路滤波、屏幕内容编码、360 度视频编码。
	- 参见：[《视频编码（3）》第 3 节](https://mp.weixin.qq.com/s/w7xIji29JKYf8cI2BxHZyw)
1. `从 H.264 到 H.265 再到 H.266，编码单元划分技术有什么改进？`
	- H.264 采用宏块、子宏块划分结构，最小 4x4，最大 16x16；
	- H.265 采用四叉树（QT）划分结构，最大 CTU 64×64；
	- H.266 采用四叉树（QT）、多类型树划（MTT）分结构，最大 CTU 128×128。
	- 参见：[《视频编码（2）》第 2.2.1 节](https://mp.weixin.qq.com/s/CAgnEIaQQ-6lT1KjESRApA)、[《视频编码（3）》第 3.1.1 节](https://mp.weixin.qq.com/s/w7xIji29JKYf8cI2BxHZyw)
1. `从 H.264 到 H.265 再到 H.266，帧内预测技术有什么改进？`
	- 支持更多的预测模式：H.264 4x4 9 种、16x16 4 种 → H.265 35 种 → H.266 67 种。
	- 参见：[《视频编码（2）》第 2.2.2 节](https://mp.weixin.qq.com/s/CAgnEIaQQ-6lT1KjESRApA)、[《视频编码（3）》第 3.1.2 节](https://mp.weixin.qq.com/s/w7xIji29JKYf8cI2BxHZyw)
1. `从 H.264 到 H.265 再到 H.266，帧间预测技术有什么改进？`
	- H.264 帧间预测是利用已编码视频帧/场和基于块的运动补偿的预测模式。
	- H.265 引入了新的帧间预测技术，包括运动信息融合技术（Merge）、先进的运动矢量预测技术（AMVP）以及基于 Merge 的 Skip 模式。
	- H.266 继承了 H.265 AMVP 和 Skip/Merge 模式并进行了扩展、引入了基于子块的时域运动推导模式（SbTMVP）、引进一个仿射运动模型、引入多个新的帧间预测编码工具、引入解码端运动细化和双向光流工具。
	- 参见：[《视频编码（2）》第 2.2.3 节](https://mp.weixin.qq.com/s/CAgnEIaQQ-6lT1KjESRApA)、[《视频编码（3）》第 3.1.3 节](https://mp.weixin.qq.com/s/w7xIji29JKYf8cI2BxHZyw)
1. `H.266 在变换和量化上有什么改进？`
	- 变换：最大变换维度从 H.265 的 32x32 提高到了 64×64；引入了非正方形变换、多变换（主变换）选择、低频不可分变换、子块变换。
	- 量化：引入了自适应色度量化参数偏差、依赖量化、量化残差联合编码。
	- 参见：[《视频编码（3）》第 3.1.4 节](https://mp.weixin.qq.com/s/w7xIji29JKYf8cI2BxHZyw)
1. `H.266 在熵编码上有什么改进？`
	- 在 CABAC 引擎方面，使用多重假设概率更新模型和上下文模型绑定的自适应率（即概率更新速度依赖于上下文模型）；
	- 在变换系数编码方面，H.266 还允许更多类型的系数组、增加了一个标志位用于依赖量化的状态过渡、一个改进的概率模型选择机制用于和变换系数绝对值相关的语法元素的编码。
	- 参见：[《视频编码（3）》第 3.1.5 节](https://mp.weixin.qq.com/s/w7xIji29JKYf8cI2BxHZyw)
1. `H.266 在环路滤波上有什么改进？`
	- 除了继续支持优化后的去方块滤波（DBF）和样点自适应补偿滤波（SAO）外，新增了带色度缩放的亮度映射（LMCS）和自适应环路滤波器（ALF）。
	- 参见：[《视频编码（3）》第 3.1.6 节](https://mp.weixin.qq.com/s/w7xIji29JKYf8cI2BxHZyw)
1. `什么是屏幕内容编码？`
	- 屏幕内容图像是直接从各类计算机或移动终端设备的图像显示单元捕获的图像，如：计算机图形和文本图像、自然视频与图形、文字混合的图像以及计算机生成的动画图像等。对这种内容进行编码称为屏幕内容编码。
	- 参见：[《视频编码（3）》第 3.1.7 节](https://mp.weixin.qq.com/s/w7xIji29JKYf8cI2BxHZyw)
1. `屏幕内容图像与自然图像有什么差别？`
	- 屏幕内容图像通常没有噪声、色调离散、线条细腻、边缘锐利；自然图像通常有噪声、色调连续、纹理比较复杂。
	- 参见：[《视频编码（3）》第 3.1.7 节](https://mp.weixin.qq.com/s/w7xIji29JKYf8cI2BxHZyw)
1. `什么是 360 度视频编码？`
	- 针对全景视频的编码。
	- 参见：[《视频编码（3）》第 3.1.8 节](https://mp.weixin.qq.com/s/w7xIji29JKYf8cI2BxHZyw)
1. `H.266 针对 360 度视频有什么编码工具？`
	- 运动矢量环绕和环路滤波虚拟边界。
	- 参见：[《视频编码（3）》第 3.1.8 节](https://mp.weixin.qq.com/s/w7xIji29JKYf8cI2BxHZyw)
1. `H.266 在高层语法（HLS）层有哪些改进？`
	- 矩形条带（Slice）和子图像（Subpicture）、自适应图像分辨率更新、自适应参数集（APS）、图像头、逐渐解码刷新（GDR）、参考图像列表（RPL）的直接编码、多层可伸缩编码设计大大简化。
	- 参见：[《视频编码（3）》第 3.2 节](https://mp.weixin.qq.com/s/w7xIji29JKYf8cI2BxHZyw)




## 3、音视频格式


1. `为什么要对音视频编码数据进行封装？`
	- 便于对音视频的展现做控制，提高对音视频数据处理的效率。比如进行音视频同步、seek 等操作。
1. `什么是 MP4？`
	- 一种标准的数字多媒体容器格式，指 MPEG-4 第 14 部分。
	- 参见：[《MP4 格式》开篇简介](https://mp.weixin.qq.com/s/GAYkhrakhQlkpnWwKrFeyg)
1. `什么是 moov Box？`
	- Movie Box，MP4 中存储所有媒体数据的索引信息的 Box。
	- 参见：[《MP4 格式》第 3 节](https://mp.weixin.qq.com/s/GAYkhrakhQlkpnWwKrFeyg)
1. `为什么 moov Box 要前置？`
	- 播放器从网络读取和播放 MP4 文件时，要获取到 moov 的数据后才能初始化解码器并开始播放。
	- 参见：[《MP4 格式》第 3 节](https://mp.weixin.qq.com/s/GAYkhrakhQlkpnWwKrFeyg)
1. `什么是 mdat Box？`
	- Media Data Box，MP4 中存储音视频实际数据的 Box。
	- 参见：[《MP4 格式》第 4、5.1 节](https://mp.weixin.qq.com/s/GAYkhrakhQlkpnWwKrFeyg)
1. `MP4 视频最少加载多少数据可以渲染出首帧？`
	- 结合 Sync Sample Box(moov/trak/mdia/minf/stbl/stss)、Sample To Chunk Box(moov/trak/mdia/minf/stbl/stsc)、Sample Size Box(moov/trak/mdia/minf/stbl/stsz)、Chunk Offset Box(moov/trak/mdia/minf/stbl/stco(co64)) 这几个 Box 的信息可以计算出读到第一个关键帧需要的字节数。
	- 参见：[《MP4 格式》第 5.2 节](https://mp.weixin.qq.com/s/GAYkhrakhQlkpnWwKrFeyg)
1. `什么是 FLV？`
	- Adobe 公司推出的一种封装规范简单、封装后文件较小、适合网络传输的流媒体格式。
	- 参见：[《FLV 格式》开篇简介](https://mp.weixin.qq.com/s/M0g6DheyUX9a9s_qpVU1Gw)
1. `为什么直播首选 FLV 作为封装格式？`
	- 基于 RTMP 推流、HTTP-FLV 播放的整套方案延时较低；服务端普遍提供 HTTP Web 服务，能更广泛的兼容 HTTP-FLV。
	- 参见：[《FLV 格式》开篇简介](https://mp.weixin.qq.com/s/M0g6DheyUX9a9s_qpVU1Gw)
1. `FLV 中有哪些类型的 Tag？`
	- Audio Tags、Video Tags、Script Tags。
	- 参见：[《FLV 格式》第 1 节](https://mp.weixin.qq.com/s/M0g6DheyUX9a9s_qpVU1Gw)
1. `什么是 FLV 中的 AAC 音频同步包？`
	- 在 FLV 的文件中，一般会用第一个 Audio Tag 来封装 AudioSpecificConfig。如果音频使用 AAC 编码格式，那么这个 Tag 就是 AAC sequence header，即 AAC 音频同步包。
	- 参见：[《FLV 格式》第 2 节](https://mp.weixin.qq.com/s/M0g6DheyUX9a9s_qpVU1Gw)
1. `用 FLV 封装 AAC 并传输音频流时，要如何处理 ADTS 头？`
	- 发送数据包时，去掉 AAC 数据的 ADTS 头；接收和解析数据包，在 AAC 裸数据前加上 ADTS 头再给解码器。
	- 参见：[《FLV 格式》第 2 节](https://mp.weixin.qq.com/s/M0g6DheyUX9a9s_qpVU1Gw)
1. `什么是 FLV 中的 AVC(H.264) 视频同步包？`
	- 在 FLV 的文件中，一般会用第一个 Video Tag 来封装 AVCDecoderConfigurationRecord。如果视频使用 AVC 编码格式，那么这个 Tag 就是 AVC sequence header，即 AVC 视频同步包。
	- 参见：[《FLV 格式》第 3 节](https://mp.weixin.qq.com/s/M0g6DheyUX9a9s_qpVU1Gw)
1. `什么是 M3U8？`
	- 一种指定一个或多个多媒体文件位置的播放列表纯文本文件格式，是 HLS 协议的基础。
	- 参见：[《M3U8 格式》开篇简介](https://mp.weixin.qq.com/s/tCWED5zSMMYF6TTlRyljdA)
1. `为什么实时直播一般不选择 M3U8 格式？`
	- HLS/M3U8/TS 这套方案在控制直播延时上不太理想。
	- 参见：[《M3U8 格式》开篇简介](https://mp.weixin.qq.com/s/tCWED5zSMMYF6TTlRyljdA)
1. `为什么直播回放一般选择 M3U8 格式？`
	- 能够在直播过程中就持续生成和存储切片。
	- 参见：[《M3U8 格式》开篇简介](https://mp.weixin.qq.com/s/tCWED5zSMMYF6TTlRyljdA)
1. `什么是 M3U8 媒体播放列表？`
	- M3U8 包含的信息是一个媒体资源一路流对应的一系列切片。
	- 参见：[《M3U8 格式》第 1.1 节](https://mp.weixin.qq.com/s/tCWED5zSMMYF6TTlRyljdA)
1. `什么是 M3U8 主播放列表？`
	- M3U8 包含的信息是同一个媒体资源的多路流资源列表。
	- 参见：[《M3U8 格式》第 1.2 节](https://mp.weixin.qq.com/s/tCWED5zSMMYF6TTlRyljdA)
1. `EXT-X-DISCONTINUITY 标签表示什么？`
	- 表示其前一个切片和下一个切片之间存在中断。在媒体文件格式、媒体轨道的数量和类型、时间戳序列、编码参数、编码序列的内容发生变化时，需要使用该标签。
	- 参见：[《M3U8 格式》第 2.3.2 节](https://mp.weixin.qq.com/s/tCWED5zSMMYF6TTlRyljdA)
1. `什么是 TS？`
	- Transport Stream，一种音视频封装格式，全称 MPEG2-TS。
	- 参见：[《TS 格式》开篇简介](https://mp.weixin.qq.com/s/IsaJZZ1RlCm8KPOYtlEoRQ)
1. `为什么直播回放的切片一般用 TS 格式？`
	- TS 任一切片开始都可以独立解码，非常适合按切片的方式存储直播内容。
	- 参见：[《TS 格式》开篇简介](https://mp.weixin.qq.com/s/IsaJZZ1RlCm8KPOYtlEoRQ)
1. `TS 格式支持 seek 吗？`	
	- TS 流中不支持 seek。
	- 参见：[《TS 格式》开篇简介](https://mp.weixin.qq.com/s/IsaJZZ1RlCm8KPOYtlEoRQ)
1. `如何实现对 TS 传输流的 seek？`
	- 需要从协议层支持。比如 HLS 协议对相关能力做了定义。
	- 参见：[《TS 格式》开篇简介](https://mp.weixin.qq.com/s/IsaJZZ1RlCm8KPOYtlEoRQ)
1. `TS 流结构整体分为哪几层？`
	- TS 传输流层；PES 分组化基本流；ES 基本流层。
	- 参见：[《TS 格式》第 1 节](https://mp.weixin.qq.com/s/IsaJZZ1RlCm8KPOYtlEoRQ)
1. `TS 传输流层是什么数据？`
	- 对分组化基本流数据打包，加上了用于网络传输相关的信息。
	- 参见：[《TS 格式》第 1 节](https://mp.weixin.qq.com/s/IsaJZZ1RlCm8KPOYtlEoRQ)
1. `PES 分组化基本流层是什么数据？`
	- 主要在音视频数据的基础上带上了时间戳信息。
	- 参见：[《TS 格式》第 1 节](https://mp.weixin.qq.com/s/IsaJZZ1RlCm8KPOYtlEoRQ)
1. `ES 基本流层是什么数据？`
	- 主要包含实际的音视频数据，一般视频为 H.264 编码数据，音频为 AAC 编码数据。
	- 参见：[《TS 格式》第 1 节](https://mp.weixin.qq.com/s/IsaJZZ1RlCm8KPOYtlEoRQ)
1. `TS 层的数据包大小是多少？`
	- 固定 188 字节。
	- 参见：[《TS 格式》第 2 节](https://mp.weixin.qq.com/s/IsaJZZ1RlCm8KPOYtlEoRQ)







## 4、音视频协议


1. `为什么设计音视频传输协议？`
	- 在网络中高效传输音视频数据。
1. `什么是 RTMP？`
	- Real Time Message Protocol，由 Adobe 公司提出的一种应用层的实时信息传输协议。
	- 参见：[《RTMP 协议》第 1 节](https://mp.weixin.qq.com/s/237aG7SE0BlDtXSkDLXubw)
1. `为什么直播推流首选 RTMP 协议？`
	- 协议设计对低延时、音视频同步等能力有良好的支持。
	- 参见：[《RTMP 协议》第 1 节](https://mp.weixin.qq.com/s/237aG7SE0BlDtXSkDLXubw)
1. `RTMP 设计的目标是解决什么问题？`
	- 解决多媒体数据传输流的分包和多路复用的问题。
	- 参见：[《RTMP 协议》第 1 节](https://mp.weixin.qq.com/s/237aG7SE0BlDtXSkDLXubw)
1. `什么是 RTMP 中的消息和块？`
	- 消息，服务于数据封装，是 RTMP 协议中的基本数据单元；块，服务于网络传输，是对消息的分片。
	- 参见：[《RTMP 协议》第 1.2 节](https://mp.weixin.qq.com/s/237aG7SE0BlDtXSkDLXubw)
1. `RTMP 的核心设计思想是什么？`
	- 分包、多路复用、消息层的优先级划分、块大小协商、压缩优化。
	- 参见：[《RTMP 协议》第 1.2 节](https://mp.weixin.qq.com/s/237aG7SE0BlDtXSkDLXubw)
1. `RTMP 分包的设计有什么好处？`
	- 可以将大的消息数据分包成小的块通过网络来进行传输，是降低延时的关键。
	- 参见：[《RTMP 协议》第 1.2 节](https://mp.weixin.qq.com/s/237aG7SE0BlDtXSkDLXubw)
1. `RTMP 多路复用的设计有什么好处？`
	- 音频、视频数据就能够合到一个传输流（块流）中进行同步传输，是音视频同步的关键。
	- 参见：[《RTMP 协议》第 1.2 节](https://mp.weixin.qq.com/s/237aG7SE0BlDtXSkDLXubw)
1. `RTMP 消息分优先级的设计有什么好处？`
	- 优先级：控制消息 > 音频消息 > 视频消息。当网络传输能力受限时，优先传输高优先级消息的数据。
	- 要使优先级能够有效执行，分块也很关键：将大消息切割成小块，可以避免大的低优先级的消息（如视频消息）堵塞了发送缓冲从而阻塞了小的高优先级的消息（如音频消息或控制消息）。
	- 参见：[《RTMP 协议》第 1.2 节](https://mp.weixin.qq.com/s/237aG7SE0BlDtXSkDLXubw)
1. `RTMP 块大小协商的设计有什么好处？`
	- 充分考虑流媒体服务器、带宽、客户端的情况，通过块大小协商动态的适应环境。
	- 参见：[《RTMP 协议》第 1.2 节](https://mp.weixin.qq.com/s/237aG7SE0BlDtXSkDLXubw)
1. `RTMP 块 Header 压缩优化的设计有什么好处？`
	- 块 Header 最大 12 字节，最小可以压缩到 1 字节，节省带宽。
	- 参见：[《RTMP 协议》第 1.2 节](https://mp.weixin.qq.com/s/237aG7SE0BlDtXSkDLXubw)
1. `什么是 KCP？`
	- 一个开源的快速可靠的传输协议。能以比 TCP 浪费 10%-20% 带宽的代价，换取平均延迟降低 30%-40%，最大延迟降低 3 倍的传输速度。
	- 参见：[《KCP 协议》开篇简介](https://mp.weixin.qq.com/s/-O3tGTisekuTwgzHSbx6dg)
1. `KCP 的核心设计思想是什么？`
	- 为流速和低延时设计：RTO 不翻倍、选择性重传、快速重传、ACK + UNA、非退让流控。
	- 参见：[《KCP 协议》第 2 节](https://mp.weixin.qq.com/s/-O3tGTisekuTwgzHSbx6dg)
1. `KCP RTO 不翻倍的设计有什么好处？`
	- RTO 是重传超时时间，KCP 启动快速模式后不像 TCP 那样 x2，只是 x1.5，相对提高了传输速度。
	- 参见：[《KCP 协议》第 2.1 节](https://mp.weixin.qq.com/s/-O3tGTisekuTwgzHSbx6dg)
1. `KCP 选择重传的设计有什么好处？`
	- 只重传真正丢失的数据包，提高传输速度。
	- 参见：[《KCP 协议》第 2.2 节](https://mp.weixin.qq.com/s/-O3tGTisekuTwgzHSbx6dg)
1. `KCP 快速重传的设计有什么好处？`
	- 当一个包被跳过的次数超过阈值，不用等待超时，直接重传，大大改善了丢包时的传输速度。
	- 参见：[《KCP 协议》第 2.3 节](https://mp.weixin.qq.com/s/-O3tGTisekuTwgzHSbx6dg)
1. `KCP ACK + UNA 的设计有什么好处？`
	- ARQ（自动重传请求）模型响应有两种：UNA，表示此编号之前的所有包已收到；ACK：表示此编号包已收到。KCP 除了单独的 ACK 包外，其他所有包都有 UNA 信息，降低全部重传的概率。
	- 参见：[《KCP 协议》第 2.4 节](https://mp.weixin.qq.com/s/-O3tGTisekuTwgzHSbx6dg)
1. `KCP 非退让流控的设计有什么好处？`
	- 以牺牲部分公平性和带宽利用率的代价，换取了流畅传输的收益。
	- 参见：[《KCP 协议》第 2.5 节](https://mp.weixin.qq.com/s/-O3tGTisekuTwgzHSbx6dg)
1. `什么是 HLS？`
	- HTTP Live Streaming，由苹果公司提出的一种流媒体传输协议，可支持流媒体的直播和点播。
	- 参见：[《HLS 协议》第 1 节](https://mp.weixin.qq.com/s/c4p_6xxQxwYZON2W2luEyA)
1. `HLS 的核心设计思想是什么？`
	- 按切片的方式对媒体文件进行生产、传输、存储。
	- 参见：[《HLS 协议》第 1 节](https://mp.weixin.qq.com/s/c4p_6xxQxwYZON2W2luEyA)
1. `HLS 有什么优点？`
	- 数据通过 HTTP 协议传输，不用考虑防火墙或者代理的问题；切片文件的时长很短，客户端可以很快的选择和切换码率，以适应不同带宽条件下的播放。
	- 参见：[《HLS 协议》第 1 节](https://mp.weixin.qq.com/s/c4p_6xxQxwYZON2W2luEyA)
1. `HLS 有什么缺点？`
	- 延迟一般会高于普通的流媒体直播协议。
	- 参见：[《HLS 协议》第 1 节](https://mp.weixin.qq.com/s/c4p_6xxQxwYZON2W2luEyA)
1. `典型的 HLS 选择什么封装格式？`
	- M3U8 + TS。M3U8 作为索引文件；TS 作为音视频数据封装文件。
	- 参见：[《HLS 协议》第 1 节](https://mp.weixin.qq.com/s/c4p_6xxQxwYZON2W2luEyA)


## 音视频基础文章列表


- [声音的表示（1）：声音的定义和特征](https://mp.weixin.qq.com/s/b4XkNaZWnSLx8KxqV0Ul1A)
- [声音的表示（2）：声音的数学描述](https://mp.weixin.qq.com/s/LOUnbvYSNBONE9TgCxjLvw)
- [声音的表示（3）：声音的数字化](https://mp.weixin.qq.com/s/lexAVx_O3Kz3-51OZ3TlLw)
- [图像的表示（1）：图像的定义和成像原理](https://mp.weixin.qq.com/s/fOiV32SZb7UcN6KLUCI61g)
- [图像的表示（2）：图像的数学描述](https://mp.weixin.qq.com/s/7QMpCe3YBgKxtHM260SZYA)
- [图像的表示（3）：图像的数字化](https://mp.weixin.qq.com/s/RhMLfOoDdf6TvRoGVQaNHA)
- [音频编码：PCM 和 AAC](https://mp.weixin.qq.com/s/tVUgH17HZe6PWj09hYzkbg)
- [视频编码（1）：H.264（AVC）](https://mp.weixin.qq.com/s/KX3wEv1Rb2sXg29P1wB88A)
- [视频编码（2）：H.265（HEVC）](https://mp.weixin.qq.com/s/CAgnEIaQQ-6lT1KjESRApA)
- [视频编码（3）：H.266（VVC）](https://mp.weixin.qq.com/s/w7xIji29JKYf8cI2BxHZyw)
- [MP4 格式：短视频常用格式](https://mp.weixin.qq.com/s/GAYkhrakhQlkpnWwKrFeyg)
- [FLV 格式：直播常用格式](https://mp.weixin.qq.com/s/M0g6DheyUX9a9s_qpVU1Gw)
- [M3U8 格式：直播回放常用格式](https://mp.weixin.qq.com/s/tCWED5zSMMYF6TTlRyljdA)
- [TS 格式：直播回放切片常用格式](https://mp.weixin.qq.com/s/IsaJZZ1RlCm8KPOYtlEoRQ)
- [RTMP 协议：直播推流常用协议](https://mp.weixin.qq.com/s/237aG7SE0BlDtXSkDLXubw)
- [KCP 协议：自研常用参考协议](https://mp.weixin.qq.com/s/-O3tGTisekuTwgzHSbx6dg)
- [HLS 协议：直播回放常用协议](https://mp.weixin.qq.com/s/c4p_6xxQxwYZON2W2luEyA)












