---
title: 音视频面试题集锦第 36 期
description: 持续更新的音视频面试题集锦。
author: Keyframe
date: 2025-02-17 05:09:28 +0800
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



下面是第 36 期面试题精选：

- 1、iOS 使用 CoreText 渲染字体的时候，如何计算字体所需要的高度？
- 2、iOS 中的相册是如何实现慢动作视频的存储？
- 3、Android 系统的字体渲染方案？
- 4、如何使用 ffprobe 快速获取视频每一帧的信息？


## 1、iOS 使用 CoreText 渲染字体的时候，如何计算字体所需要的高度？ 

主要是使用 CTFramesetterSuggestFrameSizeWithConstraints 计算文本的高度和宽度，记得要给CTFramesetterRef 设置相关的属性（行间距和自动换行）。

以下是使用 CoreText 绘制并计算文本高度的 Objective-C 代码示例:

```objc
NSString  *text = @"This\nis\nsome\nmulti-line\nsample\ntext."
UIFont    *uiFont = [UIFont fontWithName:@"Helvetica" size:17.0];
CTFontRef ctFont = CTFontCreateWithName((CFStringRef) uiFont.fontName, uiFont.pointSize, NULL);

// 设置行间距
CGFloat leading = uiFont.lineHeight - uiFont.ascender + uiFont.descender;
CTParagraphStyleSetting LineSpacing;
    
LineSpacing.spec = kCTParagraphStyleSpecifierLineSpacingAdjustment;
LineSpacing.value = &leading;
LineSpacing.valueSize = sizeof(CGFloat);
    
// 设置换行模式
CTParagraphStyleSetting lineBreakMode;
CTLineBreakMode lineBreak = kCTLineBreakByCharWrapping;
lineBreakMode.spec = kCTParagraphStyleSpecifierLineBreakMode;
lineBreakMode.value = &lineBreak;
lineBreakMode.valueSize = sizeof(CTLineBreakMode);

CTParagraphStyleSetting paragraphSettings[] = {lineBreakMode,LineSpacing};

CTParagraphStyleRef  paragraphStyle = CTParagraphStyleCreate(paragraphSettings, 2);
CFRange textRange = CFRangeMake(0, text.length);

CFMutableAttributedStringRef string = CFAttributedStringCreateMutable(kCFAllocatorDefault, text.length);
CFAttributedStringReplaceString(string, CFRangeMake(0, 0), (CFStringRef) text);

// 设置字体行间距和大小
CFAttributedStringSetAttribute(string, textRange, kCTFontAttributeName, ctFont);
CFAttributedStringSetAttribute(string, textRange, kCTParagraphStyleAttributeName, paragraphStyle);

CTFramesetterRef framesetter = CTFramesetterCreateWithAttributedString(string);
CFRange fitRange;

// 计算文本需要的高度
CGSize frameSize = CTFramesetterSuggestFrameSizeWithConstraints(framesetter, textRange, NULL, bounds, &fitRange);

CFRelease(framesetter);
CFRelease(string);
```

## 2、iOS 中的相册是如何实现慢动作视频的存储？

使用 iOS 中的相册类 PHAsset 获取慢动作视频的 AVAsset ，可以看到慢动作视频实际上是一个 AVComposition 类。

将同一个视频 URL 的各个部分，创建出速度不同的 AVSegment 后，再拼接成 AVComposition 进行储存。相比给一整段视频设置相同的慢速效果，这种实现使得各个segment速度从快到慢到快，所以猜测苹果是为了实现较为丝滑的变速效果所以使用AVComposition储存。

以下是从 PHAsset 中取出的 VideoTrack 中的 Segment 结构，可以看到 URL 路径都是同一个，但是设置了不同的速度。

