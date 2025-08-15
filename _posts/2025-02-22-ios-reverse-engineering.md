---
title: iOS 逆向工具：逆向做的好，码农下班早
description: 介绍常用的 iOS 客户端逆向工具。
author: Keyframe
date: 2025-02-22 12:08:08 +0800
categories: [音视频实用工具]
tags: [音视频实用工具, 音视频, Charles, Wireshark]
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


App 逆向工程是做竞品分析的常用方法，常言道『逆向做的好，码农下班早』，懂的都懂。这里我们对 iOS 逆向做一下简单介绍，这里面会涉及如下工具：

- [Theos](https://github.com/theos/theos "Theos")：一款基于 Make 的构建系统，主要用于iOS 越狱软件开发，也支持为其他支持平台构建软件。
- [MonkeyDev](https://github.com/AloneMonkey/MonkeyDev "MonkeyDev")：一款非越狱插件开发集成神器。
- [FLEX](https://github.com/Flipboard/FLEX "FLEX") ：一个探索和调试 iOS App UI 和堆栈的工具。
- [checkra1n](https://checkra.in/ "checkra1n")：一款基于 checkm8 漏洞的 iPhone 越狱工具。
- [frida-ios-dump](https://github.com/AloneMonkey/frida-ios-dump "frida-ios-dump")：一款 iOS App 砸壳工具。
- [usbmuxd](https://github.com/libimobiledevice/usbmuxd "usbmuxd")：一个套接字守护进程，可以用于多路复用来自和到 iOS 设备的连接。


## 1、非越狱 App 调试

### 1.1、环境配置

使用下列命令下载最新的 **Theos**：

```
sudo git clone --recursive https://github.com/theos/theos.git /opt/theos
```

### 1.2、安装 MonkeyDev

使用下列命令安装 **MonkeyDev**：

```
sudo /bin/sh -c "$(curl -fsSL https://raw.githubusercontent.com/AloneMonkey/MonkeyDev/master/bin/md-install)"
```

使用下列命令卸载 MonkeyDev：

```
sudo /bin/sh -c "$(curl -fsSL https://raw.githubusercontent.com/AloneMonkey/MonkeyDev/master/bin/md-uninstall)"
```

使用下列命令更新 MonkeyDev：

```
sudo /bin/sh -c "$(curl -fsSL https://raw.githubusercontent.com/AloneMonkey/MonkeyDev/master/bin/md-update)"
```

安装/更新之后需要重启下 Xcode 再新建项目。



MonkeyDev 主要包含四个模块：

- `Logos Tweak`：使用 Theos 提供的 logify.pl 工具将 `.xm`文件转成 `.mm` 文件进行编译，集成了 CydiaSubstrate，可以使用 MSHookMessageEx 和 MSHookFunction 来 Hook OC 函数和指定地址。
- `CaptainHook Tweak`：使用 CaptainHook 提供的头文件进行 OC 函数的 Hook 以及属性的获取。
- `Command-line Tool`：可以直接创建运行于越狱设备的命令行工具。
- `MonkeyApp`：这是自动给第三方应用集成 Reveal、Cycript 和注入 dylib 的模块，支持调试 dylib 和第三方应用，支持 Pod 给第三放应用集成 SDK，只需要准备一个砸壳后的 ipa 或者 App 文件即可。


### 1.3、使用 MonkeyApp 调试 App

这里主要介绍一下使用 MonkeyApp 调试 App。

App Store 里的应用都是加密的，直接拿上来是无法调试的，所以在此之前一般会有一个砸壳的过程。砸壳需要在越狱的环境下进行的，如果没有越狱机器和环境，那可以在一些其他平台上下载已经砸壳后的 App。

1）在 Xcode 中创建一个 MonkeyApp：

![](assets/resource/av-tool/monkey-app-1.png)

2）把砸壳后的 ipa 文件拖到 TargetApp 目录下：

![](assets/resource/av-tool/monkey-app-2.png)

这时候就运行项目就可以开始调试 App 了。

3）为了更好的调试 App 我们可以集成 FLEX 来做一些辅助，这需要我们在项目下增加一个 Podfile 并 `pod install` 一下。

![](assets/resource/av-tool/monkey-app-3.png)

Podfile 的内容如下：

```
target 'MyAppTestDylib' do
	pod 'FLEX', '~> 2.0'
end
```

同时需要在项目的 `MyAppTestDylib.m` 的增加代码：

```
#import "MyAppTestDylib.h"
#import <CaptainHook/CaptainHook.h>
#import <UIKit/UIKit.h>
#import <Cycript/Cycript.h>
#import <MDCycriptManager.h>
#import <FLEX/FLEXManager.h> // 引入头文件。

CHConstructor{
    printf(INSERT_SUCCESS_WELCOME);
    
    [[NSNotificationCenter defaultCenter] addObserverForName:UIApplicationDidFinishLaunchingNotification object:nil queue:[NSOperationQueue mainQueue] usingBlock:^(NSNotification * _Nonnull note) {
        
#ifndef __OPTIMIZE__
        CYListenServer(6666);

        MDCycriptManager* manager = [MDCycriptManager sharedInstance];
        [manager loadCycript:NO];

        NSError* error;
        NSString* result = [manager evaluateCycript:@"UIApp" error:&error];
        NSLog(@"result: %@", result);
        if(error.code != 0){
            NSLog(@"error: %@", error.localizedDescription);
        }
        
        [[FLEXManager sharedManager] showExplorer]; // 展示 FLEX 组件工具栏。
#endif
        
    }];
}
```

现在运行起来就可以看到 **FLEX** 工具栏了。



更多细节内容可以参考：[MonkeyDev Wiki](https://github.com/AloneMonkey/MonkeyDev/wiki "MonkeyDev Wiki")。


### 1.4、导出调试 App 的沙盒文件

通常我们会想要导出调试 App 的沙盒文件，这时候我们可以在 MonkeyApp 的 `Info.plist` 文件中添加 `Application supports iTunes file sharing` 并设置其为 `YES`。

![](assets/resource/av-tool/monkey-app-4.png)

这样我们就可以在 Finder 里面选中设备，在 Files 里拷贝 App 的 Document 文件夹的文件了。









## 2、越狱

上面说到要调试 App 需要砸壳后的 ipa 文件，而砸壳需要在越狱环境下进行，所以这里继续介绍一下如何越狱。

这里使用的越狱工具是 **checkra1n**。下载地址见：[checkra1n 下载](https://checkra.in/releases/0.12.4-beta#all-downloads "checkra1n 下载")。

越狱的过程，在下载安装工具后，照着工具的提示一步一步照做就行了。













## 3、砸壳


最早的砸壳工具是 [dumpdecrypted](https://github.com/stefanesser/dumpdecrypted "dumpdecrypted")，其原理是让 App 预先加载一个解密的 `dumpdecrypted.dylib`，然后在程序运行后将代码动态解密，最后在内存中 dump 出来整个程序。这种砸壳只能砸主 App 可执行文件。

对于应用程序里面存在 framework 的情况可以使用 conradev 的 dumpdecrypted，通过 `_dyld_register_func_for_add_image` 注册回调对每个模块进行 dump 解密。但是这种还是需要拷贝 `dumpdecrypted.dylib`，然后找路径什么的，还是挺麻烦的。

下面要介绍的是 **frida-ios-dump**，该工具基于 **frida** 提供的强大功能通过注入 js 实现内存 dump 然后通过 python 自动拷贝到电脑生成 ipa 文件，通过以下方式配置完成之后实现一条命令砸壳。


### 3.1、环境配置

首先要在手机和 Mac 电脑上面安装 frida，安装方式参考官网的文档：[frida home](https://www.frida.re/docs/home/ "frida home")。

1）手机端安装 frida：

- 手机越狱之后，`Cydia → 软件源 → 编辑 → 添加源(build.frida.re)`。
- 进入 `build.frida.re` 源下载 Frida。


2）Mac 端安装 frida：

```
$ sudo pip install frida
```



3）Mac 端安装 frida-ios-dump：


```
$ git clone https://github.com/AloneMonkey/frida-ios-dump.git
$ cd frida-ios-dump
$ sudo pip install -r requirements.txt --upgrade
```

这个安装过程可能会遇到一些依赖包版本不对的问题，可以按照提示安装符合要求的版本。

<!-- 

如果 Mac 端报如下错：

```
Uninstalling a distutils installed project (six) has been deprecated and will be removed in a future version. This is due to the fact that uninstalling a distutils project will only partially uninstall the project.
```

使用如下命令安装即可：

```
sudo pip install frida –upgrade –ignore-installed six
```
-->


### 3.2、连接手机

首先安装一下 **usbmuxd**，会自带一个 iproxy 的工具，我们用它来进行端口映射：

```
$ brew install usbmuxd
$ iproxy 2222 22
Creating listening port 2222 for device port 22
waiting for connection
New connection for 2222->22, fd = 5
waiting for connection
```

在我们的越狱手机上安装好 OpenSSH，然后在 Mac 上我们新开一个终端窗口，然后登陆到手机上：

```
$ ssh -p 2222 root@127.0.0.1 
// password:alpine // 这个密码参照越狱设备上 OpenSSH 的访问教程。
```


到此环境就配置好了，接下来就可以一键砸壳了。


### 3.3、一键砸壳

最简单的方式直接使用 `./dump + 应用显示的名字` 即可，如下：

```
$ cd frida-ios-dump
$ ./dump.py XXX
open target app......
Waiting for the application to open......
start dump target app......
start dump /var/containers/Bundle/Application/6665AA28-68CC-4845-8610-7010E96061C6/XXX.app/XXX
XXX                                        100%   68MB  11.4MB/s   00:05
start dump /private/var/containers/Bundle/Application/6665AA28-68CC-4845-8610-7010E96061C6/XXX.app/Frameworks/WCDB.framework/WCDB
WCDB                                          100% 2555KB  11.0MB/s   00:00
start dump /private/var/containers/Bundle/Application/6665AA28-68CC-4845-8610-7010E96061C6/XXX.app/Frameworks/MMCommon.framework/MMCommon
MMCommon                                      100%  979KB  10.6MB/s   00:00
start dump /private/var/containers/Bundle/Application/6665AA28-68CC-4845-8610-7010E96061C6/XXX.app/Frameworks/MultiMedia.framework/MultiMedia
MultiMedia                                    100% 6801KB  11.1MB/s   00:00
start dump /private/var/containers/Bundle/Application/6665AA28-68CC-4845-8610-7010E96061C6/XXX.app/Frameworks/mars.framework/mars
mars                                          100% 7462KB  11.1MB/s   00:00
AppIcon60x60@2x.png                           100% 2253   230.9KB/s   00:00
AppIcon60x60@3x.png                           100% 4334   834.8KB/s   00:00
AppIcon76x76@2x~ipad.png                      100% 2659   620.6KB/s   00:00
AppIcon76x76~ipad.png                         100% 1523   358.0KB/s   00:00
AppIcon83.5x83.5@2x~ipad.png                  100% 2725   568.9KB/s   00:00
Assets.car                                    100%   10MB  11.1MB/s   00:00
.......
AppIntentVocabulary.plist                     100%  197    52.9KB/s   00:00
AppIntentVocabulary.plist                     100%  167    43.9KB/s   00:00
AppIntentVocabulary.plist                     100%  187    50.2KB/s   00:00
InfoPlist.strings                             100% 1720   416.4KB/s   00:00
TipsPressTalk@2x.png                          100%   14KB   2.2MB/s   00:00
mm.strings                                    100%  404KB  10.2MB/s   00:00
network_setting.html                          100% 1695   450.4KB/s   00:00
InfoPlist.strings                             100% 1822   454.1KB/s   00:00
mm.strings                                    100%  409KB  10.2MB/s   00:00
network_setting.html                          100% 1819   477.5KB/s   00:00
InfoPlist.strings                             100% 1814   466.8KB/s   00:00
mm.strings                                    100%  409KB  10.3MB/s   00:00
network_setting.html                          100% 1819   404.9KB/s   00:00
```

如果存在应用名称重复了怎么办呢？没关系首先使用如下命令查看安装的应用的名字和 bundle id：

```
$ ./dump.py -l
  PID  Name                       Identifier
-----  -------------------------  ----------------------------------------
 9661  App Store                  com.apple.AppStore
16977  Moment                     com.kevinholesh.Moment
 1311  Safari                     com.apple.mobilesafari
16586  信息                         com.apple.MobileSMS
 4147  XXX                         com.XXX.YYY
10048  相机                         com.apple.camera
 7567  设置                         com.apple.Preferences
    -  CrashReporter              crash-reporter
    -  Cydia                      com.saurik.Cydia
    -  通讯录                        com.apple.MobileAddressBook
    -  邮件                         com.apple.mobilemail
    -  音乐                         com.apple.Music
    ......
```

然后使用如下命令对指定的 bundle id 应用进行砸壳即可：

```
$ ./dump.py -b com.XXX.YYY
```

等待自动砸壳传输完成之后便会到当前目录生成一个解密后的 ipa 文件。


更多的细节你还可以参考：[一条命令完成砸壳](https://www.alonemonkey.com/2018/01/30/frida-ios-dump/ "一条命令完成砸壳")。








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

