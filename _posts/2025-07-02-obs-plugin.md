---
title: 探索 OBS 开发（2）：插件
description: 系统介绍 OBS 开发相关的基础技术。
author: Keyframe
date: 2025-07-02 18:08:08 +0800
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




几乎所有自定义功能都通过插件模块添加，这些插件模块通常是动态链接库或脚本。例如，捕获和/或输出音频/视频、录音、输出到 RTMP 流、使用 x264 编码等功能都是通过插件模块实现的。

插件可以实现源、输出、编码器和服务。

刚开始编写插件？我们提供了一个基本的模板插件来帮助你入门。

## 常见的目录结构和 CMakeLists.txt

通常，源文件的组织方式是一个文件用于插件初始化，然后每个对象分别对应一个文件。例如，如果你创建一个名为 “my - plugin” 的插件，你会有 my - plugin.c 文件用于插件初始化，my - source.c 文件用于定义自定义源，my - output.c 文件用于定义自定义输出等。（当然，这不是一个硬性规定）

以下是一个原生插件模块的常见目录结构示例：

```
my-plugin/data/locale/en-US.ini
my-plugin/CMakeLists.txt
my-plugin/my-plugin.c
my-plugin/my-source.c
my-plugin/my-output.c
my-plugin/my-encoder.c
my-plugin/my-service.c
```

以下是一个与这些文件关联的常见 CMakeLists.txt 文件示例：

```
# my-plugin/CMakeLists.txt

project(my-plugin)

set(my-plugin_SOURCES
      my-plugin.c
      my-source.c
      my-output.c
      my-encoder.c
      my-service.c)

add_library(my-plugin MODULE
      ${my-plugin_SOURCES})
target_link_libraries(my-plugin
      libobs)

install_obs_plugin_with_data(my-plugin data)
```

## 原生插件初始化

要创建一个原生插件模块，你需要包含 libobs/obs-module.h 头文件，使用 `OBS_DECLARE_MODULE()` 宏，然后创建 `obs_module_load()` 函数的定义。在你的 `obs_module_load()` 函数中，你需要注册你的自定义源、输出、编码器或服务。有关更多信息，请参阅模块 API 参考。

以下是一个 my-plugin.c 的示例，它将注册每种类型的对象：

```
/* my-plugin.c */
#include <obs-module.h>

/* 定义通用函数（必需） */
OBS_DECLARE_MODULE()

/* 实现基于 ini 的本地化（可选） */
OBS_MODULE_USE_DEFAULT_LOCALE("my-plugin", "en-US")

extern struct obs_source_info  my_source;  /* 在 my-source.c 中定义 */
extern struct obs_output_info  my_output;  /* 在 my-output.c 中定义 */
extern struct obs_encoder_info my_encoder; /* 在 my-encoder.c 中定义 */
extern struct obs_service_info my_service; /* 在 my-service.c 中定义 */

bool obs_module_load(void)
{
        obs_register_source(&my_source);
        obs_register_output(&my_output);
        obs_register_encoder(&my_encoder);
        obs_register_service(&my_service);
        return true;
}
```

## 源

源用于在流媒体中渲染视频和/或音频。例如，捕获显示器/游戏/音频、播放视频、显示图像或播放音频。源还可以用于实现音频和视频滤镜以及转场。libobs/obs-source.h 文件是实现源的专用头文件。有关更多信息，请参阅源 API 参考（obs_source_t）。

例如，要实现一个源对象，你需要定义一个 `obs_source_info` 结构并填写与你的源相关的信息和回调函数：

```
/* my-source.c */

[...]

struct obs_source_info my_source {
        .id           = "my_source",
        .type         = OBS_SOURCE_TYPE_INPUT,
        .output_flags = OBS_SOURCE_VIDEO,
        .get_name     = my_source_name,
        .create       = my_source_create,
        .destroy      = my_source_destroy,
        .update       = my_source_update,
        .video_render = my_source_render,
        .get_width    = my_source_width,
        .get_height   = my_source_height
};
```

