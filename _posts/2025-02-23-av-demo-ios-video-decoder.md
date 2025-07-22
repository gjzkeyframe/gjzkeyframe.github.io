---
title: iOS AVDemo（12）：视频解码，代码开源并提供解析
description: 介绍 iOS 视频解码流程和原理，并提供 Demo 源码和解析。
author: Keyframe
date: 2025-02-23 09:08:08 +0800
categories: [音视频源码示例]
tags: [音视频源码示例, 音视频, iOS, 视频, 编解码]
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

iOS/Android 客户端开发同学如果想要开始学习音视频开发，最丝滑的方式是对[音视频基础概念知识](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2140155659944787969#wechat_redirect)有一定了解后，再借助 iOS/Android 平台的音视频能力上手去实践音视频的`采集 → 编码 → 封装 → 解封装 → 解码 → 渲染`过程，并借助[音视频实用工具](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2216997905264082945#wechat_redirect)来分析和理解对应的音视频数据。

在[音视频工程示例](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MjM5MTkxOTQyMQ==&action=getalbum&album_id=2273301900659851268#wechat_redirect)这个栏目，我们将通过拆解`采集 → 编码 → 封装 → 解封装 → 解码 → 渲染`流程并实现 Demo 来向大家介绍如何在 iOS/Android 平台上手音视频开发。

这里是第十二篇：**iOS 视频解码 Demo**。这个 Demo 里包含以下内容：

- 1）实现一个视频解封装模块；
- 2）实现一个视频解码模块；
- 3）串联视频解封装和解码模块，将解封装的 H.264/H.265 数据输入给解码模块进行解码，并存储解码后的 YUV 数据；
- 4）详尽的代码注释，帮你理解代码逻辑和原理。



在本文中，我们将详解一下 Demo 的具体实现和源码。读完本文内容相信就能帮你掌握相关知识。

>>不过，如果你的需求是：1）直接获得全部工程源码；2）想进一步咨询音视频技术问题；3）咨询音视频职业发展问题。可以根据自己的需要考虑是否加入『关键帧的音视频开发圈』，这是一个收费的社群服务，目前还有少量优惠券可用。
>>![长按识别二维码→加入我们](assets/img/keyframe-zsxq.png)
>>_长按识别二维码→加入我们_
 
 



## 1、视频解封装模块

视频解封装模块即 `KFMP4Demuxer`，复用了[《iOS 音频解封装 Demo》](https://mp.weixin.qq.com/s/fCZfIXriTXUPcI4d4te_ew)中介绍的 demuxer，这里就不再重复介绍了，其接口如下：

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


## 2、视频解码模块

接下来，我们来实现一个视频解码模块 `KFVideoDecoder`，在这里输入解封装后的编码数据，输出解码后的数据。


`KFVideoDecoder.h`

```objc
#import <Foundation/Foundation.h>
#import <CoreMedia/CoreMedia.h>

NS_ASSUME_NONNULL_BEGIN

@interface KFVideoDecoder : NSObject
@property (nonatomic, copy) void (^pixelBufferOutputCallBack)(CVPixelBufferRef pixelBuffer, CMTime ptsTime); // 视频解码数据回调。
@property (nonatomic, copy) void (^errorCallBack)(NSError *error); // 视频解码错误回调。
- (void)decodeSampleBuffer:(CMSampleBufferRef)sampleBuffer; // 解码。
- (void)flush; // 清空解码缓冲区。
- (void)flushWithCompleteHandler:(void (^)(void))completeHandler; // 清空解码缓冲区并回调完成。
@end

NS_ASSUME_NONNULL_END
```

上面是 `KFVideoDecoder` 接口的设计，主要是有视频解码`数据回调`和`错误回调`的接口，另外就是`解码`和`清空解码缓冲区`的接口。

在上面的`解码`接口中，我们使用的是依然 [CMSampleBufferRef](https://developer.apple.com/documentation/coremedia/cmsamplebufferref/ "CMSampleBufferRef") 作为参数。而`解码器数据回调`接口则使用 [CVPixelBufferRef](https://developer.apple.com/documentation/corevideo/cvpixelbufferref/ "CVPixelBufferRef") 作为返回值类型。

在`解码`接口中，我们通过 `CMSampleBufferRef` 打包的是解封装后得到的 H.264/H.265 编码数据。

在`解码器数据回调`接口中，我们通过 `CVPixelBufferRef` 打包的是对 H.264/H.265 解码后得到的 YUV 数据。


`KFVideoDecoder.m`

```objc
#import "KFVideoDecoder.h"
#import <VideoToolBox/VideoToolBox.h>

#define KFDecoderRetrySessionMaxCount 5
#define KFDecoderDecodeFrameFailedMaxCount 20

@interface KFVideoDecoderInputPacket : NSObject
@property (nonatomic, assign) CMSampleBufferRef sampleBuffer;
@end

@implementation KFVideoDecoderInputPacket
@end

@interface KFVideoDecoder ()
@property (nonatomic, assign) VTDecompressionSessionRef decoderSession; // 视频解码器实例。
@property (nonatomic, strong) dispatch_queue_t decoderQueue;
@property (nonatomic, strong) dispatch_semaphore_t semaphore;
@property (nonatomic, assign) NSInteger retrySessionCount; // 解码器重试次数。
@property (nonatomic, assign) NSInteger decodeFrameFailedCount; // 解码失败次数。
@property (nonatomic, strong) NSMutableArray *gopList;
@property (nonatomic, assign) NSInteger inputCount;
@property (nonatomic, assign) NSInteger outputCount;
@end

@implementation KFVideoDecoder
#pragma mark - LifeCycle
- (instancetype)init {
    self = [super init];
    if (self) {
        _decoderQueue = dispatch_queue_create("com.KeyFrameKit.videoDecoder", DISPATCH_QUEUE_SERIAL);
        _semaphore = dispatch_semaphore_create(1);
        _gopList = [NSMutableArray new];
    }
    
    return self;
}

- (void)dealloc {
    // 清理解码器。
    dispatch_semaphore_wait(_semaphore, DISPATCH_TIME_FOREVER);
    [self _releaseDecompressionSession];
    [self _clearCompressQueue];
    dispatch_semaphore_signal(_semaphore);
}

#pragma mark - Public Method
- (void)decodeSampleBuffer:(CMSampleBufferRef)sampleBuffer {
    if (!sampleBuffer || self.retrySessionCount >= KFDecoderRetrySessionMaxCount || self.decodeFrameFailedCount >= KFDecoderDecodeFrameFailedMaxCount) {
        return;
    }
    
    __weak typeof(self) weakSelf = self;
    CFRetain(sampleBuffer);
    dispatch_async(_decoderQueue, ^{
        dispatch_semaphore_wait(weakSelf.semaphore, DISPATCH_TIME_FOREVER);
        
        // 1、如果还未创建解码器实例，或者解码器需要重建，则创建解码器。
        OSStatus setupStatus = noErr;
        if (!weakSelf.decoderSession) {
            // 支持重试，记录重试次数。
            setupStatus = [weakSelf _setupDecompressionSession:CMSampleBufferGetFormatDescription(sampleBuffer)];
            weakSelf.retrySessionCount = setupStatus == noErr ? 0 : (weakSelf.retrySessionCount + 1);
            if (setupStatus != noErr) {
                [weakSelf _releaseDecompressionSession];
            }
        }
        
        if (!weakSelf.decoderSession) {
            // 重试超过 KFDecoderRetrySessionMaxCount 次仍然失败则认为创建失败，报错。
            CFRelease(sampleBuffer);
            dispatch_semaphore_signal(weakSelf.semaphore);
            if (weakSelf.retrySessionCount >= KFDecoderRetrySessionMaxCount && weakSelf.errorCallBack) {
                dispatch_async(dispatch_get_main_queue(), ^{
                    weakSelf.errorCallBack([NSError errorWithDomain:NSStringFromClass([KFVideoDecoder class]) code:setupStatus userInfo:nil]);
                });
            }
            return;
        }
        
        // 2、对 sampleBuffer 进行解码。
        VTDecodeFrameFlags flags = kVTDecodeFrame_EnableAsynchronousDecompression;
        VTDecodeInfoFlags flagOut = 0;
        // 解码当前 sampleBuffer。
        OSStatus decodeStatus = VTDecompressionSessionDecodeFrame(weakSelf.decoderSession, sampleBuffer, flags, NULL, &flagOut);
        if (decodeStatus == kVTInvalidSessionErr) {
            // 解码当前帧失败，进行重建解码器重试。
            [weakSelf _releaseDecompressionSession];
            setupStatus = [weakSelf _setupDecompressionSession:CMSampleBufferGetFormatDescription(sampleBuffer)];
            // 记录重建解码器次数。
            weakSelf.retrySessionCount = setupStatus == noErr ? 0 : (weakSelf.retrySessionCount + 1);
            if (setupStatus == noErr) {
                // 重建解码器成功后，要从当前 GOP 开始的 I 帧解码。所以这里先解码缓存的当前 GOP 的前序帧。
                flags = kVTDecodeFrame_DoNotOutputFrame;
                for (KFVideoDecoderInputPacket *packet in weakSelf.gopList) {
                    VTDecompressionSessionDecodeFrame(weakSelf.decoderSession, packet.sampleBuffer, flags, NULL, &flagOut);
                }
                // 解码当前帧。
                flags = kVTDecodeFrame_EnableAsynchronousDecompression;
                decodeStatus = VTDecompressionSessionDecodeFrame(weakSelf.decoderSession, sampleBuffer, flags, NULL, &flagOut);
            } else {
                // 重建解码器失败。
                [weakSelf _releaseDecompressionSession];
            }
        } else if (decodeStatus != noErr) {
            // 解码当前帧失败。
            NSLog(@"KFVideoDecoder decode error:%d", decodeStatus);
        }
        
        // 统计解码入帧数。
        weakSelf.inputCount++;
        
        // 遇到新的 I 帧后，清空上一个 GOP 序列缓存，开始进行下一个 GOP 的缓存。
        if ([weakSelf _isKeyFrame:sampleBuffer]) {
            [weakSelf _clearCompressQueue];
        }
        KFVideoDecoderInputPacket *packet = [KFVideoDecoderInputPacket new];
        packet.sampleBuffer = sampleBuffer;
        [weakSelf.gopList addObject:packet];
        
        // 记录解码失败次数。
        weakSelf.decodeFrameFailedCount = decodeStatus == noErr ? 0 : (weakSelf.decodeFrameFailedCount + 1);
        
        dispatch_semaphore_signal(weakSelf.semaphore);
        
        // 解码失败次数超过 KFDecoderDecodeFrameFailedMaxCount 次，报错。
        if (weakSelf.decodeFrameFailedCount >= KFDecoderDecodeFrameFailedMaxCount && weakSelf.errorCallBack) {
            dispatch_async(dispatch_get_main_queue(), ^{
                weakSelf.errorCallBack([NSError errorWithDomain:NSStringFromClass([KFVideoDecoder class]) code:decodeStatus userInfo:nil]);
            });
        }
    });
}

- (void)flush {
    // 清空解码缓冲区。
    __weak typeof(self) weakSelf = self;
    dispatch_async(_decoderQueue, ^{
        dispatch_semaphore_wait(weakSelf.semaphore, DISPATCH_TIME_FOREVER);
        [weakSelf _flush];
        dispatch_semaphore_signal(weakSelf.semaphore);
    });
}

- (void)flushWithCompleteHandler:(void (^)(void))completeHandler {
    // 清空解码缓冲区并回调完成。
    __weak typeof(self) weakSelf = self;
    dispatch_async(self.decoderQueue, ^{
        dispatch_semaphore_wait(weakSelf.semaphore, DISPATCH_TIME_FOREVER);
        [weakSelf _flush];
        dispatch_semaphore_signal(weakSelf.semaphore);
        if (completeHandler) {
            completeHandler();
        }
    });
}

#pragma mark - Private Method
- (OSStatus)_setupDecompressionSession:(CMFormatDescriptionRef)videoDescription {
    if (_decoderSession) {
        return noErr;
    }
        
    // 1、设置颜色格式。
    NSDictionary *attrs = @{(NSString *) kCVPixelBufferPixelFormatTypeKey: @(kCVPixelFormatType_420YpCbCr8BiPlanarFullRange)};
    
    //  2、设置解码回调。
    VTDecompressionOutputCallbackRecord callBackRecord;
    callBackRecord.decompressionOutputCallback = decompressionOutputCallback;
    callBackRecord.decompressionOutputRefCon = (__bridge void *) self;
    
    // 3、创建解码器实例。
    OSStatus status = VTDecompressionSessionCreate(kCFAllocatorDefault,
                                                   videoDescription,
                                                   NULL,
                                                   (__bridge void *) attrs,
                                                   &callBackRecord,
                                                   &_decoderSession);
    
    return status;
}

- (void)_releaseDecompressionSession {
    // 清理解码器。
    if (_decoderSession) {
        VTDecompressionSessionWaitForAsynchronousFrames(_decoderSession);
        VTDecompressionSessionInvalidate(_decoderSession);
        _decoderSession = NULL;
    }
}

- (void)_flush {
    // 清理解码器缓冲。
    if (_decoderSession) {
        VTDecompressionSessionFinishDelayedFrames(_decoderSession);
        VTDecompressionSessionWaitForAsynchronousFrames(_decoderSession);
    }
}

- (void)_clearCompressQueue {
    // 清空当前 GOP 缓冲区。
    for (KFVideoDecoderInputPacket *packet in self.gopList) {
        if (packet.sampleBuffer) {
            CFRelease(packet.sampleBuffer);
        }
    }
    [self.gopList removeAllObjects];
}

- (BOOL)_isKeyFrame:(CMSampleBufferRef)sampleBuffer {
    CFArrayRef array = CMSampleBufferGetSampleAttachmentsArray(sampleBuffer, true);
    if (!array) {
        return NO;
    }
    
    CFDictionaryRef dic = (CFDictionaryRef)CFArrayGetValueAtIndex(array, 0);
    if (!dic) {
        return NO;
    }
    
    // 是否关键帧。
    BOOL keyframe = !CFDictionaryContainsKey(dic, kCMSampleAttachmentKey_NotSync);
    
    return keyframe;
}

#pragma mark - DecoderOutputCallback
static void decompressionOutputCallback( void *decompressionOutputRefCon, void *sourceFrameRefCon, OSStatus status, VTDecodeInfoFlags infoFlags, CVImageBufferRef imageBuffer, CMTime presentationTimeStamp, CMTime presentationDuration ) {
    if (status != noErr) {
        return;
    }
    
    if (infoFlags & kVTDecodeInfo_FrameDropped) {
        NSLog(@"KFVideoDecoder drop frame");
        return;
    }
    
    // 向外层回调解码数据。
    KFVideoDecoder *videoDecoder = (__bridge KFVideoDecoder *) decompressionOutputRefCon;
    if (videoDecoder && imageBuffer && videoDecoder.pixelBufferOutputCallBack) {
        videoDecoder.pixelBufferOutputCallBack(imageBuffer, presentationTimeStamp);
        videoDecoder.outputCount++; // 统计解码出帧数。
    }
}

@end
```


上面是 `KFVideoDecoder` 的实现，从代码上可以看到主要有这几个部分：

- 1）创建视频解码实例。
	- 在 `-_setupDecompressionSession:` 方法中实现。在 `-decodeSampleBuffer:` 中检查到还未创建解码器实例，或者解码器需要重建，则创建解码器。
- 2）实现视频解码逻辑。
	- 在 `-decodeSampleBuffer:` 中实现。支持出错重建解码器和 GOP 解码缓存。
- 3）实现清空解码缓冲区逻辑。
	- 在 `-flush` 和 `-flushWithCompleteHandler:` 中分别实现同步和异步带回调的方式。
- 4）捕捉视频解码过程中的错误，抛给 KFVideoDecoder 的对外错误回调接口。
	- 在 `-decodeSampleBuffer:` 中捕捉错误。
- 5）清理视频解码器实例、解码缓存。
	- 在 `-dealloc` 中实现。

更具体细节见上述代码及其注释。


## 3、解封装和解码 MP4 文件中的视频部分存储为 YUV 文件

我们在一个 ViewController 中来实现视频解封装及解码逻辑，并将解码后的数据存储为 YUV 文件。

`KFVideoDecoderViewController.m`


```objc
#import "KFVideoDecoderViewController.h"
#import "KFMP4Demuxer.h"
#import "KFVideoDecoder.h"

#define KFDecompressionMaxCount 5

@interface KFVideoDecoderFrame : NSObject
@property (nonatomic, strong) NSData *data;
@property (nonatomic, assign) Float64 time;
@end

@implementation KFVideoDecoderFrame
@end

@interface KFVideoDecoderViewController ()
@property (nonatomic, strong) KFDemuxerConfig *demuxerConfig;
@property (nonatomic, strong) KFMP4Demuxer *demuxer;
@property (nonatomic, strong) KFVideoDecoder *decoder;
@property (nonatomic, strong) NSMutableArray *yuvDataArray;
@property (nonatomic, strong) NSFileHandle *fileHandle;
@end

@implementation KFVideoDecoderViewController
#pragma mark - Property
- (KFDemuxerConfig *)demuxerConfig {
    if (!_demuxerConfig) {
        _demuxerConfig = [[KFDemuxerConfig alloc] init];
        _demuxerConfig.demuxerType = KFMediaVideo;
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

- (KFVideoDecoder *)decoder {
    if (!_decoder) {
        __weak typeof(self) weakSelf = self;
        _decoder = [[KFVideoDecoder alloc] init];
        _decoder.errorCallBack = ^(NSError *error) {
            NSLog(@"KFVideoDecoder error %zi %@",error.code,error.localizedDescription);
        };
        _decoder.pixelBufferOutputCallBack = ^(CVPixelBufferRef pixelBuffer, CMTime ptsTime) {
            // 解码数据回调。存储解码后的数据为 YUV 文件。
            [weakSelf savePixelBuffer:pixelBuffer time:ptsTime];
        };
    }
    
    return _decoder;
}

- (NSMutableArray *)yuvDataArray {
    if (!_yuvDataArray) {
        _yuvDataArray = [[NSMutableArray alloc] init];
    }
    
    return _yuvDataArray;
}

- (NSFileHandle *)fileHandle {
    if (!_fileHandle) {
        NSString *videoPath = [[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject] stringByAppendingPathComponent:@"output.yuv"];
        [[NSFileManager defaultManager] removeItemAtPath:videoPath error:nil];
        [[NSFileManager defaultManager] createFileAtPath:videoPath contents:nil attributes:nil];
        _fileHandle = [NSFileHandle fileHandleForWritingAtPath:videoPath];
    }

    return _fileHandle;
}

#pragma mark - Lifecycle
- (void)viewDidLoad {
    [super viewDidLoad];

    self.view.backgroundColor = [UIColor whiteColor];
    self.title = @"Video Decoder";
    UIBarButtonItem *startBarButton = [[UIBarButtonItem alloc] initWithTitle:@"Start" style:UIBarButtonItemStylePlain target:self action:@selector(start)];
    self.navigationItem.rightBarButtonItems = @[startBarButton];
    
    // 完成音频解码后，可以将 App Document 文件夹下面的 output.yuv 文件拷贝到电脑上，使用 ffplay 播放：
    // ffplay -f rawvideo -pix_fmt nv12 -video_size 1280x720 -i output.yuv

}

#pragma mark - Action
- (void)start {
    __weak typeof(self) weakSelf = self;
    NSLog(@"KFMP4Demuxer start");
    [self.demuxer startReading:^(BOOL success, NSError * _Nonnull error) {
        if (success) {
            // Demuxer 启动成功后，就可以从它里面获取解封装后的数据了。
            [weakSelf fetchAndDecodeDemuxedData];
        } else {
            NSLog(@"KFMP4Demuxer error: %zi %@",error.code,error.localizedDescription);
        }
    }];
}

#pragma mark - Private Method
- (void)fetchAndDecodeDemuxedData {
    // 异步地从 Demuxer 获取解封装后的 H.264/H.265 编码数据，送给解码器进行解码。
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        while (self.demuxer.hasVideoSampleBuffer) {
            CMSampleBufferRef videoBuffer = [self.demuxer copyNextVideoSampleBuffer];
            if (videoBuffer) {
                [self.decoder decodeSampleBuffer:videoBuffer];
                CFRelease(videoBuffer);
            }
        }
        [self.decoder flushWithCompleteHandler:^{
            for (NSInteger index = 0; index < self.yuvDataArray.count; index++) {
                KFVideoDecoderFrame *frame = self.yuvDataArray[index];
                [self.fileHandle writeData:frame.data];
            }
            [self.yuvDataArray removeAllObjects];
        }];
        if (self.demuxer.demuxerStatus == KFMP4DemuxerStatusCompleted) {
            NSLog(@"KFMP4Demuxer complete");
        }
    });
}

- (void)savePixelBuffer:(CVPixelBufferRef)pixelBuffer time:(CMTime)time{
    if (!pixelBuffer) {
        return;
    }
    
    // 取出 YUV 数据，按照 NV12 的 YUV 格式存储。
    CVPixelBufferLockBaseAddress(pixelBuffer, 0);
    NSMutableData *mutableData = [NSMutableData new];
    for (size_t index = 0; index < CVPixelBufferGetPlaneCount(pixelBuffer); index++) {
        size_t bytesPerRowOfPlane = CVPixelBufferGetBytesPerRowOfPlane(pixelBuffer, index);
        size_t height = CVPixelBufferGetHeightOfPlane(pixelBuffer, index);
        void *data = CVPixelBufferGetBaseAddressOfPlane(pixelBuffer, index);
        [mutableData appendBytes:data length:bytesPerRowOfPlane * height];
    }
    KFVideoDecoderFrame *newFrame = [KFVideoDecoderFrame new];
    newFrame.data = mutableData;
    newFrame.time = CMTimeGetSeconds(time);
    
    [self.yuvDataArray addObject:newFrame];
    CVPixelBufferUnlockBaseAddress(pixelBuffer, 0);
    
    // 以下排序性能太差，仅用于 Demo。
    if (self.yuvDataArray.count > KFDecompressionMaxCount) {
        NSArray *sortedArray = [self.yuvDataArray sortedArrayUsingComparator:^NSComparisonResult(id a, id b) {
            Float64 first = [(KFVideoDecoderFrame *) a time];
            Float64 second = [(KFVideoDecoderFrame *) b time];
            return first >= second;
        }];
        self.yuvDataArray = [[NSMutableArray alloc] initWithArray:sortedArray];
        KFVideoDecoderFrame *firstFrame = [self.yuvDataArray firstObject];
        [self.fileHandle writeData:firstFrame.data];
        [self.yuvDataArray removeObjectAtIndex:0];
    }
}

@end
```

上面是 `KFVideoDecoderViewController` 的实现，其中主要包含这几个部分：

- 1）通过启动视频解封装来驱动整个解封装和解码流程。
	- 在 `-start` 中实现开始动作。
- 2）在解封装模块 `KFMP4Demuxer` 启动成功后，开始读取解封装数据并启动解码。
	- 在 `-startReading:` 方法的回调中实现。
- 3）将解封装后的视频数据送给解码器解码。
	- 在 `-fetchAndDecodeDemuxedData` 方法中实现。
- 4）在解码模块 `KFVideoDecoder` 的数据回调中获取解码后的 YUV 数据存储为文件。
	- 在 `KFVideoDecoder` 的 `sampleBufferOutputCallBack` → `-savePixelBuffer:time:` 中实现。
	- 这里按照 NV12 的 YUV 格式存储。




## 4、用工具播放 YUV 文件

完成 Demo 后，可以将 App Document 文件夹下面的 `output.yuv` 文件拷贝到电脑上，使用 `ffplay` 播放来验证一下效果是否符合预期：

```c
$ ffplay -f rawvideo -pix_fmt nv12 -video_size 1280x720 -i output.yuv
```

注意这里的参数要对齐在工程中存储的 YUV 格式，我们 Demo 中的视频尺寸是 `1280x720`，我们是用 `NV12` 格式存储的 YUV。

关于播放 YUV 文件的工具，可以参考[《FFmpeg 工具》第 2 节 ffplay 命令行工具](https://mp.weixin.qq.com/s/Rl7fxOP-YH37mQEvGxhfUA)和[《可视化音视频分析工具》第 1.2 节 YUVToolkit 或 1.3 节 YUVView](https://mp.weixin.qq.com/s/jCYih3qgEIUctuWxn0aTGQ)。









