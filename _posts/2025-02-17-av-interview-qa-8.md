---
title: 音视频面试题集锦第 8 期
description: 持续更新的音视频面试题集锦。
author: Keyframe
date: 2025-02-17 02:18:08 +0800
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

下面是 2023.10 月音视频面试题集锦的几条干货精选：

- **1、如何代码实现 PSNR 来评估编码质量？**
- **2、如何测试码率质量甜点？**
- **3、iOS 如何实现夜晚自动提示打开手电筒？**
- **4、Android Camera 如何优化视频录制的卡顿？**
- **5、Android Surface 解码如何支持带角度视频？**

## 1、如何代码实现 PSNR 来评估编码质量？

**1）PSNR 定义**

`PSNR`（`Peak Signal-to-Noise Ratio`，峰值信噪比）是一种衡量图像质量的指标，常用于评估压缩算法的效果。它通过比较原始图像与压缩/恢复后的图像之间的差异，来量化图像质量的损失程度。

**2）PSNR 计算公式**

`PSNR = 10 * log10((MAX^2) / MSE)`

其中，MAX 是图像像素值的最大可能取值（通常为255），`MSE` 是均方误差（`Mean Squared Error`），表示压缩/恢复图像与原始图像之间的平均像素差的平方。

**3）PSNR 计算实现**

```c
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

double psnr(unsigned char* original_img, unsigned char* compressed_img, int width, int height) {
    double mse = 0.0;
    for (int i = 0; i < height; i++) {
        for (int j = 0; j < width; j++) {
            int diff = original_img[i * width + j] - compressed_img[i * width + j];
            mse += diff * diff;
        }
    }
    mse /= (double)(width * height);
    double max_value = 255.0;
    double psnr = 10.0 * log10((max_value * max_value) / mse);
    return psnr;
}

int main() {
    // 读取原始图像和压缩/恢复后的图像
    FILE* original_file = fopen("original_image.raw", "rb");
    FILE* compressed_file = fopen("compressed_image.raw", "rb");
    if (original_file == NULL || compressed_file == NULL) {
        printf("无法打开图像文件\n");
        return 1;
    }

    int width = 512;  // 图像宽度
    int height = 512; // 图像高度

    unsigned char* original_img = (unsigned char*)malloc(width * height * sizeof(unsigned char));
    unsigned char* compressed_img = (unsigned char*)malloc(width * height * sizeof(unsigned char));
    fread(original_img, sizeof(unsigned char), width * height, original_file);
    fread(compressed_img, sizeof(unsigned char), width * height, compressed_file);

    fclose(original_file);
    fclose(compressed_file);

    // 计算PSNR
    double psnr_value = psnr(original_img, compressed_img, width, height);
    printf("PSNR: %.2f\n", psnr_value);

    // 释放内存
    free(original_img);
    free(compressed_img);

    return 0;
}
```

请注意，上述代码中的 `original_image.raw` 和 `compressed_image.raw` 是示例图像的文件名，你需要根据实际情况修改为你自己的图像文件名。此外，代码中假设图像是灰度图像，如果你的图像是彩色图像，需要进行相应的修改。


通过计算 `PSNR`，我们可以得到一个数值来评估压缩/恢复图像与原始图像之间的质量损失程度。`PSNR` 的数值越高，表示图像质量的损失越小。
	


## 2、如何测试码率质量甜点？

在视频领域，质量甜点指的是在既定的码率和屏幕大小下通过设定合理的分辨率和帧率来得到最佳视频主观质量体验。因为编码复杂度和编解码质量亦不是线性关系，两者之间也存在一个*质量甜点*。在音频领域也有类似的情况，针对具体的情况，我们可以测试手机的编码质量来选择指定分辨率、帧率时对应的码率甜点。

在这种测试中我们一般需要分场景进行，比如：

- 外景拍摄，低运动
- 外景拍摄，中等运动
- 外景拍摄，高运动
- 人像拍摄，低运动
- 人像拍摄，中等运动
- 人像拍摄，高运动