然后，在 my-plugin.c 中，你将在 `obs_module_load()` 中调用 `obs_register_source()` 来向 libobs 注册源。

```
/* my-plugin.c */

[...]

extern struct obs_source_info  my_source;  /* 在 my-source.c 中定义 */

bool obs_module_load(void)
{
        obs_register_source(&my_source);

        [...]

        return true;
}
```

一些简单的源示例：

  * 同步视频源：图像源
  * 异步视频源：随机纹理测试源
  * 音频源：正弦波测试源
  * 视频滤镜：测试视频滤镜
  * 音频滤镜：增益音频滤镜

## 输出

输出允许输出当前正在渲染的音频/视频。流媒体和录音是输出的两个常见示例，但并非唯一的输出类型。输出可以接收原始数据或编码后的数据。libobs/obs-output.h 文件是实现输出的专用头文件。有关更多信息，请参阅输出 API 参考（obs_output_t）。

例如，要实现一个输出对象，你需要定义一个 `obs_output_info` 结构并填写与你的输出相关的信息和回调函数：

```
/* my-output.c */

[...]

struct obs_output_info my_output {
        .id                   = "my_output",
        .flags                = OBS_OUTPUT_AV | OBS_OUTPUT_ENCODED,
        .get_name             = my_output_name,
        .create               = my_output_create,
        .destroy              = my_output_destroy,
        .start                = my_output_start,
        .stop                 = my_output_stop,
        .encoded_packet       = my_output_data,
        .get_total_bytes      = my_output_total_bytes,
        .encoded_video_codecs = "h264",
        .encoded_audio_codecs = "aac"
};
```

然后，在 my-plugin.c 中，你将在 `obs_module_load()` 中调用 `obs_register_output()` 来向 libobs 注册输出。

```
/* my-plugin.c */

[...]

extern struct obs_output_info  my_output;  /* 在 my-output.c 中定义 */

bool obs_module_load(void)
{
        obs_register_output(&my_output);

        [...]

        return true;
}
```

一些输出示例：

  * 编码后的视频/音频输出：

    * FLV 输出
    * FFmpeg 封装器输出
    * RTMP 流输出

  * 原始视频/音频输出：

    * FFmpeg 输出

## 编码器

编码器是 OBS 特有的视频/音频编码器实现，用于使用编码器的输出。x264、NVENC、Quicksync 是编码器实现的示例。libobs/obs-encoder.h 文件是实现编码器的专用头文件。有关更多信息，请参阅编码器 API 参考（obs_encoder_t）。

例如，要实现一个编码器对象，你需要定义一个 `obs_encoder_info` 结构并填写与你的编码器相关的信息和回调函数：

```
/* my-encoder.c */

[...]

struct obs_encoder_info my_encoder_encoder = {
        .id             = "my_encoder",
        .type           = OBS_ENCODER_VIDEO,
        .codec          = "h264",
        .get_name       = my_encoder_name,
        .create         = my_encoder_create,
        .destroy        = my_encoder_destroy,
        .encode         = my_encoder_encode,
        .update         = my_encoder_update,
        .get_extra_data = my_encoder_extra_data,
        .get_sei_data   = my_encoder_sei,
        .get_video_info = my_encoder_video_info
};
```

然后，在 my-plugin.c 中，你将在 `obs_module_load()` 中调用 `obs_register_encoder()` 来向 libobs 注册编码器。

```
/* my-plugin.c */

[...]

extern struct obs_encoder_info my_encoder; /* 在 my-encoder.c 中定义 */

bool obs_module_load(void)
{
        obs_register_encoder(&my_encoder);

        [...]

        return true;
}
```

**重要说明：** 编码器设置目前有一些预期的通用设置值，这些值应遵循特定的命名约定：

  * **“bitrate”** ：此值应用于视频和音频编码器，表示比特率，单位为千比特。
  * **“rate_control”** ：此设置用于视频编码器。通常至少应有一个 “CBR” 速率控制。其他常见的速率控制有 “VBR”、“CQP”。
  * **“keyint_sec”** ：对于视频编码器，设置关键帧间隔值，单位为秒，或尽可能接近的近似值。（作者注：这应该命名为 “keyint”，以帧为单位。）

