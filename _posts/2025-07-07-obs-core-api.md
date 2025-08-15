---
title: 探索 OBS 开发（7）：OBS 的核心 API
description: 系统介绍 OBS 开发相关的基础技术。
author: Keyframe
date: 2025-07-07 18:08:08 +0800
categories: [音视频基础知识]
tags: [音视频基础知识, 音视频, 视频, 音频, 拍摄, OBS]
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



## 初始化、关闭和信息

```cpp
bool obs_startup(const char* locale, const char* module_config_path, profiler_name_store_t* store)
```
初始化 OBS 核心上下文。

参数：
- **locale**：模块使用的本地化设置（例如“en-US”）
- **module_config_path**：模块配置存储目录的路径（如果无则为_NULL_）
- **store**：OBS 使用的分析器名称存储，或者 NULL

返回值：
- 如果已初始化或初始化失败，返回_false_

```cpp
void obs_shutdown(void)
```
释放与 OBS 关联的所有数据并终止 OBS 上下文。

```cpp
bool obs_initialized(void)
```
返回值：
- 如果主 OBS 上下文已初始化，返回_true_

```cpp
uint32_t obs_get_version(void)
```
返回值：
- 当前核心版本

```cpp
const char* obs_get_version_string(void)
```
返回值：
- 当前核心版本字符串

```cpp
void obs_set_locale(const char* locale)
```
设置模块使用的新本地化设置，这会为每个模块调用`obs_module_set_locale()`。

参数：
- **locale**：模块使用的本地化设置

```cpp
const char* obs_get_locale(void)
```
返回值：
- 当前本地化设置

```cpp
profiler_name_store_t* obs_get_profiler_name_store(void)
```
返回值：
- OBS 使用的分析器名称存储（见 util/profiler.h），可以是传递给`obs_startup()`的名称存储、内部名称存储，或者如果`obs_initialized()`返回 false 则为 NULL。

```cpp
int obs_reset_video(struct obs_video_info* ovi)
```
设置基础视频输出的分辨率、帧率和格式。

注意：如果当前有输出在运行，则无法更改此数据。

注意：如果不完全销毁 OBS 上下文，则无法更改图形模块。

参数：
- **ovi**：指向包含图形子系统规范的 obs_video_info 结构体。

返回值：
- OBS_VIDEO_SUCCESS - 成功
- OBS_VIDEO_NOT_SUPPORTED - 适配器缺乏能力
- OBS_VIDEO_INVALID_PARAM - 参数无效
- OBS_VIDEO_CURRENTLY_ACTIVE - 视频当前正在运行
- OBS_VIDEO_MODULE_NOT_FOUND - 未找到图形模块
- OBS_VIDEO_FAIL - 通用失败

与此函数相关联的数据类型：

```cpp
struct obs_video_info {
        /**
         * 使用的图形模块（通常是"libobs-opengl"或"libobs-d3d11"）
         */
        const char          *graphics_module;

        uint32_t            fps_num;       /**< 输出帧率分子 */
        uint32_t            fps_den;       /**< 输出帧率分母 */

        uint32_t            base_width;    /**< 基础合成宽度 */
        uint32_t            base_height;   /**< 基础合成高度 */

        uint32_t            output_width;  /**< 输出宽度 */
        uint32_t            output_height; /**< 输出高度 */
        enum video_format   output_format; /**< 输出格式 */

        /** 使用的视频适配器索引（注：避免在 Optimus 笔记本电脑上使用） */
        uint32_t            adapter;

        /** 使用着色器转换为不同的颜色格式 */
        bool                gpu_conversion;

        enum video_colorspace colorspace;  /**< YUV 类型（如果使用 YUV） */
        enum video_range_type range;       /**< YUV 范围（如果使用 YUV） */

        enum obs_scale_type scale_type;    /**< 如果需要缩放，则使用何种方式缩放 */
};
```

```cpp
bool obs_reset_audio(const struct obs_audio_info* oai)
```
设置基础音频输出的格式、声道、采样率等。

注意：如果当前有输出在运行，则无法重置基础音频。

返回值：
- 成功返回_true_，失败返回_false_

与此函数相关联的数据类型：

```cpp
struct obs_audio_info {
        uint32_t            samples_per_sec;
        enum speaker_layout speakers;
};
```

```cpp
bool obs_reset_audio2(const struct obs_audio_info2* oai)
```
设置基础音频输出的格式、声道、采样率等。还可以设置 OBS 的最大音频延迟，以及音频缓冲是固定还是动态增加。

