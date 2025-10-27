---
title: Xcode 再见吧！iOS 开发这样操作来投向 Cursor AI 开发的怀抱
description: 使用 Cursor AI 开发 iOS 项目的经验。
author: Keyframe
date: 2025-04-07 08:08:08 +0800
categories: [AI 编程]
tags: [AI 编程, AI, AIGC, Cursor]
pin: false
math: true
mermaid: true
---



>想要学习 AI 技术的朋友，快来加入我们的<a href="https://t.zsxq.com/nd3Wj" target="_blank" rel="noopener noreferrer">【AI 赚钱社群】</a>，加入后你就能：
>
>- 1）获得全套 Stable Diffusion WebUI AI 图片和视频制作教程
>- 2）获得全套 ComfyUI AI 图片和视频制作教程
>- 3）获得自动化批量生产 AI 套图的程序
>- 4）获得 <a href="https://aigchub.ai" target="_blank" rel="noopener noreferrer">AIGCHub</a> 创作者资格，用 AI 生产图片和视频来赚钱
>
>现在加入，送你一张 20 元优惠券：<a href="https://t.zsxq.com/nd3Wj" target="_blank" rel="noopener noreferrer">点击领取优惠券</a>
>
>![知识星球新人优惠券](assets/img/aigc-zsxq-coupon.png){: w="300" }
>_微信扫码也可领取优惠券_
{: .prompt-tip }

---







随着 Cursor 等 AI 开发工具不断被大家所接受，这里我们来介绍一下如何正确配置 Cursor/VSCode 来使用它进行流畅的 iOS 开发，拥抱 AI 开发的福利。


## 为什么选择用 Cursor/VSCode 开发 iOS 应用？

最主要还是 Xcode 不争气啊！Xcode 作为苹果官方的开发工具，虽然与苹果整个开发体系融合得更深入，但是也存在很多不足：AI 功能缺失、预览卡顿、界面布局臃肿。这些问题让许多 iOS 开发者常常一边狠敲键盘、一边骂娘。Xcode 在 AppStore 长期以来艰难维持着不到 3 星的打分。


![](assets/resource/aigc-programming/idwc-1.png)


为了改善开发体验，Cursor/VSCode 成为了许多 iOS 开发者的新选择。特别是 Cursor，在交互逻辑和 AI 模型上表现出色，能够助力开发者更高效地编写代码。


## 在 Cursor/VSCode 中配置 iOS 开发环境


**（1）安装 Swift 插件**

Swift 插件提供了 Swift 语言的基本支持。

- Swift 插件：`https://marketplace.visualstudio.com/items?itemName=swiftlang.swift-vscode`


**（2）安装 CodeLLDB 插件**

CodeLLDB 插件提供了调试功能。

- CodeLLDB 插件：`https://marketplace.visualstudio.com/items?itemName=vadimcn.vscode-lldb`


**（3）安装 SweetPad 插件**

SweetPad 插件可用于配置在编辑器中代码格式化、调试等功能，自行探索配置。

- SweetPad 插件：`https://marketplace.visualstudio.com/items?itemName=sweetpad.sweetpad`

事实上大部分配置都可以通过 SweetPad 完成，但是也有失败的情况，所以下面具体介绍详细的配置方法。


**（4）使用 Xcode-Build-Server 编译项目**

首先安装 xcode-build-server：

```
brew install xcode-build-server
```

然后在项目根目录下根据你的项目文件类型执行对应的命令：

```
xcode-build-server config -workspace *.xcworkspace -scheme <XXX> 
xcode-build-server config -project *.xcodeproj -scheme <XXX>
```

例如你用的是 `exampleProject.xcodeproj`：

```
xcode-build-server config -project exampleProject.xcodeproj -scheme exampleProject
```

重新启动编辑器，即可开始享受用 Cursor/VSCode 开发 iOS 项目，赶快试试丝滑的 AI 开发体验吧！


**（5）设置热加载**

使用 Cursor/VSCode 开发 iOS 项目的一个问题就是无法在使用 Xcode 的预览功能，导致 UI 调试效率低下。然而我们可以通过热加载来解决这个问题，实时预览 UI 变化，体验甚至比 Xcode 的更加好用。

设置热加载的方法有很多，以下是一种方案：


- 1、在 Xcode 中添加 Inject package：`https://github.com/krzysztofzablocki/Inject`。
- 2、打开项目对应 Target 的设置 -> Build Settings -> 搜索 Other Linker Flags，分别添加 `-Xlinker` 和 `-interposable`。
- 3、下载并安装 InjectionIII 最新版：`https://github.com/johnno1962/InjectionIII/releases/`。
- 4、打开 InjectionIII，它会在右上角菜单栏中显示一个小图标，选择项目的目录，再次点击小图标，选择 Prepare Project，为项目中的 SwiftUI 文件添加注入代码。可以选择为所有 View 添加，也可以手动添加。

>Prepare Project 会给所有的 View 添加 .enableInjection() 方法和 @ObserveInjection var forceRedraw，如果不想一次性给所有的 View 添加，可以在需要的 View 中手动添加。

- 5、在 Xcode 中编译运行项目，当 console 显示 InjectionIII 连接成功信息后，即可在 Cursor/VSCode 中修改 SwiftUI 文件并实时预览 UI 变化。


目前体验下来这套方法非常好用， UI 实时更新，不用每次都重新编译运行项目，也不用等 Preview 加载，大大提高了开发效率。

如有问题或想了解更多关于热加载的内容可以参考 Inject 的文档。




## 常见问题及解决方案

**（1）Swift LSP无法正确识别项目代码？**

如果编辑器报错提示找不到某些方法或变量，可能是因为 swift 的 sourcekit-lsp 没有将项目文件加入索引。可以尝试重新构建项目索引或重启编辑器。


**（2）代码格式化如何设置？**


首次配置一下默认格式化就行了，当然你也可以选择其他插件来做格式化。

使用 SweetPad 插件安装对应的格式化插件（如 swift-format）即可实现代码自动格式化。


**（3）Xcode 和 Cursor/VSCode 文件结构不一致？**

这里最好保持一致，不然 VSCode 里新建文件，项目里找不到。

将 Xcode 中的所有 Group 转变成 Folder，这样在 Cursor/VSCode 中创建的文件/文件夹就会在 Xcode 中显示。


**（4）该方案的其他使用问题**

- 热更新能力有限，需要频繁重启。特别是社会布局结构的时候，可以试试其他热更新方案。
- 有一定的代码入侵性，上架前需要移除注入代码。

以上的问题都源于热更新，如果能有更好的方案就完美了。不用来回切换 XCode 与 Cursor 真的太棒了。



## 参考

- [在 Cursor 打造高效 iOS 开发环境： AI 编程 + 实时预览完整指南](https://blog.imjp.uk/fxxk-xcode)













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

