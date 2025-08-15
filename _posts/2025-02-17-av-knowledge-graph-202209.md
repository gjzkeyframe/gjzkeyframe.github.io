---
title: 音视频知识图谱 2022.09
description: 持续更新的音视频知识图谱。
author: Keyframe
date: 2025-02-17 08:38:08 +0800
categories: [音视频知识图谱]
tags: [音视频知识图谱, 音视频]
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

前些时间，我在知识星球上创建了一个音视频技术社群：**关键帧的音视频开发圈**，在这里群友们会一起做一些打卡任务。比如：周期性地整理音视频相关的面试题，汇集一份**音视频面试题集锦**，你可以看看这个合集：[音视频面试题集锦](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2380776196751425539#wechat_redirect)。再比如：循序渐进地归纳总结音视频技术知识，绘制一幅**音视频知识图谱**，你可以看看这个合集：[音视频知识图谱](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2349658423078092802#wechat_redirect)。

下面是 2022.09 月知识图谱新增的内容节选：


1）图谱路径：**图像算法/图像识别**

- 留一法（LEAVE-ONE-OUT）
	- 将输入图像分割为一系列子区域，运行一系列预测，每次遮罩（即将像素值设置为零）其中一个子区域，根据每个区域的「蒙版」相对于原始图像影响预测的程度，为每个区域分配一个重要度分数。分数量化了哪一部分的区域最有助于进行预测。
	- 优点：简单、灵活、通用。
	- 缺点：速度慢、没考虑区域之间相互依赖性。
- 梯度上升（VANILLA GRADIENT ASECENT [2013]）
	- 使用标准的反向传播，可以计算出模型损失相对于权值的梯度。梯度是一个包含每个权重值的向量，反映了该权重的微小变化将对输出产生了多大的影响，并从本质上告诉我们哪些权重对于损失最重要。通过取该梯度的负值，我们可以将训练过程中的损失降到最低。对于梯度上升，取而代之的是类分数相对于输入像素的梯度，并告诉我们哪些输入像素对图像分类最重要。通过网络的这一单个步骤为我们提供了每个像素的重要性值。
	- 优点：速度快。
	- 缺点：Vanilla 梯度上升的原始公式会传播负梯度，会导致干扰和噪声的输出。
	- 论文：Visualizing Image Classification Models and Saliency Maps [2013]。
- 引导反向传播（GUIDED BACK-PROPOGATION [2014]）
	- 在反向传播的常规步骤中增加一个来自更高层的额外引导信号。当输出为负时，该方法会阻止来自神经元的梯度反向流动，仅保留导致输出增加的梯度，减少噪声。
	- 优点：速度快、物体边缘附近输出更清晰。
	- 缺点：图像中存在两个以上类别时，通常无法正常工作。
	- 论文：Striving for Simplicity: The All Convolutional Net [2014]。
- 梯度类别响应图（GRAD-CAM [2016]）
	- 当在最后一个卷积层的每个滤波器处而不是在类分数上（但仍相对于输入像素）提取梯度时，其解释的质量得到了改善。为了得到特定于类的解释，Grad-CAM 对这些梯度进行加权平均，其权重基于过滤器对类分数的贡献。结果远远好于单独的引导反向传播。
	- 论文：Visual Explanations from Deep Networks via Gradient-based Localization [2016]。
- 平滑梯度（SMOOTHGRAD [2017]）
	- 如果输入图像首先受到噪声干扰，则可以为每个版本的干扰输入计算一次梯度，然后将灵敏度图平均化。
	- 优点：会得到更清晰的结果。
	- 缺点：运行时间更长。
	- 论文：SmoothGrad, presented in SmoothGrad: removing noise by adding noise [2017]。
- 集成梯度（INTEGRATED GRADIENTS [2017]）
	- 优点：通常可以产生更准确的灵敏度图。
	- 缺点：速度较慢。
	- 论文：Axiomatic Attribution for Deep Networks [2017]。
- 模糊集成梯度（BLUR INTEGRATED GRADIENTS [2020]）
	- 通过测量一系列原始输入图像逐渐模糊的版本梯度（而不是像集成梯度那样变暗的图像），旨在解决具有集成梯度的特定问题，包括消除「基线」参数，并消除某些易于在解释中出现的视觉伪像。
	- 论文：Attribution in Scale and Space [2020]。

参考资料：

- 图像识别解释方法的视觉演变：https://mp.weixin.qq.com/s/smEpKeM14ACWAjL8iumDRA


2）图谱路径：**图像算法/图像降噪**


- 分类：滤波（Filters）
	- 均值滤波（Mean Filter），均值滤波是典型的线性滤波算法，它是指在图像上对目标像素给一个模板，该模板包括了其周围的临近像素（以目标像素为中心的周围 8 个像素，构成一个滤波模板，即包括目标像素本身），再用模板中的全体像素的平均值来代替原来像素值。
	- 高斯滤波（Gauss Filter），高斯滤波就是对整幅图像进行加权平均的过程，每一个像素点的值，都由其本身和邻域内的其他像素值经过加权平均后得到。高斯滤波的具体操作是：用一个模板（或称卷积、掩模）扫描图像中的每一个像素，用模板确定的邻域内像素的加权平均灰度值去替代模板中心像素点的值。
	- 中值滤波（Medium Filter），中值滤波法是一种非线性平滑技术，它将每一像素点的灰度值设置为该点某邻域窗口内的所有像素点灰度值的中值。
	- 双边滤波（Bilateral Filter），双边滤波本质是基于高斯滤波，目的结合图像的空间邻近度和像素值相似度来解决高斯滤波造成的边缘模糊的一种滤波算法，同时考虑空域信息和灰度相似性，达到保边去噪的目的。具有简单、非迭代、局部的特点。
	- 导向滤波（Guided Filter），导向滤波通过输入一副图像（矩阵）作为导向图，这样滤波器就知道什么地方是边缘，这样就可以更好的保护边缘，最终达到在滤波的同时，保持边缘细节。所以有个说法是导向滤波是各向异性的滤波器，而高斯滤波、双边滤波这些是各向同性滤波器，我觉得也是很贴切。
	- 非局部均值滤波（NLM，Non-Local Mean Filter）
	- 离散傅里叶变换（Discrete Fourier Transform）
	- 维纳滤波（Weiner Filter），假设观察信号 y(t) 含有彼此统计独立的期望信号 x(t) 和白噪声 ω(t)，可用维纳滤波从观察信号 y(t) 中恢复期望信号 x(t)。
	- 离散小波变换（Discrete Wavelets Transform）
	- 脊波变换（Ridgelet Transform）
	- 曲波变换（Curvelet Transform）
	- 轮廓波变换（Contourlet Transform）
	- 块匹配 3D 协同过滤（Block Matching 3D Collaborative Filtering）
	- PID（Progressive Image Denoising）
- 分类：时域降噪算法
	- EDVR，Video Restoration with Enhanced Deformable Convolutional Networks
	- FastDVDNet，FastDVDNet 是一种视频去噪的 STOA（乌燕鸥优化算法，Sooty Tern Optimization Algorithm） 方法，与其他 STOA 方法有着相近或者更好的性能，但是有着更低的时间复杂度。
- 分类：稀疏表达（Spare Representation）
	- K-SVD，K-Singular Value Decomposition
	- LSSC，Non-Local Spare Models
	- NCSR，Non-Local Centralized Sparse Representation
- 分类：聚类低秩（Low Rankness）
	- NNM（Nuclear Nom Minimization）
	- WNNM（Weighted Nuclear Nom Minimization）
- 分类：统计模型（Statistical Model）
	- 隐马尔可夫模型（HMM，Hidden Markov Model）
	- 高斯混合模型（Gaussian Mixture Model）
- 分类：深度学习（Deep Learning）
	- DnCNN
	- FFDNet

3）图谱路径：**图像算法/图像增强**

- 分类：直方图均衡图像增强
	- 直方图均衡化算法，简言之就是对图像直方图的每个『灰度级』来进行统计。实现『归一化』的处理，再对每一灰度值求『累积分布』的结果，可求得它的『灰度映射表』，由灰度映射表，可对原始图像中的对应像素来进行『修正』，生成一个修正后的图像。
	- 传统标准直方图均衡算法：传统直方图均衡算法是通过图像灰度级的映射，在变换函数作用下，呈现出『相对均匀分布的输出图像灰度级』，『增强了图像的对比度』。
	- 保持亮度的双直方图均衡算法：BBHE 实质是利用两个独立的子图像的『直方图等价性』。两个子图像的直方图等价性是根据输入图像的均值对其进行分解得到，其『约束条件』是得到均衡化后的子图像在输入均值附近彼此有界作为基于图像均值进行的分割，均衡后图像均值偏离原始图像均值的现象不会出现，达到了『亮度保持』的目的。
- 分类：小波变换图像增强
	- 高通滤波：高频信号可以通过，而低频信号不能通过。
	- 通过小波逆变换将同态滤波处理的低频分量和经自应阈值噪、改进模糊增强的高频分量得到增强处理后的红外图像。
	- 标准小波变换图像增强：小波理论具有低熵和多分辨率的性质，『处理小波系数』对降噪有一定作用，噪声主要在高通系数中呈现，对高低通子带均需要增强对比度和去噪处理。标准小波变换图像增强(WT)将图像分解为 1 个低通子图像和 3 个具有方向性的高通子图像，高通子图像包括水平细节图像、垂直细节图像和对角细节图像。
	- 改进后的小波变换图像增强算法：针对传统方法对图像『多聚焦模糊特征』进行增强会出现图像不清晰、细节丢失现象，小波变换图像多聚焦模糊特征增强方法，利用『背景差分法』对目标图像的『提取前景区域』，背景区域亮度会随时间发生变化，进而完成背景区域特征更新；根据全局像素点熵值和预设阈值校正加强模糊特征，突出小波变换图像边界局部纹理细节信息，完成增强变换。
- 分类：偏微分方程图像增强
	- 标准偏微分方程图像增强
	- 改进的偏微分方程增强方法：为避免增强图像梯度场同时造成噪声的危害加剧，寻找的一种比较适合的增强方法。
- 分类：分数阶微分的图像增强
	- 图像增强的分数阶微分算子构造
	- 改进的分数阶微分算子增强图像
- 分类：基于 Retinex 理论的图像增强
	- Retinex 理论的基础是人类视觉系统的色彩恒常性，『人类视觉感知系统的色知觉存在先入为主的特性』，即光源条件发生改变，视网膜接收到的彩色信息也会被人们的大脑驳回。
	- 经典的 Retinex 图像增强：单尺度 Retinex 算法（SSR）、多尺度 Retinex 算法（MSR）、带色彩恢复的多尺度 Retinex 算法（MSMCR）
	- 改进的 Retinex 图像增强
- 分类：基于深度学习的图像增强算法
	- 卷积神经网络图像增强算法
	- 基于深度学习图像增强的改进算法



---

下面是 2022.09 月的知识图谱新增内容快照（图片被平台压缩不够清晰，可以加文章后面微信索要清晰原图）：

![2022.09 知识图谱新增内容](assets/resource/av-knowledge-graph/av-graph-add-202209.png)

---

如果你也对音视频技术感兴趣，比如，符合下面的情况：

- 在校大学生 → 学习音视频开发
- iOS/Android 客户端开发 → 转入音视频领域
- 直播/短视频业务开发 → 深入音视频底层 SDK 开发
- 音视频 SDK 开发 → 提升技能，解决优化瓶颈

可以长按识别或扫描下面二维码，了解一下这个社群，根据自己的情况按需加入：

![识别二维码加入我们](assets/img/keyframe-zsxq.png)
_识别二维码加入我们_




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