当使用固定音频缓冲时，OBS将在启动时自动缓冲到最大音频延迟。

最大音频延迟将被限制为音频输出帧数（通常是 1024 音频帧）的最接近的倍数。

注意：如果当前有输出在运行，则无法重置基础音频。

返回值：
- 成功返回_true_，失败返回_false_

与此函数相关联的数据类型：

```cpp
struct obs_audio_info2 {
        uint32_t            samples_per_sec;
        enum speaker_layout speakers;

        uint32_t max_buffering_ms;
        bool fixed_buffering;
};
```

```cpp
bool obs_get_video_info(struct obs_video_info* ovi)
```
获取当前视频设置。

返回值：
- 如果没有视频返回_false_

```cpp
float obs_get_video_sdr_white_level(void)
```
获取当前 SDR 白色水平。

返回值：
- 如果没有视频，返回 300.0f

```cpp
float obs_get_video_hdr_nominal_peak_level(void)
```
获取当前 HDR 标称峰值水平。

返回值：
- 如果没有视频，返回 1000.0f

```cpp
void obs_set_video_sdr_white_level(float sdr_white_level, float hdr_nominal_peak_level)
```
设置当前视频水平。

```cpp
bool obs_get_audio_info(struct obs_audio_info* oai)
```
获取当前音频设置。

返回值：
- 如果没有音频返回_false_

## Libobs 对象

```cpp
bool obs_enum_source_types(size_t idx, const char** id)
```
枚举所有源类型（输入源、滤镜、转场等）。

```cpp
bool obs_enum_input_types(size_t idx, const char** id)
```
枚举所有可用的输入源类型。

输入源是通用的源输入（如捕获源、设备源等）。

```cpp
bool obs_enum_filter_types(size_t idx, const char** id)
```
枚举所有可用的滤镜源类型。

滤镜是用于修改其他源的视频/音频输出的源。

```cpp
bool obs_enum_transition_types(size_t idx, const char** id)
```
枚举所有可用的转场源类型。

转场是用于在两个或多个其他源之间进行切换的源。

```cpp
bool obs_enum_output_types(size_t idx, const char** id)
```
枚举所有可用的输出类型。

```cpp
bool obs_enum_encoder_types(size_t idx, const char** id)
```
枚举所有可用的编码器类型。

```cpp
bool obs_enum_service_types(size_t idx, const char** id)
```
枚举所有可用的服务类型。

```cpp
void obs_enum_sources(bool(*enum_proc)(void*, obs_source_t*), void* param)
```
枚举所有输入源。

回调函数返回 true 继续枚举，返回 false 结束枚举。

如果要在 obs_enum_sources 结束后保留引用，请使用`obs_source_get_ref()`或`obs_source_get_weak_source()`。

对于脚本，使用`obs_enum_sources()`。

```cpp
void obs_enum_scenes(bool(*enum_proc)(void*, obs_source_t*), void* param)
```
枚举所有场景。如果需要将场景作为`obs_scene_t`使用，请使用`obs_scene_from_source()`。枚举顺序不应被依赖。如果打算按照 OBS Studio 前端显示的顺序枚举场景，请使用`obs_frontend_get_scenes()`。

回调函数返回 true 继续枚举，返回 false 结束枚举。

如果要在 obs_enum_scenes 结束后保留引用，请使用`obs_source_get_ref()`或`obs_source_get_weak_source()`。

```cpp
void obs_enum_outputs(bool(*enum_proc)(void*, obs_output_t*), void* param)
```
枚举输出。

回调函数返回 true 继续枚举，返回 false 结束枚举。

如果要在 obs_enum_outputs 结束后保留引用，请使用`obs_output_get_ref()`或`obs_output_get_weak_output()`。

```cpp
void obs_enum_encoders(bool(*enum_proc)(void*, obs_encoder_t*), void* param)
```
枚举编码器。

回调函数返回 true 继续枚举，返回 false 结束枚举。

如果要在 obs_enum_encoders 结束后保留引用，请使用`obs_encoder_get_ref()`或`obs_encoder_get_weak_encoder()`。

```cpp
obs_source_t* obs_get_source_by_name(const char* name)
```
通过名称获取源。

增加源的引用计数，完成时请使用`obs_source_release()`释放。

```cpp
obs_source_t* obs_get_source_by_uuid(const char* uuid)
```
通过 UUID 获取源。