```
<__NSArrayM 0x303fe0300>(
<AVCompositionTrackSegment: 0x30076dc00 timeRange [0.000,+2.397] from trackID 1 of asset file:///var/mobile/Media/DCIM/101APPLE/IMG_1592.MOV sourceTimeRange [0.000,+2.397]>,
<AVCompositionTrackSegment: 0x30076f720 timeRange [2.397,+0.015] from trackID 1 of asset file:///var/mobile/Media/DCIM/101APPLE/IMG_1592.MOV sourceTimeRange [2.397,+0.013]>,
<AVCompositionTrackSegment: 0x30076e5a0 timeRange [2.412,+0.045] from trackID 1 of asset file:///var/mobile/Media/DCIM/101APPLE/IMG_1592.MOV sourceTimeRange [2.410,+0.033]>,
<AVCompositionTrackSegment: 0x30076d8f0 timeRange [2.457,+0.083] from trackID 1 of asset file:///var/mobile/Media/DCIM/101APPLE/IMG_1592.MOV sourceTimeRange [2.443,+0.052]>,
<AVCompositionTrackSegment: 0x30076fb10 timeRange [2.540,+0.130] from trackID 1 of asset file:///var/mobile/Media/DCIM/101APPLE/IMG_1592.MOV sourceTimeRange [2.495,+0.065]>,
<AVCompositionTrackSegment: 0x30076e6f0 timeRange [2.670,+0.182] from trackID 1 of asset file:///var/mobile/Media/DCIM/101APPLE/IMG_1592.MOV sourceTimeRange [2.560,+0.068]>,
<AVCompositionTrackSegment: 0x30076fe20 timeRange [2.852,+40.662] from trackID 1 of asset file:///var/mobile/Media/DCIM/101APPLE/IMG_1592.MOV sourceTimeRange [2.628,+10.165]>,
<AVCompositionTrackSegment: 0x30076ea00 timeRange [43.513,+0.107] from trackID 1 of asset file:///var/mobile/Media/DCIM/101APPLE/IMG_1592.MOV sourceTimeRange [12.793,+0.040]>,
<AVCompositionTrackSegment: 0x30076ebc0 timeRange [43.620,+0.075] from trackID 1 of asset file:///var/mobile/Media/DCIM/101APPLE/IMG_1592.MOV sourceTimeRange [12.833,+0.038]>,
<AVCompositionTrackSegment: 0x30076d110 timeRange [43.695,+0.048] from trackID 1 of asset file:///var/mobile/Media/DCIM/101APPLE/IMG_1592.MOV sourceTimeRange [12.872,+0.030]>,
<AVCompositionTrackSegment: 0x30076eae0 timeRange [43.743,+0.027] from trackID 1 of asset file:///var/mobile/Media/DCIM/101APPLE/IMG_1592.MOV sourceTimeRange [12.902,+0.020]>,
<AVCompositionTrackSegment: 0x30076f9c0 timeRange [43.770,+0.008] from trackID 1 of asset file:///var/mobile/Media/DCIM/101APPLE/IMG_1592.MOV sourceTimeRange [12.922,+0.008]>,
<AVCompositionTrackSegment: 0x30076f100 timeRange [43.778,+0.078] from trackID 1 of asset file:///var/mobile/Media/DCIM/101APPLE/IMG_1592.MOV sourceTimeRange [12.930,+0.080]>,
<AVCompositionTrackSegment: 0x30076ff70 timeRange [43.857,+2.303] from trackID 1 of asset file:///var/mobile/Media/DCIM/101APPLE/IMG_1592.MOV sourceTimeRange [13.010,+2.303]>
)
```




## 3、Android 系统的字体渲染方案？

Android 中用于绘制文本的类是 android.graphics.Paint 。

Paint 可以设置颜色，字体大小，字体等属性，支持设置抗锯齿和文字对齐效果。k开发者还可以重用 Paint 对象，减少内存分配和垃圾回收的压力。

```
import android.content.Context;
import android.graphics.Bitmap;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;

public class TextToImage {

    public static Bitmap createImageFromText(Context context, String text, int textSize) {
        // 创建一个Bitmap对象
        Paint paint = new Paint();
        paint.setTextSize(textSize);
        paint.setColor(Color.BLACK);
        paint.setAntiAlias(true);

        // 计算文本的宽高
        float textWidth = paint.measureText(text);
        float textHeight = paint.getFontMetrics().bottom - paint.getFontMetrics().top;

        // 创建Bitmap，并指定宽高
        Bitmap bitmap = Bitmap.createBitmap((int) textWidth, (int) textHeight, Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(bitmap);

        // 设置背景颜色（可选）
        canvas.drawColor(Color.WHITE);

        // 绘制文本
        canvas.drawText(text, 0, textHeight - paint.getFontMetrics().bottom, paint);

        return bitmap;
    }
}
```

## 4、如何使用 ffprobe 快速获取视频每一帧信息？

```
ffprobe -show_packets -select_streams v -of xml input.mp4
```

- show_packets：此选项告诉 ffprobe 显示数据包信息。
- of xml： 设置输出格式为 XML。

输出的数据形式如下：

```
<packet codec_type="video" stream_index="0" pts="0" pts_time="0.000000" dts="-2002" dts_time="-0.066733" duration="1001" duration_time="0.033367" size="48064" pos="33134" 
flags="K__"/>
```

