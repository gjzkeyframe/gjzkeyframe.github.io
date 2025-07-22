---
title: AVCC/HVCC 与 AnnexB 码流格式相互转换
description: 介绍 AVCC/HVCC 与 AnnexB 码流格式相互转换的技术方案和实现源码。
author: Keyframe
date: 2025-02-25 07:08:08 +0800
categories: [音视频实战经验]
tags: [音视频实战经验, 音视频, 视频编辑, AnnexB, AVCC/HVCC, 编解码]
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


H.264 的码流中用于解码的关键信息包括 SPS、PPS，H.265 码流中则包括 VPS、SPS 和 PPS。H.264 码流有 AVCC 和 AnnexB 两种格式，H.265 码流则对应的有 HVCC 和 AnnexB 两种格式。通常工程实践中对 MP4 进行解码时默认是使用 AVCC、HVCC 码流格式，但由于 Android 平台的解码器通常仅支持 AnnexB 格式，这时候就需要对码流格式做一下转换。我们这篇文章里就来介绍一下如何用代码实现 AVCC/HVCC 与 AnnexB 码流格式的相互转换。


## 1、H.264/H.265 NALU 数据格式定义

**1）H.264 AVCDecoderConfigurationRecord 数据结构**

`AVCDecoderConfigurationRecord` 即 AVC Sequence Header，是将 SPS、PPS 封装为 `ExtraData` 的一种数据结构，封装好的 `ExtraData` 可以放置在 FLV 与 MP4 文件头中，播放器获取后用于初始化解码器。

```c
aligned(8) class AVCDecoderConfigurationRecord {
    unsigned int(8) configurationVersion = 1;
    unsigned int(8) AVCProfileIndication;
    unsigned int(8) profile_compatibility;
    unsigned int(8) AVCLevelIndication;
    bit(6) reserved = ‘111111’b;
    // lengthSizeMinusOne + 1 表示 NALU Length Size，即一个 NALU 长度用几个字节表示，一般是 4 字节
    unsigned int(2) lengthSizeMinusOne;
    bit(3) reserved = ‘111’b;
    // sps 的个数
    unsigned int(5) numOfSequenceParameterSets;
    for (i=0; i< numOfSequenceParameterSets; i++) {
        // 两个字节表示一个 sps 的长度
        unsigned int(16) sequenceParameterSetLength ;
        // sps 的内容
        bit(8*sequenceParameterSetLength) sequenceParameterSetNALUnit;
    }
    // pps 的个数
    unsigned int(8) numOfPictureParameterSets;
    for (i=0; i< numOfPictureParameterSets; i++) {
        // 两个字节表示一个 pps 的长度
        unsigned int(16) pictureParameterSetLength;
        // pps 的内容
        bit(8*pictureParameterSetLength) pictureParameterSetNALUnit;
    }
}
```

**2）H.265 HEVCDecoderConfigurationRecord 数据结构**

这种数据结构用于将 VPS、SPS、PPS 封装为 `ExtraData`，封装好的 `ExtraData` 可以放置在 FLV 与 MP4 文件头中，播放器获取后用于初始化解码器。

```c
class HEVCDecoderConfigurationRecord {
    unsigned int(8)  configurationVersion;
    unsigned int(2)  general_profile_space;
    unsigned int(1)  general_tier_flag;
    unsigned int(5)  general_profile_idc;
    unsigned int(32) general_profile_compatibility_flags;
    unsigned int(48) general_constraint_indicator_flags;
    unsigned int(8)  general_level_idc;
    bit(4) reserved = ‘1111’b;
    unsigned int(12) min_spatial_segmentation_idc;
    bit(6) reserved = ‘111111’b;
    unsigned int(2)  parallelismType;
    bit(6) reserved = ‘111111’b;
    unsigned int(2)  chromaFormat;
    bit(5) reserved = ‘11111’b;
    unsigned int(3)  bitDepthLumaMinus8;
    bit(5) reserved = ‘11111’b;
    unsigned int(3)  bitDepthChromaMinus8;
    bit(16) avgFrameRate;
    bit(2)  constantFrameRate;
    bit(3)  numTemporalLayers;
    bit(1)  temporalIdNested;
    // lengthSizeMinusOne + 1 表示 NALU Length Size，即一个 NALU 长度用几个字节表示，一般是 4 字节
    unsigned int(2) lengthSizeMinusOne;
    // 数组长度
    unsigned int(8) numOfArrays;
    for (j=0; j < numOfArrays; j++) {
        bit(1) array_completeness;
        unsigned int(1)  reserved = 0;
        // NALU 的类型
        unsigned int(6)  NAL_unit_type;
        // 上面 NALU 类型的数量
        unsigned int(16) numNalus;
        for (i=0; i< numNalus; i++) {
            unsigned int(16) nalUnitLength;
            bit(8*nalUnitLength) nalUnit;
        }
    }
    // ...
}
```