增加源的引用计数，完成时请使用`obs_source_release()`释放。

```cpp
obs_source_t* obs_get_transition_by_name(const char* name)
```
通过名称获取转场。

增加源的引用计数，完成时请使用`obs_source_release()`释放。

```cpp
obs_source_t* obs_get_transition_by_uuid(const char* uuid)
```
通过 UUID 获取转场。

增加源的引用计数，完成时请使用`obs_source_release()`释放。

```cpp
obs_scene_t* obs_get_scene_by_name(const char* name)
```
通过名称获取场景。

增加场景的引用计数，完成时请使用`obs_scene_release()`释放。

```cpp
obs_output_t* obs_get_output_by_name(const char* name)
```
通过名称获取输出。

增加输出的引用计数，完成时请使用`obs_output_release()`释放。

```cpp
obs_encoder_t* obs_get_encoder_by_name(const char* name)
```
通过名称获取编码器。

增加编码器的引用计数，完成时请使用`obs_encoder_release()`释放。

```cpp
obs_service_t* obs_get_service_by_name(const char* name)
```
通过名称获取服务。

增加服务的引用计数，完成时请使用`obs_service_release()`释放。

```cpp
obs_data_t* obs_save_source(obs_source_t* source)
```
返回值：
- 源保存数据的新引用。使用`obs_data_release()`在完成时释放。

```cpp
obs_source_t* obs_load_source(obs_data_t* data)
```
返回值：
- 从保存的数据创建的源。

```cpp
void obs_load_sources(obs_data_array_t* array, obs_load_source_cb cb, void* private_data)
```
辅助函数，用于从数据数组加载活动源。

与此函数相关联的数据类型：

```cpp
typedef void (*obs_load_source_cb)(void *private_data, obs_source_t *source);
```

```cpp
obs_data_array_t* obs_save_sources(void)
```
返回值：
- 包含所有活动源保存数据的数据数组。

```cpp
obs_data_array_t* obs_save_sources_filtered(obs_save_source_filter_cb cb, void* data)
```
返回值：
- 通过_cb_函数过滤后的所有活动源保存数据的数据数组。

与此函数相关联的数据类型：

```cpp
typedef bool (*obs_save_source_filter_cb)(void *data, obs_source_t *source);
```

## 视频、音频和图形

```cpp
void obs_enter_graphics(void)
```
辅助函数，用于进入 OBS 图形上下文。

```cpp
void obs_leave_graphics(void)
```
辅助函数，用于离开 OBS 图形上下文。

```cpp
audio_t* obs_get_audio(void)
```
返回值：
- 此 OBS 上下文的主音频输出处理器。

```cpp
video_t* obs_get_video(void)
```
返回值：
- 此 OBS 上下文的主视频输出处理器。

```cpp
void obs_set_output_source(uint32_t channel, obs_source_t* source)
```
设置通道的主输出源。

```cpp
obs_source_t* obs_get_output_source(uint32_t channel)
```
获取通道的主输出源并增加该源的引用计数。使用`obs_source_release()`释放。

```cpp
gs_effect_t* obs_get_base_effect(enum obs_base_effect effect)
```
返回一个常用的基效果。

参数：
- **effect**：

可以是以下值之一：
- OBS_EFFECT_DEFAULT - RGB/YUV
- OBS_EFFECT_DEFAULT_RECT - RGB/YUV（使用 texture_rect）
- OBS_EFFECT_OPAQUE - RGB/YUV（将 alpha 设置为 1.0）
- OBS_EFFECT_SOLID - RGB/YUV（仅纯色）
- OBS_EFFECT_BICUBIC - 双三次缩小
- OBS_EFFECT_LANCZOS - Lanczos 缩小
- OBS_EFFECT_BILINEAR_LOWRES - 双线性低分辨率缩小
- OBS_EFFECT_PREMULTIPLIED_ALPHA - 预乘 alpha

```cpp
void obs_render_main_texture(void)
```
渲染主输出纹理。对于渲染预览窗格的主输出很有用。

```cpp
bool obs_audio_monitoring_available(void)
```
返回值：
- 当前平台上是否支持音频监控。

```cpp
void obs_reset_audio_monitoring(void)
```

```cpp
void obs_enum_audio_monitoring_devices(obs_enum_audio_device_cb cb, void* data)
```
枚举可用于音频监控的音频设备。

回调函数返回 true 继续枚举，返回 false 结束枚举。

与此函数相关联的数据类型：

