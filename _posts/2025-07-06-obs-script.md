---
title: 探索 OBS 开发（6）：OBS 的脚本编程
description: 系统介绍 OBS 开发相关的基础技术。
author: Keyframe
date: 2025-07-06 18:08:08 +0800
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





脚本编程 (21.0+) 增加了对 Python 3 和 Luajit 2（大致相当于 Lua 5.2）的支持，允许快速扩展、添加功能或自动化程序，无需构建原生插件模块。

可以通过 OBS Studio 的工具菜单中的脚本选项访问脚本编程，这将弹出脚本对话框。脚本可以在程序运行时实时添加、移除和重新加载。

**注意：** 在 Windows 或 macOS 上使用 Python 时，必须下载并安装与 OBS 架构匹配的 Python 版本。然后，在脚本对话框中，必须在 “Python 设置” 选项卡中设置 Python 安装路径。

所有 API 绑定在 Python 中通过 **obspython** 模块提供，在 Lua 中通过 **obslua** 模块提供。

为了提供脚本回调，某些函数已被更改/替换，更多信息请参见与 C API 的其他差异。

**警告：** 由于提供了整个 API 的绑定，使用编写不当的脚本可能会导致内存泄漏或程序崩溃。请在编写脚本时小心谨慎，并检查日志文件中的内存泄漏计数器，以确保您编写的脚本没有内存泄漏。**请将 API 绑定视为编写 C 程序：阅读您所使用函数的文档，并通过 API 引用或创建的对象进行释放/销毁。**

## 脚本功能导出

脚本可以提供以下多个全局函数：

### script_description()

被调用以获取要在脚本窗口中显示给用户的描述字符串。

### script_load(settings)

在脚本启动时被调用，具有与脚本相关的特定设置。提供的 settings 参数通常不用于用户设置的设置；相反，该参数用于脚本可能使用的任何额外内部设置数据。

参数：

**settings** – 与脚本相关的设置。

### script_unload()

在脚本被卸载时被调用。

### script_save(settings)

在脚本被保存时被调用。这对于用户设置的设置不是必需的；相反，这是用于脚本可能使用的任何额外内部设置数据。

参数：

**settings** – 与脚本相关的设置。

### script_defaults(settings)

被调用以设置与脚本相关的默认设置（如果有）。通常会为设置调用默认值函数，以设置其默认值。

参数：

**settings** – 与脚本相关的设置。

### script_update(settings)

在用户更改脚本的设置（如果有）时被调用。

参数：

**settings** – 与脚本相关的设置。

### script_properties()

被调用以定义与脚本关联的用户属性。这些属性用于定义如何向用户显示设置属性。

返回值：

通过 `obs_properties_create()` 创建的 obs_properties_t 对象。

### script_tick(seconds)

在每一帧中被调用，以满足每帧处理的需要。如果需要计时器，请使用脚本计时器，因为如果只需要基本计时器功能，计时器效率更高。不推荐在 Python 中使用此函数，因为 Python 的全局解释器锁。

参数：

**seconds** – 自上一帧以来经过的秒数。

## 获取当前脚本的路径

有一个可以用来获取当前脚本路径的函数。此函数在脚本加载之前自动实现到每个脚本中，并且是脚本名称空间的一部分，而不是 obslua/obspython 的一部分：

### script_path()

返回值：

脚本的路径。

## 脚本计时器

脚本计时器提供了一种有效的计时器回调方式，无需每帧锁定脚本/解释器。

（这些函数是 obspython/obslua 模块/名称空间的一部分）。

### timer_add(callback, milliseconds)

添加一个计时器回调，该回调每 milliseconds 触发一次。

注意：不支持将实例方法用作回调。请始终使用模块方法。

### timer_remove(callback)

移除计时器回调。（注意：你也可以使用 `remove_current_callback()` 从计时器回调中终止计时器）

## 脚本源（仅限 Lua）

可以在 Lua 中注册源。为此，创建一个表，并像定义 `obs_source_info` 结构一样定义其键：

```
local info = {}
info.id = "my_source_id"
info.type = obslua.OBS_SOURCE_TYPE_INPUT
info.output_flags = obslua.OBS_SOURCE_VIDEO

info.get_name = function()
        return "My Source"
end

info.create = function(settings, source)
        -- 通常源数据将作为表存储
        local my_source_data = {}

        [...]

        return my_source_data
end

info.video_render = function(my_source_data, effect)
        [...]
end

info.get_width = function(my_source_data)
        [...]

        -- 假设源数据包含 'width' 键
        return my_source_data.width
end

info.get_height = function(my_source_data)
        [...]

        -- 假设源数据包含 'height' 键
        return my_source_data.height
end

-- 注册源
obs_register_source(info)
```