编码器示例：

  * 视频编码器：

    * x264 编码器
    * FFmpeg NVENC 编码器
    * Quicksync 编码器

  * 音频编码器：

    * FFmpeg AAC/Opus 编码器

## 服务

服务是流媒体服务的自定义实现，用于与流媒体输出配合使用。例如，你可以为 Twitch 实现一个自定义服务，也可以为 YouTube 实现一个自定义服务，以允许登录并使用它们的 API 执行诸如获取 RTMP 服务器或控制频道等操作。libobs/obs-service.h 文件是实现服务的专用头文件。有关更多信息，请参阅服务 API 参考（obs_service_t）。

（作者注：截至本文撰写时，服务 API 尚未完善）

例如，要实现一个服务对象，你需要定义一个 `obs_service_info` 结构并填写与你的服务相关的信息和回调函数：

```
/* my-service.c */

[...]

struct obs_service_info my_service_service = {
        .id       = "my_service",
        .get_name = my_service_name,
        .create   = my_service_create,
        .destroy  = my_service_destroy,
        .encode   = my_service_encode,
        .update   = my_service_update,
        .get_url  = my_service_url,
        .get_key  = my_service_key
};
```

然后，在 my-plugin.c 中，你将在 `obs_module_load()` 中调用 `obs_register_service()` 来向 libobs 注册服务。

```
/* my-plugin.c */

[...]

extern struct obs_service_info my_service; /* 在 my-service.c 中定义 */

bool obs_module_load(void)
{
        obs_register_service(&my_service);

        [...]

        return true;
}
```

目前唯一存在的两个服务对象是 “common RTMP services” 和 “custom RTMP service” 对象，位于 plugins/rtmp-services 中。

## 设置

设置（参阅 libobs/obs-data.h）用于获取或设置通常与 libobs 对象相关联的设置数据，然后可以通过 Json 文本保存和加载。有关更多信息，请参阅数据设置 API 参考（obs_data_t）。

`obs_data_t` 相当于一个 Json 对象，是一个子对象的字符串表，而 `obs_data_array_t` 则用于存储 `obs_data_t` 对象数组，类似于 Json 数组（尽管不完全相同）。

要创建一个 `obs_data_t` 或 `obs_data_array_t` 对象，你可以调用 `obs_data_create()` 或 `obs_data_array_create()` 函数。`obs_data_t` 和 `obs_data_array_t` 对象是引用计数的，因此当你不再需要该对象时，需要调用 `obs_data_release()` 或 `obs_data_array_release()` 来释放这些引用。每当函数返回一个 `obs_data_t` 或 `obs_data_array_t` 对象时，它们的引用计数都会增加，因此每次都需要释放这些引用。

要为 `obs_data_t` 对象设置值，你可以使用以下函数之一：

```
/* 设置函数 */
EXPORT void obs_data_set_string(obs_data_t *data, const char *name, const char *val);
EXPORT void obs_data_set_int(obs_data_t *data, const char *name, long long val);
EXPORT void obs_data_set_double(obs_data_t *data, const char *name, double val);
EXPORT void obs_data_set_bool(obs_data_t *data, const char *name, bool val);
EXPORT void obs_data_set_obj(obs_data_t *data, const char *name, obs_data_t *obj);
EXPORT void obs_data_set_array(obs_data_t *data, const char *name, obs_data_array_t *array);
```

同样，要从 `obs_data_t` 对象获取值，你可以使用以下函数之一：

```
/* 获取函数 */
EXPORT const char *obs_data_get_string(obs_data_t *data, const char *name);
EXPORT long long obs_data_get_int(obs_data_t *data, const char *name);
EXPORT double obs_data_get_double(obs_data_t *data, const char *name);
EXPORT bool obs_data_get_bool(obs_data_t *data, const char *name);
EXPORT obs_data_t *obs_data_get_obj(obs_data_t *data, const char *name);
EXPORT obs_data_array_t *obs_data_get_array(obs_data_t *data, const char *name);
```