```cpp
typedef bool (*obs_enum_audio_device_cb)(void *data, const char *name, const char *id);
```

```cpp
bool obs_set_audio_monitoring_device(const char* name, const char* id)
```
设置当前用于音频监控的音频设备。

```cpp
void obs_get_audio_monitoring_device(const char** name, const char** id)
```
获取当前用于音频监控的音频设备。

```cpp
void obs_add_main_render_callback(void(*draw)(void* param,uint32_t cx,uint32_t cy), void* param)
```

```cpp
void obs_remove_main_render_callback(void(*draw)(void* param,uint32_t cx,uint32_t cy), void* param)
```
添加/移除主渲染回调。允许自定义渲染到主流/录制输出。

对于脚本（仅限 Lua），使用`obs_add_main_render_callback()`和`obs_remove_main_render_callback()`。

```cpp
void obs_add_main_rendered_callback(void(*rendered)(void* param), void* param)
```

```cpp
void obs_remove_main_rendered_callback(void(*rendered)(void* param), void* param)
```
添加/移除主渲染回调。允许使用主流/录制输出的结果。

```cpp
void obs_add_raw_video_callback(const struct video_scale_info* conversion, void(*callback)(void* param,struct video_data* frame), void* param)
```

```cpp
void obs_remove_raw_video_callback(void(*callback)(void* param,struct video_data* frame), void* param)
```
添加/移除原始视频回调。允许获取原始视频帧而无需使用输出。

参数：
- **conversion**：指定转换要求。可以是 NULL。
- **callback**：接收原始视频帧的回调。
- **param**：与回调关联的私有数据。

```cpp
void obs_add_raw_audio_callback(size_t mix_idx, const struct audio_convert_info* conversion, audio_output_callback_t callback, void* param)
```

```cpp
void obs_remove_raw_audio_callback(size_t track, audio_output_callback_t callback, void* param)
```
添加/移除原始音频回调。允许获取原始音频数据而无需使用输出。

参数：
- **mix_idx**：指定要获取数据的音频轨道。
- **conversion**：指定转换要求。可以是 NULL。
- **callback**：接收原始音频数据的回调。
- **param**：与回调关联的私有数据。

## 主要信号/过程处理程序

```cpp
signal_handler_t* obs_get_signal_handler(void)
```
返回值：
- 主 OBS 信号处理程序。不应手动释放，因为其生命周期由 libobs 管理。

有关核心信号的更多信息，请参见核心 OBS 信号。

```cpp
proc_handler_t* obs_get_proc_handler(void)
```
返回值：
- 主 OBS 过程处理程序。不应手动释放，因为其生命周期由 libobs 管理。

## 核心 OBS 信号

- **source_create** (ptr source)
  > 源创建时调用。

- **source_destroy** (ptr source)
  > 源销毁时调用。

- **source_remove** (ptr source)
  > 源被移除时调用（调用了`obs_source_remove()`）。

- **source_update** (ptr source)
  > 源的设置被更新时调用。

- **source_save** (ptr source)
  > 源被保存时调用。

- **source_load** (ptr source)
  > 源被加载时调用。

- **source_activate** (ptr source)
  > 源在主视图中被激活时调用（在流/录制中可见）。

- **source_deactivate** (ptr source)
  > 源在主视图中被停用时调用（不再在流/录制中可见）。

- **source_show** (ptr source)
  > 源在任何显示和/或主视图中可见时调用。

- **source_hide** (ptr source)
  > 源在任何显示和/或主视图中不再可见时调用。

- **source_rename** (ptr source, string new_name, string prev_name)
  > 源重命名时调用。

- **source_volume** (ptr source, in out float volume)
  > 源的音量更改时调用。

- **source_audio_activate** (ptr source)
  > 源的音频激活时调用。

- **source_audio_deactivate** (ptr source)
  > 源的音频停用时调用。

- **source_filter_add** (ptr source, ptr filter)
  > 滤镜被添加到源时调用。

- **source_filter_remove** (ptr source, ptr filter)
  > 滤镜被从源中移除时调用。

- **source_transition_start** (ptr source)
  > 转场开始时调用。

- **source_transition_video_stop** (ptr source)
  > 转场的视频部分停止时调用。

- **source_transition_stop** (ptr source)
  > 转场停止时调用。

- **channel_change** (int channel, in out ptr source, ptr prev_source)
  > 调用`obs_set_output_source()`时调用。