**3）AnnexB 格式**

除了使用 H.264 的 `AVCDecoderConfigurationRecord` 格式和 H.265 的 `HEVCDecoderConfigurationRecord` 格式来封装 SPS、PPS、VPS，还可以用 `StartCode` 来分割 VPS、SPS、PPS。

```c
[start code] NALU | [start code] NALU | ...
```

这种格式比较常见，它在每个帧前面使用 `0x 00 00 00 01` 或者 `0x 00 00 01` 作为起始码。H.264 码流数据存储为文件时一般就是采用的这种格式，每个帧前面都要有一个起始码。SPS、PPS 作为一类 NALU 存储在这个码流中，一般放在码流最前面。也就是说这种格式包含 VCL 和非 VCL 类型的 NALU。

## 2、AVCC/HVCC 与 AnnexB 相互转换的底层源码

**1）AVCDecoderConfigurationRecord 转换为 AnnexB**

要实现 H.264 两种数据格式之间的转换，需要依赖 SPS、PPS 作为媒介。整体转换过程为：`AVCDecoderConfigurationRecord` <-> `SPS、PPS` <-> `00 00 00 01 SPS 00 00 00 01 PPS`。

具体实现可以参考 FFmpeg 的 `libavcodec/h264_mp4toannexb_bsf.c` 文件中的 `h264_extradata_to_annexb` 函数：

```c
static int h264_extradata_to_annexb(AVBSFContext *ctx, const int padding)
{
    H.264BSFContext *s = ctx->priv_data;
    GetByteContext ogb, *gb = &ogb;
    uint16_t unit_size;
    uint32_t total_size                 = 0;
    uint8_t *out                        = NULL, unit_nb, sps_done = 0;
    static const uint8_t nalu_header[4] = { 0, 0, 0, 1 };
    int length_size, pps_offset = 0;

    bytestream2_init(gb, ctx->par_in->extradata, ctx->par_in->extradata_size);

    bytestream2_skipu(gb, 4);

    /* retrieve length coded size */
    length_size = (bytestream2_get_byteu(gb) & 0x3) + 1;

    /* retrieve sps and pps unit(s) */
    unit_nb = bytestream2_get_byteu(gb) & 0x1f; /* number of sps unit(s) */
    if (!unit_nb) {
        goto pps;
    }

    while (unit_nb--) {
        int err;

        /* possible overread ok due to padding */
        unit_size   = bytestream2_get_be16u(gb);
        total_size += unit_size + 4;
        av_assert1(total_size <= INT_MAX - padding);
        if (bytestream2_get_bytes_left(gb) < unit_size + !sps_done) {
            av_log(ctx, AV_LOG_ERROR, "Global extradata truncated, "
                   "corrupted stream or invalid MP4/AVCC bitstream\n");
            av_free(out);
            return AVERROR_INVALIDDATA;
        }
        if ((err = av_reallocp(&out, total_size + padding)) < 0)
            return err;
        memcpy(out + total_size - unit_size - 4, nalu_header, 4);
        bytestream2_get_bufferu(gb, out + total_size - unit_size, unit_size);
pps:
        if (!unit_nb && !sps_done++) {
            unit_nb = bytestream2_get_byteu(gb); /* number of pps unit(s) */
            pps_offset = total_size;
        }
    }

    if (out)
        memset(out + total_size, 0, padding);

    if (pps_offset) {
        s->sps      = out;
        s->sps_size = pps_offset;
    } else {
        av_log(ctx, AV_LOG_WARNING,
               "Warning: SPS NALU missing or invalid. "
               "The resulting stream may not play.\n");
    }
    if (pps_offset < total_size) {
        s->pps      = out + pps_offset;
        s->pps_size = total_size - pps_offset;
    } else {
        av_log(ctx, AV_LOG_WARNING,
               "Warning: PPS NALU missing or invalid. "
               "The resulting stream may not play.\n");
    }

    av_freep(&ctx->par_out->extradata);
    ctx->par_out->extradata      = out;
    ctx->par_out->extradata_size = total_size;

    return length_size;
}
```