与典型的 Json 数据对象不同，`obs_data_t` 对象还可以设置默认值。这允许在从 Json 字符串或 Json 文件加载数据时，控制在特定字符串未分配值的情况下返回的内容。每个 libobs 对象都有一个 `_get_defaults` 回调，用于在创建对象时设置其默认设置。

以下函数用于控制默认值：

```
/* 默认值函数。 */
EXPORT void obs_data_set_default_string(obs_data_t *data, const char *name, const char *val);
EXPORT void obs_data_set_default_int(obs_data_t *data, const char *name, long long val);
EXPORT void obs_data_set_default_double(obs_data_t *data, const char *name, double val);
EXPORT void obs_data_set_default_bool(obs_data_t *data, const char *name, bool val);
EXPORT void obs_data_set_default_obj(obs_data_t *data, const char *name, obs_data_t *obj);
```

## 属性

属性（参阅 libobs/obs-properties.h）用于自动生成用户界面以修改 libobs 对象的设置（如果需要）。每个 libobs 对象都有一个 `_get_properties` 回调，用于生成属性。属性 API 定义了与对象设置相关联的特定属性，前端使用这些属性生成小部件，以便用户可以修改设置。例如，如果你有一个布尔设置，你可以使用 `obs_properties_add_bool()` 来允许用户更改该设置。有关更多信息，请参阅属性 API 参考（obs_properties_t）。

以下是一个示例：

```
static obs_properties_t *my_source_properties(void *data)
{
        obs_properties_t *ppts = obs_properties_create();
        obs_properties_add_bool(ppts, "my_bool",
                        obs_module_text("MyBool"));
        UNUSED_PARAMETER(data);
        return ppts;
}

[...]

struct obs_source_info my_source {
        .get_properties = my_source_properties,
        [...]
};
```

`data` 参数是对象的数据（如果对象存在）。通常这个参数未使用，可能的话不应使用。如果在没有与之关联的对象的情况下检索属性，它可以是 null。

根据显示的设置，属性也可以被修改。例如，你可以根据某个特定设置的值，使用 `obs_property_set_modified_callback()` 函数标记某些属性为禁用或不可见。

例如，如果你希望布尔属性 A 隐藏文本属性 B：

```
static bool setting_a_modified(obs_properties_t *ppts,
                obs_property_t *p, obs_data_t *settings)
{
        bool enabled = obs_data_get_bool(settings, "setting_a");
        p = obs_properties_get(ppts, "setting_b");
        obs_property_set_enabled(p, enabled);

        /* 返回 true 以更新属性小部件，否则返回 false */
        return true;
}

[...]

static obs_properties_t *my_source_properties(void *data)
{
        obs_properties_t *ppts = obs_properties_create();
        obs_property_t *p;

        p = obs_properties_add_bool(ppts, "setting_a",
                        obs_module_text("SettingA"));
        obs_property_set_modified_callback(p, setting_a_modified);

        obs_properties_add_text(ppts, "setting_b",
                        obs_module_text("SettingB"),
                        OBS_TEXT_DEFAULT);
        return ppts;
}
```

## 本地化

通常，与 OBS Studio 捆绑的大多数插件将使用简单的 ini 文件本地化方法，每个文件对应一种不同的语言。当使用这种方法时，`OBS_MODULE_USE_DEFAULT_LOCALE()` 宏将被使用，它将自动加载和销毁本地化数据，无需插件额外的努力。然后在需要文本查找时使用 `obs_module_text()` 函数（该函数由 libobs/obs-module.h 自动声明为 extern）。

模块用于加载和销毁本地化的两个导出是：`obs_module_set_locale()` 导出和 `obs_module_free_locale()` 导出。`obs_module_set_locale()` 导出由 libobs 调用以设置当前语言，然后 `obs_module_free_locale()` 导出由 libobs 在模块销毁时调用。如果你希望为你的插件实现自定义本地化，你需要自己定义这些导出以及 `obs_module_text()` extern，而不是依赖 `OBS_MODULE_USE_DEFAULT_LOCALE()` 宏。

---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
{: .prompt-tip }