- **hotkey_layout_change** ()
  > 热键布局更改时调用。

- **hotkey_register** (ptr hotkey)
  > 热键注册时调用。

- **hotkey_unregister** (ptr hotkey)
  > 热键注销时调用。

- **hotkey_bindings_changed** (ptr hotkey)
  > 热键绑定更改时调用。

## 显示

```cpp
obs_display_t* obs_display_create(const struct gs_init_data* graphics_data)
```
添加一个新的与主渲染管道关联的窗口显示。这会创建一个新的交换链，每帧更新一次。

（重要提示：在同一基础窗口的层次结构中不要使用多个显示小部件，这会在 macOS 上导致呈现停滞。）

参数：
- **graphics_data**：交换链初始化数据。

返回值：
- 新的显示上下文，如果失败返回 NULL。

与此函数相关联的数据类型：

```cpp
enum gs_color_format {
        [...]
        GS_RGBA,
        GS_BGRX,
        GS_BGRA,
        GS_RGBA16F,
        GS_RGBA32F,
        [...]
};

enum gs_zstencil_format {
        GS_ZS_NONE,
        GS_Z16,
        GS_Z24_S8,
        GS_Z32F,
        GS_Z32F_S8X24
};

struct gs_window {
#if defined(_WIN32)
        void                    *hwnd;
#elif defined(__APPLE__)
        __unsafe_unretained id  view;
#elif defined(__linux__) || defined(__FreeBSD__)
        uint32_t                id;
        void                    *display;
#endif
};

struct gs_init_data {
        struct gs_window        window;
        uint32_t                cx, cy;
        uint32_t                num_backbuffers;
        enum gs_color_format    format;
        enum gs_zstencil_format zsformat;
        uint32_t                adapter;
};
```

```cpp
void obs_display_destroy(obs_display_t* display)
```
销毁显示上下文。

```cpp
void obs_display_resize(obs_display_t* display, uint32_t cx, uint32_t cy)
```
更改显示上下文的大小。

```cpp
void obs_display_add_draw_callback(obs_display_t* display, void(*draw)(void* param,uint32_t cx,uint32_t cy), void* param)
```
为显示上下文添加绘制回调，该回调将在每次显示渲染时被调用。

参数：
- **display**：显示上下文
- **draw**：每次帧更新时调用的绘制回调
- **param**：与此绘制回调关联的用户数据

```cpp
void obs_display_remove_draw_callback(obs_display_t* display, void(*draw)(void* param,uint32_t cx,uint32_t cy), void* param)
```
移除显示上下文的绘制回调。

```cpp
void obs_display_set_enabled(obs_display_t* display, bool enable)
```

```cpp
bool obs_display_enabled(obs_display_t* display)
```
返回值：
- 如果显示已启用，返回_true_，否则返回_false_

```cpp
void obs_display_set_background_color(obs_display_t* display, uint32_t color)
```
设置显示上下文的背景（清除）颜色。

## 视图

```cpp
obs_view_t* obs_view_create(void)
```
返回值：
- 视图上下文。

```cpp
void obs_view_destroy(obs_view_t* view)
```
销毁视图上下文。

```cpp
void obs_view_render(obs_view_t* view)
```
渲染此视图上下文的源。

```cpp
video_t* obs_view_add(obs_view_t* view)
```
返回值：
- 添加到主渲染循环的视图上下文的主视频输出处理器。

```cpp
video_t* obs_view_add2(obs_view_t* view, struct obs_video_info* ovi)
```
将视图添加到主渲染循环，并使用自定义视频设置。

返回值：
- 添加到主渲染循环的视图上下文的主视频输出处理器。

```cpp
void obs_view_remove(obs_view_t* view)
```
从主渲染循环中移除视图。

```cpp
void obs_view_set_source(obs_view_t* view, uint32_t channel, obs_source_t* source)
```
设置此视图上下文使用的源。

```cpp
obs_source_t* obs_view_get_source(obs_view_t* view, uint32_t channel)
```
返回值：
- 此视图上下文当前使用的源。

```cpp
bool obs_view_get_video_info(obs_view_t* view, struct obs_video_info* ovi)
```
获取此视图上下文当前使用的第一个匹配混音的视频设置。

返回值：
- 如果没有视频返回_false_

```cpp
void obs_view_enum_video_info(obs_view_t* view, bool(*enum_proc)(void*,struct obs_video_info*), void* param)
```
枚举使用指定混音的所有混音的视频信息。

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