**2）AnnexB 转换为 AVCDecoderConfigurationRecord**

`libavformat/avc.c` 文件的 `ff_isom_write_avcc` 函数负责把 `AVStream->par->extradata` 中的 SPS、PPS 封装成 `AVCDecoderConfigurationRecord`。

`ff_isom_write_avcc` 主要逻辑是判断 `AVStream->par->extradata` 是什么形式的 SPS、PPS，若 `AVStream->par->extradata` 是以 AnnexB 分割的 SPS、PPS，则首先提取 SPS、PPS，然后按照 `AVCDecoderConfigurationRecord` 格式写入，否则可以直接写入。

```c
int ff_isom_write_avcc(AVIOContext *pb, const uint8_t *data, int len)
{
    AVIOContext *sps_pb = NULL, *pps_pb = NULL, *sps_ext_pb = NULL;
    uint8_t *buf, *end, *start;
    uint8_t *sps, *pps, *sps_ext;
    uint32_t sps_size = 0, pps_size = 0, sps_ext_size = 0;
    int ret, nb_sps = 0, nb_pps = 0, nb_sps_ext = 0;

    if (len <= 6)
        return AVERROR_INVALIDDATA;

    /* check for H.264 start code */
    if (AV_RB32(data) != 0x00000001 &&
        AV_RB24(data) != 0x000001) {
        avio_write(pb, data, len);
        return 0;
    }

    ret = ff_avc_parse_nal_units_buf(data, &buf, &len);
    if (ret < 0)
        return ret;
    start = buf;
    end = buf + len;

    ret = avio_open_dyn_buf(&sps_pb);
    if (ret < 0)
        goto fail;
    ret = avio_open_dyn_buf(&pps_pb);
    if (ret < 0)
        goto fail;
    ret = avio_open_dyn_buf(&sps_ext_pb);
    if (ret < 0)
        goto fail;

    /* look for sps and pps */
    while (end - buf > 4) {
        uint32_t size;
        uint8_t nal_type;
        size = FFMIN(AV_RB32(buf), end - buf - 4);
        buf += 4;
        nal_type = buf[0] & 0x1f;

        if (nal_type == 7) { /* SPS */
            nb_sps++;
            if (size > UINT16_MAX || nb_sps >= H.264_MAX_SPS_COUNT) {
                ret = AVERROR_INVALIDDATA;
                goto fail;
            }
            avio_wb16(sps_pb, size);
            avio_write(sps_pb, buf, size);
        } else if (nal_type == 8) { /* PPS */
            nb_pps++;
            if (size > UINT16_MAX || nb_pps >= H.264_MAX_PPS_COUNT) {
                ret = AVERROR_INVALIDDATA;
                goto fail;
            }
            avio_wb16(pps_pb, size);
            avio_write(pps_pb, buf, size);
        } else if (nal_type == 13) { /* SPS_EXT */
            nb_sps_ext++;
            if (size > UINT16_MAX || nb_sps_ext >= 256) {
                ret = AVERROR_INVALIDDATA;
                goto fail;
            }
            avio_wb16(sps_ext_pb, size);
            avio_write(sps_ext_pb, buf, size);
        }

        buf += size;
    }
    sps_size = avio_get_dyn_buf(sps_pb, &sps);
    pps_size = avio_get_dyn_buf(pps_pb, &pps);
    sps_ext_size = avio_get_dyn_buf(sps_ext_pb, &sps_ext);

    if (sps_size < 6 || !pps_size) {
        ret = AVERROR_INVALIDDATA;
        goto fail;
    }

    avio_w8(pb, 1); /* version */
    avio_w8(pb, sps[3]); /* profile */
    avio_w8(pb, sps[4]); /* profile compat */
    avio_w8(pb, sps[5]); /* level */
    avio_w8(pb, 0xff); /* 6 bits reserved (111111) + 2 bits nal size length - 1 (11) */
    avio_w8(pb, 0xe0 | nb_sps); /* 3 bits reserved (111) + 5 bits number of sps */

    avio_write(pb, sps, sps_size);
    avio_w8(pb, nb_pps); /* number of pps */
    avio_write(pb, pps, pps_size);

    if (sps[3] != 66 && sps[3] != 77 && sps[3] != 88) {
        H.264SPS seq;
        ret = ff_avc_decode_sps(&seq, sps + 3, sps_size - 3);
        if (ret < 0)
            goto fail;

        avio_w8(pb, 0xfc |  seq.chroma_format_idc); /* 6 bits reserved (111111) + chroma_format_idc */
        avio_w8(pb, 0xf8 | (seq.bit_depth_luma - 8)); /* 5 bits reserved (11111) + bit_depth_luma_minus8 */
        avio_w8(pb, 0xf8 | (seq.bit_depth_chroma - 8)); /* 5 bits reserved (11111) + bit_depth_chroma_minus8 */
        avio_w8(pb, nb_sps_ext); /* number of sps ext */
        if (nb_sps_ext)
            avio_write(pb, sps_ext, sps_ext_size);
    }

fail:
    ffio_free_dyn_buf(&sps_pb);
    ffio_free_dyn_buf(&pps_pb);
    ffio_free_dyn_buf(&sps_ext_pb);
    av_free(start);

    return ret;
}
```

