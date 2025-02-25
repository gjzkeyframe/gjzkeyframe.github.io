---
title: iOS AVDemo（11）：视频转封装，代码开源并提供解析
description: 介绍 iOS 视频转封装流程和原理，并提供 Demo 源码和解析。
author: Keyframe
date: 2025-02-23 08:58:08 +0800
categories: [音视频源码示例]
tags: [音视频源码示例, 音视频, iOS, 视频, 视频封装]
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

iOS/Android 客户端开发同学如果想要开始学习音视频开发，最丝滑的方式是对[音视频基础概念知识](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2140155659944787969#wechat_redirect)有一定了解后，再借助 iOS/Android 平台的音视频能力上手去实践音视频的`采集 → 编码 → 封装 → 解封装 → 解码 → 渲染`过程，并借助[音视频实用工具](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2216997905264082945#wechat_redirect)来分析和理解对应的音视频数据。

在[音视频工程示例](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2273301900659851268#wechat_redirect)这个栏目，我们将通过拆解`采集 → 编码 → 封装 → 解封装 → 解码 → 渲染`流程并实现 Demo 来向大家介绍如何在 iOS/Android 平台上手音视频开发。

这里是第十一篇：**iOS 音视频转封装 Demo**。这个 Demo 里包含以下内容：

- 1）实现一个音视频解封装模块；
- 2）实现一个音视频封装模块；
- 3）实现对 MP4 文件中音视频的解封装逻辑，将解封装后的音视频编码数据重新封装存储为一个新的 MP4 文件；
- 4）详尽的代码注释，帮你理解代码逻辑和原理。



在本文中，我们将详解一下 Demo 的具体实现和源码。读完本文内容相信就能帮你掌握相关知识。

>>不过，如果你的需求是：1）直接获得全部工程源码；2）想进一步咨询音视频技术问题；3）咨询音视频职业发展问题。可以根据自己的需要考虑是否加入『关键帧的音视频开发圈』，这是一个收费的社群服务，目前还有少量优惠券可用。
>>![长按识别二维码→加入我们](assets/img/keyframe-zsxq.png)
>>_长按识别二维码→加入我们_
 
 


## 1、音视频解封装模块

视频编码模块即 `KFMP4Demuxer`，复用了[《iOS 音频解封装 Demo》](https://mp.weixin.qq.com/s/fCZfIXriTXUPcI4d4te_ew)中介绍的 demuxer，这里就不再重复介绍了，其接口如下：

`KFMP4Demuxer.h`


```objc
#import <Foundation/Foundation.h>
#import <CoreMedia/CoreMedia.h>
#import "KFDemuxerConfig.h"

NS_ASSUME_NONNULL_BEGIN

typedef NS_ENUM(NSInteger, KFMP4DemuxerStatus) {
    KFMP4DemuxerStatusUnknown = 0,
    KFMP4DemuxerStatusRunning = 1,
    KFMP4DemuxerStatusFailed = 2,
    KFMP4DemuxerStatusCompleted = 3,
    KFMP4DemuxerStatusCancelled = 4,
};

@interface KFMP4Demuxer : NSObject
+ (instancetype)new NS_UNAVAILABLE;
- (instancetype)init NS_UNAVAILABLE;
- (instancetype)initWithConfig:(KFDemuxerConfig *)config;

@property (nonatomic, strong, readonly) KFDemuxerConfig *config;
@property (nonatomic, copy) void (^errorCallBack)(NSError *error);
@property (nonatomic, assign, readonly) BOOL hasAudioTrack; // 是否包含音频数据。
@property (nonatomic, assign, readonly) BOOL hasVideoTrack; // 是否包含视频数据。
@property (nonatomic, assign, readonly) CGSize videoSize; // 视频大小。
@property (nonatomic, assign, readonly) CMTime duration; // 媒体时长。
@property (nonatomic, assign, readonly) CMVideoCodecType codecType; // 编码类型。
@property (nonatomic, assign, readonly) KFMP4DemuxerStatus demuxerStatus; // 解封装器状态。
@property (nonatomic, assign, readonly) BOOL audioEOF; // 是否音频结束。
@property (nonatomic, assign, readonly) BOOL videoEOF; // 是否视频结束。
@property (nonatomic, assign, readonly) CGAffineTransform preferredTransform; // 图像的变换信息。比如：视频图像旋转。

- (void)startReading:(void (^)(BOOL success, NSError *error))completeHandler; // 开始读取数据解封装。
- (void)cancelReading; // 取消读取。

- (BOOL)hasAudioSampleBuffer; // 是否还有音频数据。
- (CMSampleBufferRef)copyNextAudioSampleBuffer CF_RETURNS_RETAINED; // 拷贝下一份音频采样。

- (BOOL)hasVideoSampleBuffer; // 是否还有视频数据。
- (CMSampleBufferRef)copyNextVideoSampleBuffer CF_RETURNS_RETAINED; // 拷贝下一份视频采样。
@end

NS_ASSUME_NONNULL_END
```








## 2、音视频封装模块

视频编码模块即 `KFMP4Muxer`，复用了[《iOS 音频封装 Demo》](https://mp.weixin.qq.com/s/R86qnQAi2njr6k7tFvTF-w)中介绍的 muxer，这里就不再重复介绍了，其接口如下：

`KFMP4Muxer.h`

```objc
#import <Foundation/Foundation.h>
#import <CoreMedia/CoreMedia.h>
#import "KFMuxerConfig.h"

NS_ASSUME_NONNULL_BEGIN

@interface KFMP4Muxer : NSObject
+ (instancetype)new NS_UNAVAILABLE;
- (instancetype)init NS_UNAVAILABLE;
- (instancetype)initWithConfig:(KFMuxerConfig *)config;

@property (nonatomic, strong, readonly) KFMuxerConfig *config;
@property (nonatomic, copy) void (^errorCallBack)(NSError *error); // 封装错误回调。

- (void)startWriting; // 开始封装写入数据。
- (void)cancelWriting; // 取消封装写入数据。
- (void)appendSampleBuffer:(CMSampleBufferRef)sampleBuffer; // 添加封装数据。
- (void)stopWriting:(void (^)(BOOL success, NSError *error))completeHandler; // 停止封装写入数据。
@end

NS_ASSUME_NONNULL_END
```






## 3、音视频重封装逻辑


我们还是在一个 ViewController 中来实现对 MP4 文件中音视频的解封装逻辑，然后将解封装后的音视频编码数据重新封装存储为一个新的 MP4 文件。

`KFRemuxerViewController.m`

```objc
#import "KFRemuxerViewController.h"
#import "KFMP4Demuxer.h"
#import "KFMP4Muxer.h"

@interface KFRemuxerViewController ()
@property (nonatomic, strong) KFDemuxerConfig *demuxerConfig;
@property (nonatomic, strong) KFMP4Demuxer *demuxer;
@property (nonatomic, strong) KFMuxerConfig *muxerConfig;
@property (nonatomic, strong) KFMP4Muxer *muxer;
@end

@implementation KFRemuxerViewController
#pragma mark - Property
- (KFDemuxerConfig *)demuxerConfig {
    if (!_demuxerConfig) {
        _demuxerConfig = [[KFDemuxerConfig alloc] init];
        _demuxerConfig.demuxerType = KFMediaAV;
        NSString *videoPath = [[NSBundle mainBundle] pathForResource:@"input" ofType:@"mp4"];
        _demuxerConfig.asset = [AVAsset assetWithURL:[NSURL fileURLWithPath:videoPath]];
    }
    
    return _demuxerConfig;
}

- (KFMP4Demuxer *)demuxer {
    if (!_demuxer) {
        _demuxer = [[KFMP4Demuxer alloc] initWithConfig:self.demuxerConfig];
        _demuxer.errorCallBack = ^(NSError *error) {
            NSLog(@"KFMP4Demuxer error:%zi %@", error.code, error.localizedDescription);
        };
    }
    
    return _demuxer;
}

- (KFMuxerConfig *)muxerConfig {
    if (!_muxerConfig) {
        _muxerConfig = [[KFMuxerConfig alloc] init];
        NSString *videoPath = [[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject] stringByAppendingPathComponent:@"output.mp4"];
        NSLog(@"MP4 file path: %@", videoPath);
        [[NSFileManager defaultManager] removeItemAtPath:videoPath error:nil];
        _muxerConfig.outputURL = [NSURL fileURLWithPath:videoPath];
        _muxerConfig.muxerType = KFMediaAV;
    }
    
    return _muxerConfig;
}

- (KFMP4Muxer *)muxer {
    if (!_muxer) {
        _muxer = [[KFMP4Muxer alloc] initWithConfig:self.muxerConfig];
    }
    
    return _muxer;
}

#pragma mark - Lifecycle
- (void)viewDidLoad {
    [super viewDidLoad];

    self.view.backgroundColor = [UIColor whiteColor];
    self.title = @"AV Remuxer";
    UIBarButtonItem *startBarButton = [[UIBarButtonItem alloc] initWithTitle:@"Start" style:UIBarButtonItemStylePlain target:self action:@selector(start)];
    self.navigationItem.rightBarButtonItems = @[startBarButton];
}

#pragma mark - Action
- (void)start {
    __weak typeof(self) weakSelf = self;
    NSLog(@"KFMP4Demuxer start");
    [self.demuxer startReading:^(BOOL success, NSError * _Nonnull error) {
        if (success) {
            // Demuxer 启动成功后，就可以从它里面获取解封装后的数据了。
            [weakSelf fetchAndRemuxData];
        }else{
            NSLog(@"KFDemuxer error: %zi %@", error.code, error.localizedDescription);
        }
    }];
}

#pragma mark - Utility
- (void)fetchAndRemuxData {
    // 异步地从 Demuxer 获取解封装后的 H.264/H.265 编码数据，再重新封装。
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        [self.muxer startWriting];
        while (self.demuxer.hasVideoSampleBuffer || self.demuxer.hasAudioSampleBuffer) {
            CMSampleBufferRef videoBuffer = [self.demuxer copyNextVideoSampleBuffer];
            if (videoBuffer) {
                [self.muxer appendSampleBuffer:videoBuffer];
                CFRelease(videoBuffer);
            }
            
            CMSampleBufferRef audioBuffer = [self.demuxer copyNextAudioSampleBuffer];
            if (audioBuffer) {
                [self.muxer appendSampleBuffer:audioBuffer];
                CFRelease(audioBuffer);
            }
        }
        if (self.demuxer.demuxerStatus == KFMP4DemuxerStatusCompleted) {
            NSLog(@"KFMP4Demuxer complete");
            [self.muxer stopWriting:^(BOOL success, NSError * _Nonnull error) {
                NSLog(@"KFMP4Muxer complete:%d", success);
            }];
        }
    });
}

@end
```

上面是 `KFRemuxerViewController` 的实现，其中主要包含这几个部分：

- 1）设置好待解封装的资源。
	- 在 `-demuxerConfig` 中实现，我们这里是一个 MP4 文件。
- 2）启动解封装器。
	- 在 `-start` 中实现。
- 3）在解封装器启动成功后，启动封装器。
	- 在 `-fetchAndRemuxData` 中启动。
- 4）读取解封装后的音视频编码数据并送给封装器进行重新封装。
	- 在 `-fetchAndRemuxData` 中实现。


## 4、用工具播放 MP4 文件


完成 Demo 后，可以将 App Document 文件夹下面的 `output.mp4` 文件拷贝到电脑上，使用 `ffplay` 播放来验证一下效果是否符合预期：

```c
$ ffplay -i output.mp4
```

关于播放 MP4 文件的工具，可以参考[《FFmpeg 工具》第 2 节 ffplay 命令行工具](https://mp.weixin.qq.com/s/Rl7fxOP-YH37mQEvGxhfUA)和[《可视化音视频分析工具》第 3.5 节 VLC 播放器](https://mp.weixin.qq.com/s/jCYih3qgEIUctuWxn0aTGQ)。


我们还可以用[《可视化音视频分析工具》第 3.1 节 MP4Box.js
](https://mp.weixin.qq.com/s/jCYih3qgEIUctuWxn0aTGQ) 等工具来查看它的格式。




