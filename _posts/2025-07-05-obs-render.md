---
title: 探索 OBS 开发（5）：OBS 的图形渲染
description: 系统介绍 OBS 开发相关的基础技术。
author: Keyframe
date: 2025-07-05 18:08:08 +0800
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




Libobs 拥有一个定制的可编程图形子系统，它封装了 Direct3D 11 和 OpenGL。设计这样一个定制图形子系统的原因是为了适应特定操作系统上独有的自定义捕获功能。

（作者注：事后看来，我可能应该使用类似 ANGLE 的东西，但我必须对其进行修改以适应我的特定用例。）

大多数渲染依赖于效果。效果被 libobs 中的所有视频对象使用；它们用于轻松地将相关的顶点/像素着色器捆绑到一个文件中。

效果文件的语法与 Direct3D 11 HLSL 效果文件几乎完全相同。唯一的区别如下：

- 采样器状态被命名为 “sampler\_state”
- 位置语义被称为 “POSITION” 而不是 “SV\_Position”
- 目标语义被称为 “TARGET” 而不是 “SV\_Target”

（作者注：我可能遗漏了一些例外情况，如果有的话请告诉我）

## 图形上下文

除非当前线程已进入图形上下文，否则无法使用图形函数，并且图形上下文一次只能被一个线程使用。要进入图形上下文，使用 `obs_enter_graphics()`，要离开图形上下文，使用 `obs_leave_graphics()`。

某些回调会自动处于图形上下文中：`obs_source_info.video_render` 以及 `obs_display_add_draw_callback()` 的绘制回调参数和 `obs_add_main_render_callback()`。

## 创建效果

### 效果参数

创建效果时，建议从效果的统一变量（参数）开始。

有多种不同类型的统一变量：

|  |  |  |  |  |
|---|---|---|---|---|
| 浮点数：| **float**| **float2**| **float3**| **float4**|
| 矩阵：| **float3x3**| **float4x4**| | |
| 整数：| **int**| **int2**| **int3**| **int4**|
| 布尔值：| **bool**| | | |
| 纹理：| **texture2d**| **texture\_cube**| | |

要获取效果统一变量参数，使用 `gs_effect_get_param_by_name()` 或 `gs_effect_get_param_by_idx()`。

然后通过以下函数设置统一变量：

- `gs_effect_set_bool()`
- `gs_effect_set_float()`
- `gs_effect_set_int()`
- `gs_effect_set_matrix4()`
- `gs_effect_set_vec2()`
- `gs_effect_set_vec3()`
- `gs_effect_set_vec4()`
- `gs_effect_set_texture()`
- `gs_effect_set_texture_srgb()`


有两种 “通用” 效果参数可能被期望：**ViewProj** 和 **image**。**ViewProj** 参数（是一个 float4x4）用于主视图/投影矩阵组合。**image** 参数（是一个 texture2d）是一个常用参数，用于主要纹理；此参数将与函数 `obs_source_draw()`、`gs_draw_sprite()` 和 `obs_source_process_filter_end()` 一起使用。

以下是一个效果参数的例子：

```
uniform float4x4 ViewProj;
uniform texture2d image;

uniform float4 my_color_param;
uniform float my_float_param;

```

效果参数也可以有默认值。具有多个元素的元素的默认参数应被视为数组。

以下是一些默认参数的例子：

```
uniform float4x4 my_matrix = {1.0, 0.0, 0.0, 0.0,
                              0.0, 1.0, 0.0, 0.0,
                              0.0, 0.0, 1.0, 0.0,
                              0.0, 0.0, 0.0, 1.0};

uniform float4 my_float4 = {1.0, 0.5, 0.25, 0.0};
uniform float my_float = 4.0;
uniform int my_int = 5;

```

### 效果采样器状态

然后，如果使用纹理，应定义采样器状态。采样器状态具有某些子参数：

- **Filter** - 要使用的过滤器类型。可以是以下值之一：
  - **Anisotropy**
  - **Point**
  - **Linear**
  - **MIN\_MAG\_POINT\_MIP\_LINEAR**
  - **MIN\_POINT\_MAG\_LINEAR\_MIP\_POINT**
  - **MIN\_POINT\_MAG\_MIP\_LINEAR**
  - **MIN\_LINEAR\_MAG\_MIP\_POINT**
  - **MIN\_LINEAR\_MAG\_POINT\_MIP\_LINEAR**
  - **MIN\_MAG\_LINEAR\_MIP\_POINT**
- **AddressU**, **AddressV** - 指定当坐标超出 0.0..1.0 时如何处理采样。可以是以下值之一：
  - **Wrap** 或 **Repeat**
  - **Clamp** 或 **None**
  - **Mirror**
  - **Border**（使用 _BorderColor_ 填充颜色）
  - **MirrorOnce**
- **BorderColor** - 如果使用 “Border” 寻址模式，则指定边框颜色。这个值应该是一个表示颜色的十六进制值，格式为：AARRGGBB。例如，7FFF0000 的 alpha 值为 127，红色值为 255，蓝色和绿色为 0。如果未使用 _Border_ 作为寻址类型，则忽略此值。


以下是在效果文件中编写采样器状态的例子：

```
sampler_state defaultSampler {
        Filter      = Linear;
        AddressU    = Border;
        AddressV    = Border;
        BorderColor = 7FFF0000;
};

```

此采样器状态将使用线性过滤，对于超出 0.0..1.0 的纹理坐标值，将使用边框寻址，边框颜色为上面指定的颜色。

当使用采样器状态时，其使用方式与 HLSL 形式完全相同：