## 与 C API 的其他差异

由于回调的工作方式，某些函数的实现与 C API 不同。（这些函数是 obspython/obslua 模块/名称空间的一部分）。

### obs_enum_sources()

枚举所有源。

返回值：

引用计数增加的源数组。使用 `source_list_release()` 释放。

### obs_scene_enum_items(scene)

枚举场景中的场景项。

参数：

**scene** – 要枚举项的 obs_scene_t 对象。

返回值：

场景项列表。使用 `sceneitem_list_release()` 释放。

### obs_sceneitem_group_enum_items(group)

枚举组中的场景项。

参数：

**group** – 要枚举项的 obs_sceneitem_t 对象。

返回值：

场景项列表。使用 `sceneitem_list_release()` 释放。

### obs_add_main_render_callback(callback)

**仅限 Lua：** 添加主输出渲染回调。此回调没有参数。

参数：

**callback** – 渲染回调。使用 `obs_remove_main_render_callback()` 或 `remove_current_callback()` 移除回调。

### obs_remove_main_render_callback(callback)

**仅限 Lua：** 移除主输出渲染回调。

参数：

**callback** – 要移除的热键回调。

### signal_handler_connect(handler, signal, callback)

将回调添加到信号处理程序上的特定信号。此回调有一个参数：calldata_t 对象。

参数：

- **handler** – 信号处理程序对象。

- **signal** – 信号处理程序上的信号（字符串）。

- **callback** – 要连接到信号的回调。使用 `signal_handler_disconnect()` 或 `remove_current_callback()` 移除回调。

### signal_handler_disconnect(handler, signal, callback)

从信号处理程序上的特定信号移除回调。

参数：

- **handler** – 信号处理程序对象。

- **signal** – 信号处理程序上的信号（字符串）。

- **callback** – 要断开连接的回调。

### signal_handler_connect_global(handler, callback)

将全局回调添加到信号处理程序。此回调有两个参数：第一个参数是信号字符串，第二个参数是 calldata_t 对象。

参数：

- **handler** – 信号处理程序对象。

- **callback** – 要连接的回调。使用 `signal_handler_disconnect_global()` 或 `remove_current_callback()` 移除回调。

### signal_handler_disconnect_global(handler, callback)

从信号处理程序移除全局回调。

参数：

- **handler** – 信号处理程序对象。

- **callback** – 要断开连接的回调。

### obs_hotkey_register_frontend(name, description, callback)

添加前端热键。回调有一个参数：表示 “按下” 参数的布尔值。

参数：

- **name** – 热键的唯一名称标识符字符串。

- **description** – 显示给用户的热键描述。

- **callback** – 热键回调。使用 `obs_hotkey_unregister()` 或 `remove_current_callback()` 移除回调。

### obs_hotkey_unregister(callback)

注销与指定回调关联的热键。

参数：

**callback** – 要注销的热键回调。

### obs_properties_add_button(properties, setting_name, text, callback)

向 obs_properties_t 对象添加按钮属性。回调有两个参数：第一个参数是 obs_properties_t 对象，第二个参数是按钮的 obs_property_t。

参数：

- **properties** – obs_properties_t 对象。

- **setting_name** – 设置标识符字符串。

- **text** – 按钮文本。

- **callback** – 按钮回调。此回调将自动清理。

### remove_current_callback()

移除正在执行的当前回调。如果不在回调中，则不执行任何操作。

### source_list_release(source_list)

释放源列表的引用。

参数：

**source_list** – 要释放的源数组。

### sceneitem_list_release(item_list)

释放场景项列表的引用。

参数：

**item_list** – 要释放的场景项数组。

### calldata_source(calldata, name)

将 calldata_t 对象的指针参数转换为 obs_source_t 对象。

参数：

- **calldata** – calldata_t 对象。

- **name** – 参数名称。

返回值：

对 obs_source_t 对象的借用引用。

### calldata_sceneitem(calldata, name)

将 calldata_t 对象的指针参数转换为 obs_sceneitem_t 对象。

参数：

- **calldata** – calldata_t 对象。

- **name** – 参数名称。

返回值：

对 obs_sceneitem_t 对象的借用引用。

---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频、AI 领域的最新技术和产品信息**：
>
>![微信公众号](assets/img/keyframe-mp.jpg){: w="300" }
>_微信扫码关注我们_
>
>你还可以加入我们的微信群：
>
>![关键帧的音视频开发群](assets/img/av-wechat-group.jpg){: w="600" }
>_微信扫码进群_
{: .prompt-tip }