测试指标我们可以采用 `PSNR`、`SSIM`、`VMAF` 进行综合考量。

比如，我们可以测试 `iOS` 硬编，使用 `540P`，15 帧推流时，设置不同的码率（800kbps-1300kbps）分别测试各场景下的各指标值，找出 R-D（码率-失真）曲线拐点出现的区间，这就是我们要找的码率甜点。


## 3、iOS 如何实现夜晚自动提示打开手电筒？

当夜晚使用共享单车扫码时，应该都见过提示“打开手电筒”，在 `iOS` 中我们如何实现呢？主要基于图像环境光参数，参考如下代码。

`iOS` 视频采集设置 `AVCaptureVideoDataOutput AVCaptureVideoDataOutputSampleBufferDelegate`，通过获取 `CMSampleBufferRef` 每一帧视频数据环境参数进行判断即可。

```objc
- (void)captureOutput:(AVCaptureOutput *)output didOutputSampleBuffer:(CMSampleBufferRef)sampleBuffer fromConnection:(AVCaptureConnection *)connection {
	if(!sampleBuffer){
        return;
    }
    
    CFDictionaryRef metadataDict = CMCopyDictionaryOfAttachments(NULL,sampleBuffer, 	kCMAttachmentMode_ShouldPropagate);

	NSDictionary *metadata = [[NSMutableDictionary alloc] initWithDictionary:(__bridge 	NSDictionary*)metadataDict];

	CFRelease(metadataDict);

	NSDictionary *exifMetadata = [[metadata objectForKey:(NSString 	*)kCGImagePropertyExifDictionary] mutableCopy];

	float brightnessValue = [[exifMetadata objectForKey:(NSString 	*)kCGImagePropertyExifBrightnessValue] floatValue];
	
	// 根据brightnessValue的值来打开和关闭手电筒
    AVCaptureDevice *device = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
    
    if ((brightnessValue < 0)) {
    // 打开手电筒
	else if(brightnessValue > 0) {
	// 关闭手电筒
	}
}
```

## 4、Android Camera 如何优化视频录制的卡顿？

**1）视频录制流程**

- 打开 `Camera`。
- 创建 `SurfaceTextue` ，将 `Camera` 输出的数据渲染到 `SurfaceTextue`。
- `SurfaceTexture` 拿到的结果进行特效处理。
- 特效处理的结果异步分发给 `RenderView` 预览与 `MediaCodec` 编码。
- 编码后的结果进行 `Muxer` 合成 `MP4` 视频。

**2）视频录制流程优化**

- 相机、编码根据不同机型控制不同帧率、分辨率。
- 实现丢帧模块，将采集后的帧进入丢帧模块进行控制帧率，降低渲染以及编码性能。
- 采集、渲染、编码按照多个线程处理，每个模块发挥最优性能。
- 预览视图优先采用 `SurfaceView`，少量使用 `TextureView`，因为 `TextureView` 占用主线程渲染。
- 编码优先使用 `Surface` 异步编码。


## 5、Android Surface 解码如何支持带角度视频？

**1）直接解码到 Surface**

需要通过 `MediaFormat` 设置解码参，通过 `MediaFormat.KEY_ROTATION` 配置旋转角度，则可以正确显示。

**2）解码到 SurfaceTexture**

解码到 `SurfaceTexture`，通过 `MediaFormat.KEY_ROTATION` 配置旋转角度，但输出纹理提供接口获取旋转矩阵，`mSurfaceTexture.getSurfaceTexture().getTransformMatrix`，拿到旋转矩阵后通过 `FBO` 渲染调整为正确尺寸，这种模式好处可以将解码后数据经过自定义处理传递给编码层与渲染上屏。


---


**更多的音视频知识、面试题、技术方案干货可以进群来看：**

![微信扫码加入](assets/img/keyframe-zsxq.png)
_微信扫码加入_