```
[...]

uniform texture2d image;

sampler_state defaultSampler {
        Filter      = Linear;
        AddressU    = Clamp;
        AddressV    = Clamp;
};

[...]

float4 MyPixelShaderFunc(VertInOut vert_in) : TARGET
{
        return image.Sample(def_sampler, vert_in.uv);
}

```

### 效果顶点/像素语义

然后应定义输入和输出顶点语义的结构体。

顶点组件可以具有以下语义：

- **COLOR** - 颜色值（_float4_）。
- **POSITION** - 位置值（_float4_）。
- **NORMAL** - 法线值（_float4_）。
- **TANGENT** - 切线值（_float4_）。
- **TEXCOORD\[0..7\]** - 纹理坐标值（_float2_, _float3_, 或 _float4_）。

以下是一个顶点语义结构体的例子：

```
struct VertexIn {
        float4 my_position : POSITION;
        float2 my_texcoord : TEXCOORD0;
};

```

然后这些语义结构体作为参数传递给主着色器入口点，并用作顶点着色器的返回值。注意，顶点着色器可以返回与它接收的不同语义；但是顶点着色器的返回类型和像素着色器的参数必须匹配。

用于顶点着色器函数参数的语义结构体将要求顶点缓冲区具有这些值，所以如果你有 POSITION 和 TEXCOORD0，顶点缓冲区将需要至少有一个位置缓冲区和一个纹理坐标缓冲区。

对于像素着色器，它们需要以 TARGET 语义返回（这是一个 float4 RGBA 值）。以下是一个像素着色器函数通常如何使用的例子：

```
float4 MyPixelShaderFunc(VertInOut vert_in) : TARGET
{
        return image.Sample(def_sampler, vert_in.uv);
}

```

### 效果技术

技术用于定义每个通道的主顶点/像素着色器入口函数。一个技术可以有多个通道或自定义通道设置。

（作者注：如今，多个通道并不真正需要；GPU 足够强大，可以在同一个着色器中执行所有操作。命名通道对于自定义绘制设置可能有用，但即使这样，你也可以创建一个单独的技术。因此，最好忽略额外的通道功能。）

如果你正在为视频源创建效果滤镜，通常会将通道命名为 **Draw**，然后 `obs_source_process_filter_end()` 将自动调用该特定效果名称。然而，你也可以使用 `obs_source_process_filter_tech_end()` 使滤镜使用特定名称的技术。

通道中顶点/像素着色器函数的第一个参数始终应该是其顶点语义结构体参数的名称。

以下是一些技术如何使用的例子：

```
uniform float4x4 ViewProj;
uniform texture2d image;

struct VertInOut {
        float4 my_position : POSITION;
        float2 my_texcoord : TEXCOORD0;
};

VertInOut MyVertexShaderFunc(VertInOut vert_in)
{
        VertInOut vert_out;
        vert_out.pos = mul(float4(vert_in.pos.xyz, 1.0), ViewProj);
        vert_out.uv  = vert_in.uv;
        return vert_out;
}

float4 MyPixelShaderFunc(VertInOut vert_in) : TARGET
{
        return image.Sample(def_sampler, vert_in.uv);
}

technique Draw
{
        pass
        {
                vertex_shader = MyVertexShaderFunc(vert_in);
                pixel_shader  = MyPixelShaderFunc(vert_in);
        }
};

```

## 使用效果

使用效果的推荐方式如下：

```
for (gs_effect_loop(effect, "technique")) {
        [绘制调用放在这里]
}

```

这将自动处理效果及其着色器的加载和卸载，针对给定的技术名称。

## 渲染视频源

同步视频源在其 `obs_source_info.video_render` 回调中渲染。

源可以使用自定义绘制（通过 OBS\_SOURCE\_CUSTOM\_DRAW 输出能力标志）或不使用自定义绘制进行渲染。当源不使用自定义渲染进行渲染时，建议使用 `obs_source_draw()` 渲染一个单纹理。否则，源应自行执行渲染并管理其自己的效果。

Libobs 提供了一组可以通过 `obs_get_base_effect()` 函数访问的默认/标准效果。你可以使用这些效果进行渲染，或者使用 `gs_effect_create_from_file()` 创建自定义效果，并使用自定义效果进行渲染。

## 渲染视频效果滤镜

对于大多数视频效果滤镜，它包括在 `obs_source_info.video_render` 回调中向现有图像添加一层处理着色器。在这种情况下，期望滤镜拥有自己的效果，并且为了绘制效果，只需使用 `obs_source_process_filter_begin()` 函数，设置自定义效果的参数，然后调用 `obs_source_process_filter_end()` 或 `obs_source_process_filter_tech_end()` 完成滤镜的渲染。

以下是颜色键滤镜中渲染滤镜的例子：

```
static void color_key_render(void *data, gs_effect_t *effect)
{
        struct color_key_filter_data *filter = data;

        if (!obs_source_process_filter_begin(filter->context, GS_RGBA,
                                OBS_ALLOW_DIRECT_RENDERING))
                return;

        gs_effect_set_vec4(filter->color_param, &filter->color);
        gs_effect_set_float(filter->contrast_param, filter->contrast);
        gs_effect_set_float(filter->brightness_param, filter->brightness);
        gs_effect_set_float(filter->gamma_param, filter->gamma);
        gs_effect_set_vec4(filter->key_color_param, &filter->key_color);
        gs_effect_set_float(filter->similarity_param, filter->similarity);
        gs_effect_set_float(filter->smoothness_param, filter->smoothness);

        obs_source_process_filter_end(filter->context, filter->effect, 0, 0);

        UNUSED_PARAMETER(effect);
}

```

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