**3）HEVCDecoderConfigurationRecord 转换为 AnnexB**

要实现 H.265 两种数据格式之间的转换，需要依赖 VPS、SPS、PPS 作为媒介。整体转换过程为：`HEVCDecoderConfigurationRecord` <-> `VPS、SPS、PPS` <-> `00 00 00 01 VPS 00 00 00 01 SPS 00 00 00 01 PPS`。

具体实现可以参考 FFmpeg 的 `libavcodec/hevc_mp4toannexb_bsf.c` 文件中的 `hevc_extradata_to_annexb` 函数：

```c
static int hevc_extradata_to_annexb(AVBSFContext *ctx)
{
    GetByteContext gb;
    int length_size, num_arrays, i, j;
    int ret = 0;

    uint8_t *new_extradata = NULL;
    size_t   new_extradata_size = 0;

    bytestream2_init(&gb, ctx->par_in->extradata, ctx->par_in->extradata_size);

    bytestream2_skip(&gb, 21);
    length_size = (bytestream2_get_byte(&gb) & 3) + 1;
    num_arrays  = bytestream2_get_byte(&gb);

    for (i = 0; i < num_arrays; i++) {
        int type = bytestream2_get_byte(&gb) & 0x3f;
        int cnt  = bytestream2_get_be16(&gb);

        if (!(type == HEVC_NAL_VPS || type == HEVC_NAL_SPS || type == HEVC_NAL_PPS ||
              type == HEVC_NAL_SEI_PREFIX || type == HEVC_NAL_SEI_SUFFIX)) {
            av_log(ctx, AV_LOG_ERROR, "Invalid NAL unit type in extradata: %d\n",
                   type);
            ret = AVERROR_INVALIDDATA;
            goto fail;
        }

        for (j = 0; j < cnt; j++) {
            int nalu_len = bytestream2_get_be16(&gb);

            if (4 + AV_INPUT_BUFFER_PADDING_SIZE + nalu_len > SIZE_MAX - new_extradata_size) {
                ret = AVERROR_INVALIDDATA;
                goto fail;
            }
            ret = av_reallocp(&new_extradata, new_extradata_size + nalu_len + 4 + AV_INPUT_BUFFER_PADDING_SIZE);
            if (ret < 0)
                goto fail;

            AV_WB32(new_extradata + new_extradata_size, 1); // add the startcode
            bytestream2_get_buffer(&gb, new_extradata + new_extradata_size + 4, nalu_len);
            new_extradata_size += 4 + nalu_len;
            memset(new_extradata + new_extradata_size, 0, AV_INPUT_BUFFER_PADDING_SIZE);
        }
    }

    av_freep(&ctx->par_out->extradata);
    ctx->par_out->extradata      = new_extradata;
    ctx->par_out->extradata_size = new_extradata_size;

    if (!new_extradata_size)
        av_log(ctx, AV_LOG_WARNING, "No parameter sets in the extradata\n");

    return length_size;
fail:
    av_freep(&new_extradata);
    return ret;
}
```

**4）AnnexB 转换为 HEVCDecoderConfigurationRecord**
    
`libavformat/hevc.c` 文件的 `ff_isom_write_hvcc` 函数负责把 `AVStream->par->extradata` 中的 VPS、SPS、PPS 封装成 `HEVCDecoderConfigurationRecord`，并写入 HVCC Box。

