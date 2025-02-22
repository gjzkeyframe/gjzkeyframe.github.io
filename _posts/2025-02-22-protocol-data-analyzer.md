---
title: 数据抓包工具
description: 介绍常用的数据抓包工具。
author: Keyframe
date: 2025-02-22 11:08:08 +0800
categories: [音视频工具]
tags: [音视频工具, 音视频, Charles, Wireshark]
pin: false
math: true
mermaid: true
---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频**、**AI** 领域的最新技术和产品信息：
>
>![微信公众号](assets/img/keyframe-mp.jpg)
_微信扫码关注我们_
{: .prompt-tip }

数据抓包是我们做业务测试、竞品分析的常用方法，在直播、短视频等常见的音视频业务场景能有好的数据抓包工具帮助，很多时候也能事半功倍。这里我们就介绍两款常见的数据抓包工具：

- **Charles**
- **Wireshark**


## 1、Charles

[Charles](https://www.charlesproxy.com/ "Charles") 是在 macOS 上最常使用的 HTTP/HTTPS 数据抓包工具。下面是它的一些功能：

- 支持 SSL 代理。可以截取分析 SSL 的请求。
- 支持流量控制。可以模拟慢速网络以及等待时间（latency）较长的请求。
- 支持 AJAX 调试。可以自动将 json 或 xml 数据格式化，方便查看。
- 支持 AMF 调试。可以将 Flash Remoting 或 Flex Remoting 信息格式化，方便查看。
- 支持重发网络请求，方便后端调试。
- 支持修改网络请求参数。
- 支持网络请求的截获并动态修改。
- 检查 HTML、CSS 和 RSS 内容是否符合 W3C 标准。


我们在这里对重点配置做一下介绍：

**1）Mac 端代理设置**

安装好 Charles 后，在菜单栏勾选 `Proxy → macOS Proxy`，系统 HTTP/HTTPS 代理将会被自动设置为本地代理，默认端口 `8888`。

![Charles](assets/resource/av-tool/charles-3.png)


**2）Mac 端 HTTPS 证书设置**

在 Charles 菜单栏选择 `Help -> SSL Proxying -> Install Charles Root Certificate`，会自动导入 Charles Proxy CA 证书并打开 Keychain Access。

这里需要我们双击新导入的证书弹出证书信息页面，将 `Secure Sockets Layer (SSL)` 设置为 `Always Trust`，关闭页面后弹出密码提示，输入密码更新系统信任设置。

![Charles](assets/resource/av-tool/charles-4.png)


**3）Mac 端 HTTPS 设置**


在 Charles 菜单栏选择 `Proxy -> SSL Proxy Settings...`，在 `SSL Proxying` 选项卡中选中 `Enable SSL Proxying`，并添加需要抓包的域名端口。

![Charles](assets/resource/av-tool/charles-1.png)

这样才能解析 HTTPS 的数据。


**4）iPhone 手机端代理设置**

要对 iPhone 手机进行 HTTP/HTTPS 抓包，需要确保 iOS 设备和 Mac 设备处于同一局域网内。

然后，设置 iOS HTTP 代理：打开 iOS 设备对应 WIFI 设置，添加代理，设置 IP 地址为 Mac 的局域网地址，端口号设置为上面讲到的默认代理端口号 `8888`。

Mac 局域网地址可以在 Charles 中从菜单栏 `Help → Local IP Address` 获取。

![Charles](assets/resource/av-tool/charles-6.jpg)

设置完成后，在 iOS 设备上访问数据链接，Charles 弹出 Access Control 确认对话框，选择 Allow，可以开始抓取 HTTP 包。


**5）iPhone 手机端 HTTPS 设置**

如果想要对 iPhone 手机进行 HTTPS 抓包，还需要在手机上进行设置。

首先，在菜单栏选择 `Help → SSL Proxying → Install Charles Root Certificate on a Mobile Device or a Remote Browser`，弹出提示框。

![Charles](assets/resource/av-tool/charles-5.jpg)


根据上述提示，在 iOS 设备上使用 Safari 浏览器访问 `chls.pro/ssl`，Safari 浏览器会自动下载证书并提示安装，根据提示一步一步安装好，证书会被添加到`设置 → 通用 → 描述文件`中。

![Charles](assets/resource/av-tool/charles-7.jpg)

进入`设置 → 通用 → 关于本机 → 证书信任设置`，对上一步安装的 Charles 证书启用完全信任。

![Charles](assets/resource/av-tool/charles-8.jpg)

接下来，在 iOS 设备上访问 HTTPS 数据链接，可以开始抓取 HTTPS 包。


在使用 Charles 抓包时，我们可能会遇到解析不了 HTTPS 数据的情况，这是为什么呢？

如下图，当我们抓包解析不了 HTTPS 数据时：

![Charles](assets/resource/av-tool/charles-2.png)

提示显示：

```
SSL Proxying not enabled for this host: enable in Proxy Settings, SSL locations
```

这是因为我们没有添加对应的域名和端口，做如下操作即可：

在菜单栏选择 `Proxy -> SSL Proxy Settings...`，在 SSL Proxying 选项卡中可以添加需要抓包的域名端口。

![Charles](assets/resource/av-tool/charles-1.png)




## 2、Wireshark

上面讲到的 Charles 处理 HTTP/HTTPS 请求的数据抓包是强项，但是对于 RTMP、DNS、TCP、UDP 协议的数据抓包就无法胜任了。这时候我们可以使用 [Wireshark](https://www.wireshark.org/ "Wireshark")，下面是它的一些功能：


- 支持数百种协议，如：HTTP、RTMP、DNS、TCP、UDP，并持续更新中。
- 支持实时抓包和离线分析。
- 支持多平台，可以在 Windows、Linux、macOS、Solaris、FreeBSD、NetBSD 和许多其他平台上运行。
- 支持强大的显示过滤器。
- 支持丰富的 VoIP 分析功能。
- 支持读写多种不同的抓包存储文件格式，如：tcpdump(libpcap)、Pcap NG、Catapult DCT2000、Cisco Secure IDS iplog、Microsoft Network Monitor、Network General Sniffer 等等。
- 支持对捕获的 gzip 压缩文件进行即时解压缩。
- 支持实时数据可以从以太网、IEEE 802.11、PPP/HDLC、ATM、蓝牙、USB、令牌环、帧中继、FDDI 等读取。
- 支持对许多协议的解密支持，包括：IPsec、ISAKMP、Kerberos、SNMPv3、SSL/TLS、WEP 和 WPA/WPA2。
- 支持将着色规则应用于数据包列表，以进行快速、直观的分析。
- 支持输出导出为 XML、PostScript、CSV 或纯文本。


我们在这里对基本配置流程和常见抓包示例做一下介绍：


### 2.1、Wireshark 安装和配置


**1）Wireshark 下载和安装**

从 [Wireshark 官网下载页面](https://www.wireshark.org/download.html "Wireshark 官网下载页面")下载 Mac 版本的安装包安装即可。

需要注意的是下图中处理 app 外的其他两个 pkg 也需要安装一下：

![Wireshark](assets/resource/av-tool/wireshark-1.png)

安装成功后，打开 Wireshark 的界面如图：

![Wireshark](assets/resource/av-tool/wireshark-2.png)

这个界面列出了当前系统所使用的网卡，点击任何一项就可以开始监听了。


**2）手机连接配置**

要用 Wireshark 来抓包 iPhone 的数据，首先需要 iPhone 的数据经由 Mac OS 传输才行。可以通过 `rvictl` 来做到，`rvictl` 是一个可以连接设备来抓包的工具。


```
Remote Virtual Interface Tool starts and stops a remote packet capture instance for any set of attached mobile devices. It can also provide feedback on any attached devices that are currently relaying packets back to this host.
```



我们可以使用下面的命令来连接一个设备：


```
$ rvictl -s <iPhone UDID> 
```

对应的，可以使用下面的命令来断开一个设备：

```
$ rvictl -x <iPhone UDID> 
```



不过在使用 `rvictl` 工具之前，通常需要做一下安装：

```
$ cd /Applications/Xcode.app/Contents/Resources/Packages/
$ open .
```

双击安装 `MobileDevice.pkg` 和 `MobileDeviceDevelopment.pkg`。

在 Mac OS 10.15.0 以上，还需要注意 `rvictl` 命令的路径发生了改变，需要做配置：

```
在 `/etc/paths` 文件中增加一行路径 `/Library/Apple/usr/bin/`。保存后，重启终端。
```

完成上述操作以后，再使用

```
$ rvictl -s <iPhone UDID> 
```

就可以启动一个虚拟的网络接口，名字是 `rvi0`（如果是多台 iPhone 则数字累加，如：rvi1，rvi2 等）。


这时候再打开 Wireshark 就可以看到这个网络接口了：


![Wireshark](assets/resource/av-tool/wireshark-3.png)

双击就可以看到数据抓包了。



### 2.2、抓包示例

#### 2.2.1、TCP 三次握手抓包

先回顾一下 TCP 三次握手和四次挥手的基础知识：

![Wireshark](assets/resource/av-tool/wireshark-4.jpg)

接下来看一下 Wireshark 的抓包情况：

下图是第一次握手的抓包数据，这个包由客户端（172.16.146.107）发送给服务端（203.55.2.249），发送的 `SYN` 包的数据中 `Sequence number` 是 0。

![Wireshark](assets/resource/av-tool/wireshark-5.png)

下图是第二次握手的抓包数据，这个包由服务端（203.55.2.249）发送给客户端（172.16.146.107），发送的 `SYN + ACK` 包的数据中 `Sequence number` 是 0，`Acknowledgement number` 是 1。

![Wireshark](assets/resource/av-tool/wireshark-6.png)

 下图是第三次握手的抓包数据，这个包由客户端（172.16.146.107）发送给服务端（203.55.2.249），发送的 `ACK` 包的数据中 `Acknowledgement number` 是 1。

![Wireshark](assets/resource/av-tool/wireshark-7.png)


#### 2.2.2、RTMP 握手和协议控制消息抓包


下面是对直播 RTMP 推流的数据进行抓包的情况：

![Wireshark](assets/resource/av-tool/wireshark-8.png)

前面的 `Handshake C0+C1`、`Handshake C2`、`Handshake S0+S1+S2` 是握手消息；后面的 `connect()`、`createStream()`、`publish()` 等是客户端发送的命令消息（Command Message）；`Set Chunk Size`、`Window Acknowledgement Size`、`Set Peer Bandwidth` 等是客户端发送的协议控制消息（Protocol Control Messages）；再到后面的 `Audio Data` 和 `Video Data` 就是音视频数据消息（Data Message）了。


**1）握手过程**

在握手过程中，RTMP 协议本身并没有规定这 6 个消息的具体传输顺序，但 RTMP 协议的实现者需要保证这几点：

- 客户端发送 C0 和 C1 块开始握手。
- 客户端必须（MUST）等接收到 S1 后才能发送 C2。
- 客户端必须（MUST）等接收到 S2 后才能发送其它数据。
- 服务器必须（MUST）等接收到 C0 才能发送 S0 和 S1，也可以（MAY）等接到 C1 一起之后。
- 服务器必须（MUST）等到 C1 才能发送 S2。
- 服务器必须（MUST）等到 C2 才能发送其它数据。


从抓包的数据来看，这里的实现并没有遵守该规定。这里的实现是客户端发送了 `Handshake C0+C1`、`Handshake C2` 两条握手消息后，服务端发回 `Handshake S0+S1+S2` 消息完成握手。






**2）Set Chunk Size 协议控制消息**

首先我们来回顾一下 RTMP 协议中关于 Set Chunk Size 协议控制消息的规定：

1、Set Chunk Size 消息类型（Message Type）为 1。

2、协议控制消息一般使用的类型为 0 的块消息头。

3、协议控制消息一般使用保留的块流 ID 2。

4、RTMP 协议定义的块（Chunk）的数据格式是：

```
+--------------+----------------+--------------------+--------------+
| Basic Header | Message Header | Extended Timestamp |  Chunk Data  |
+--------------+----------------+--------------------+--------------+
|                                                    |
|<------------------- Chunk Header ----------------->|
```

4.1、`Basic Header` 部分，协议控制消息一般使用保留的块流 ID 2，而块流 ID 在 2-63 之间的，会使用 1 字节类型的格式，所以 Set Chunk Size 协议控制消息使用的 Basic Header 的格式会是下面这种类型：

```
   0 1 2 3 4 5 6 7
  +-+-+-+-+-+-+-+-+
  |fmt|   cs id   |
  +-+-+-+-+-+-+-+-+
 Chunk basic header 1
```

其中，对应的 fmt 字段值是 0，表示使用类型为 0 的块消息头；cs id 字段值是 2，即使用协议控制消息保留的块流 ID。


4.2、`Message Header` 部分，协议控制消息会使用类型为 0 的块消息头格式，所以 Set Chunk Size 协议控制消息的块消息头格式如下：

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                   timestamp                   | message length|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|      message length (cont)    |message type id| msg stream id |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          message stream id (cont)             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
            Chunk Message Header - Type 0
```

其中，对应的 timestamp 字段值根据情况设置；message length 字段值是 4，表示 Set Chunk Size 消息的长度是 4 字节；message type id 字段的值是 1，表示是 Set Chunk Size 消息；message stream id 的值则根据情况设置。


4.3、`Extended Timestamp` 部分这里是没有的。


4.4、`Chunk Data` 部分，RTMP 协议中 `Set Chunk Size` 协议控制消息的格式如下：

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|0|                   chunk size (31 bits)                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

其中，chunk size 字段是我们设置的块大小的值。注意这里是小端字节序。


接下来，我们看一下我们抓包到的 `Set Chunk Size` 消息块：

![Wireshark](assets/resource/av-tool/wireshark-9.png)

上面抓包得到这条 16 字节的 `Set Chunk Size` 消息的数据（十六进制）如下：

`02 00 00 00 00 00 04 01 00 00 00 00 00 00 10 00`


其中：

- 块类型（Chunk Type）：0。对应 fmt 字段为 0。
- 块流 ID（Chunk Stream ID）：2。对应 cs id 字段为 2。块类型和块流 ID 一共 1 字节。
- 时间戳（Timestamp）：0。对应 timestamp 字段为 0。
- 负载长度（Message Length）：4。对应 message length 字段为 4，表示消息负载长度是 4 字节。
- 消息类型（Message Type）：1。对应 message type id 字段为 1。
- 消息流 ID（Message Stream ID）：0。对应 message stream id 字段为 0。
- 块长度（Chunk Size）：4096。对应 chunk size 字段为 `0x0001 = 2 ^ 12 = 4096`，表示设置的块大小值为 4096 字节。


关于 Wireshark 的抓包示例我们就介绍到这里，更多的强大的功能，大家可以自己动手去试试，相信不会让你失望。