```c
int ff_isom_write_hvcc(AVIOContext *pb, const uint8_t *data,
                       int size, int ps_array_completeness)
{
    HEVCDecoderConfigurationRecord hvcc;
    uint8_t *buf, *end, *start;
    int ret;

    if (size < 6) {
        /* We can't write a valid hvcC from the provided data */
        return AVERROR_INVALIDDATA;
    } else if (*data == 1) {
        /* Data is already hvcC-formatted */
        avio_write(pb, data, size);
        return 0;
    } else if (!(AV_RB24(data) == 1 || AV_RB32(data) == 1)) {
        /* Not a valid Annex B start code prefix */
        return AVERROR_INVALIDDATA;
    }

    ret = ff_avc_parse_nal_units_buf(data, &start, &size);
    if (ret < 0)
        return ret;

    hvcc_init(&hvcc);

    buf = start;
    end = start + size;

    while (end - buf > 4) {
        uint32_t len = FFMIN(AV_RB32(buf), end - buf - 4);
        uint8_t type = (buf[4] >> 1) & 0x3f;

        buf += 4;

        for (unsigned i = 0; i < FF_ARRAY_ELEMS(hvcc.arrays); i++) {
            static const uint8_t array_idx_to_type[] =
                { HEVC_NAL_VPS, HEVC_NAL_SPS, HEVC_NAL_PPS,
                  HEVC_NAL_SEI_PREFIX, HEVC_NAL_SEI_SUFFIX };

            if (type == array_idx_to_type[i]) {
                ret = hvcc_add_nal_unit(buf, len, ps_array_completeness,
                                        &hvcc, i);
                if (ret < 0)
                    goto end;
                break;
            }
        }

        buf += len;
    }

    ret = hvcc_write(pb, &hvcc);

end:
    hvcc_close(&hvcc);
    av_free(start);
    return ret;
}
```

## 3、AVCC/HVCC 与 AnnexB 相互转换工程实现

**1）使用 FFmpeg 实现 AVCDecoderConfigurationRecord、HEVCDecoderConfigurationRecord 转换为 AnnexB**

通过调用 `AVBitStreamFilterContext` 进行实现，H.264 查找 `h264_mp4toannexb`，H.265 查找 `hevc_mp4toannexb`。

**处理单帧 AVPacket：**

```c
void run()
{
    AVBitStreamFilterContext* bsfc = ifmt_ctx->streams[pkt->stream_index]->codec->codec_id == AV_CODEC_ID_H.264 ? av_bitstream_filter_init("h264_mp4toannexb") : av_bitstream_filter_init("hevc_mp4toannexb"); 
    while (running)
    {
        ret = av_read_frame(ifmt_ctx, pkt);
        if(isVideo){
            av_bitstream_filter_filter(bsfc, ifmt_ctx->streams[pkt->stream_index]->codec, NULL, &pkt->data, &pkt->size, pkt->data, pkt->size, 0);
        }
    }
    
    if (bsfc) {
        av_bitstream_filter_close(bsfc);
    }
}
```

**处理 ExtraData：**

```c
void run()
{
    AVBitStreamFilterContext* bsfc = ifmt_ctx->streams[pkt->stream_index]->codec->codec_id == AV_CODEC_ID_H.264 ? av_bitstream_filter_init("h264_mp4toannexb") : 
    uint8_t *tmpData = NULL;
    int tmpSize = 0;
    av_bitstream_filter_filter(bsfc, stream->codec, NULL, &tmpData, &tmpSize, tmpData, tmpSize, 1);
    BSFCompatContext *compatContext = bsfc->priv_data;
    AVBSFContext *bsfContext = compatContext->ctx;
    tmpData = bsfContext->par_out->extradata;
    tmpSize = bsfContext->par_out->extradata_size;
}
```

**2）使用 FFmpeg 实现 AnnexB 转换为 AVCDecoderConfigurationRecord、HEVCDecoderConfigurationRecord**

H.264 通过调用文件 `libavformat/avc.c` 文件的 `ff_isom_write_avcc` 函数实现，H.265 通过调用文件 `libavformat/hevc.c` 文件的 `ff_isom_write_hvcc` 函数实现。

```c
void run()
{
    uint8_t *outExtra = NULL;
    int outExtraSize = 0;
    AVIOContext *pb;
    if (avio_open_dyn_buf(&pb) < 0) {
        return;
    }

    if (format_id == kCMVideoCodecType_HEVC) {
        ff_isom_write_hvcc(pb, annexb_extra_data_, annexb_extra_size_, 0);
    } else {
        ff_isom_write_avcc(pb, annexb_extra_data_, annexb_extra_size_);
    }
   outExtraSize = avio_close_dyn_buf(pb, &outExtra);
}
```
