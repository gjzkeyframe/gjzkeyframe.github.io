---
title: RTMP协议：为什么直播推流都爱用这个传输协议？
description: 介绍 RTMP 协议的数据传输流程和协议设计思想、消息概念具体细节、块概念具体细节等基础知识。
author: Keyframe
date: 2025-02-22 06:08:08 +0800
categories: [音视频基础知识]
tags: [音视频基础知识, 音视频, 传输协议, RTMP]
pin: false
math: true
mermaid: true
---

> 本文转自微信公众号 `关键帧Keyframe`，推荐您关注来获取**音视频**、**AI** 领域的最新技术和产品信息：
>
>![微信公众号](assets/img/keyframe-mp.jpg)
_微信扫码关注我们_
{: .prompt-tip }

（本文基本逻辑：RTMP 协议的数据传输流程和协议设计思想 → 消息概念具体细节 → 块概念具体细节）


## 1、协议概览

RTMP 协议是 Real Time Message Protocol（实时信息传输协议）的缩写，它是由 Adobe 公司提出的一种应用层的协议，用来解决多媒体数据传输流的**多路复用（Multiplexing）**和**分包（Packetizing）**的问题。

RTMP 在两个对等的通信端之间通过可靠的传输协议（例如 TCP）提供双向的消息多路服务，用来传输带有时间信息的并行的视频、音频和数据。通常的协议的实现会给不同类型的消息赋予不同的优先级，当传输能力受到限制时它会影响消息下层流发送的队列顺序。

**由于协议设计对低延时、音视频同步等能力的良好支持，RTMP 是实时直播场景，尤其是在推流上行链路中，最常用的传输协议之一。**


### 1.1、数据传输流程

使用 RTMP 协议来传输音视频数据的流程大致如下：

![image](assets/resource/av-protocol-rtmp-1.webp)

在发送端：

- 把数据封装成消息（Message）；
- 把消息分割成块（Chunk）；
- 将分割后的块（Chunk）通过传输协议（如 TCP）协议发送到网络传输出去。

在接收端：

- 在通过 TCP 协议收到后块（Chunk）数据；
- 先将块（Chunk）重新组装成消息（Message）；
- 通过对消息（Message）进行解封装就可以恢复出可处理数据。




### 1.2、设计思想

**1）分包**

从 RTMP 的数据传输流程可以看出 RTMP 里有两个重要的概念：**消息**和**块**。

消息，服务于数据封装，是 RTMP 协议中的基本数据单元；块，服务于网络传输。

通过这种分层的设计，就**可以将大的消息（Message）数据分包成小的块（Chunk）通过网络来进行传输，这个也是 RTMP 能够实现降低延时的核心原因**。


**2）多路复用**

由于『消息』和『块』的分层设计，使得音频、视频数据在分割成块时对传输通道是透明的，这样**音频、视频数据就能够合到一个传输流中进行同步传输，实现了多路复用。这个流就是『块流（Chunk Stream）』**。

在 RTMP 直播中，实时生成视频 Chunk 和音频 Chunk，依次加入到数据流，通过网络发送到客户端。这样的**复用传输流，也是音视频同步的关键**。


**3）优先级**

**块流（Chunck Stream）**这一层没有优先级的划分，**优先级的设计是放在『消息流（Message Stream）』这层来实现的**。不同的消息具有不同的优先级。当网络传输能力受限时，优先级用来控制消息在网络底层的排队顺序。

控制消息，包括『协议控制消息（Protocol Control Messages）』和『用户控制消息（User Control Messages）』，具有最高的优先级。

通常情况下，优先级是这样的：**控制消息 > 音频消息 > 视频消息**。要使得这样的优先级能够得到有效执行，分块也是非常关键的措施：**将大消息切割成小块，可以避免大的低优先级的消息（如视频消息）堵塞了发送缓冲从而阻塞了小的高优先级的消息（如音频消息或控制消息）**。

**4）块大小协商**

RTMP 发送端，在将消息（Message）切割成块（Chunk）的过程中，是以 Chunk Size（默认值 128 字节）为基准进行切割的。

**Chunk Size 越大，切割时 CPU 的负担越小；但在带宽不够宽裕的环境下，发送比较耗时，会阻塞其他消息的发送。**

**Chunk Size 越小，利于网络发送；但切割时 CPU 的负担越大，而且服务器 CPU 负担也相对较大。不适用于高码率数据流的情况。**

Chunk Size 是可以根据实际情况进行改变的，即通过发送控制命令 `Set Chunk Size` 的方式进行更新。

充分考虑流媒体服务器、带宽、客户端的情况，通过 `Set Chunk Size` 可动态的适应环境，从而达到最优效果。



**5）压缩优化**

`RTMP Chunk Header` 的长度不是固定的，分为：12 字节、8 字节、4 字节、1 字节四种。

最完整的 `RTMP Chunk Header` 长度是 12 字节。

![image](assets/resource/av-protocol-rtmp-2.webp)

一般情况下，`msg stream id` 是不会变的，所以针对视频或音频，除了第一个 Chunk 的 RTMP Chunk Header 是 12 字节的，后续的 Chunk 可省略这个 4 字节的字段，采用 8 字节的 RTMP Chunk Header。

![image](assets/resource/av-protocol-rtmp-3.webp)

如果和前一条 Chunk 相比，当前 Chunk 的消息长度 `message length` 和消息类型 `msg type id`（视频为 9；音频为 8）字段又相同，即可将这两部分也省去，RTMP Chunk Header 采用 4 字节类型。

![image](assets/resource/av-protocol-rtmp-4.webp)

如果和前一条 Chunk 相比，当前 Chunk 的 `msg stream id`、`msg type id`、`message length` 字段都相同，而且都属于同一个消息（由同一个 Message 切割而来），那么这些 Chunk 的时间戳 `timestamp` 字段也会是相同的，故后面的 Chunk 也可以省去 `timestamp` 字段，RTMP Chunk Header 采用 1 字节类型。

![image](assets/resource/av-protocol-rtmp-5.webp)


当 Chunk Size 很大时，此时所有的 Message 都只能相应切割成一个 Chunk，后续的 Chunk 与前一条 Chunk 仅 `msg stream id` 相同。此时基本上除了第一个 Chunk 的 Header 是 12 字节外，其它所有 Chunk 的 Header都是 8 字节。


![image](assets/resource/av-protocol-rtmp-6.webp)



上面从整体上了解了 RTMP 的数据传输流程和协议设计思想后，我们来细看一下『消息（Message）』和『块（Chunk）』。



## 2、消息（Message）

RTMP 的消息被设计为工作在 RTMP 块流之上，所以理论上它可以使用任意的传输层协议来发送消息。客户端和服务器通过网络发送 RTMP 消息进行通信，消息可以包括音频、视频、数据，甚至其它任何信息。


### 2.1、消息格式

RTMP 的消息分为两个部分：

- 消息头
- 有效负载



其中消息头包含以下几个字段：

- 消息类型（Message Type），1 字节，表示消息类型。其中 1-6 的取值是保留给协议控制消息使用的。
- 长度（Length），3 字节，表示有效负载的长度（不包含消息头的长度）。单位是字节，使用大端格式。
- 时间戳（Timestamp），4 字节，表示消息时间戳。使用大端格式。
- 消息流 ID（Message Stream ID），3 字节，用来标识消息流。使用大端格式。


消息头的结构如下所示：

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Message Type  |                Payload length                 |
|   (1 byte)    |                  (3 bytes)                    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                       Timestamp                               |
|                       (4 bytes)                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                Stream ID                      |
|                (3 bytes)                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```


消息的其它部分就是有效负载，这是包含在消息中的实际数据。举例说，它可能是一些音频采样或是被压缩的视频数据。



### 2.2、消息类型

RTMP 的消息类型分为：

- 协议控制消息（Protocol Control Messages）
- 用户控制消息（User Control Messages）
- 其他消息

#### 2.2.1、协议控制消息

**1）设置块大小消息（Set Chunk Size (1)）**

设置块大小消息类型（Message Type）为 1。

该消息用来通知对端一个新的最大块的大小。最大块的大小默认值是 128 字节，但是客户端和服务器可以改变这个值，并且通过这个消息通知对方。比如，当前客户端要发送 131 字节大小的音频块，但是最大块大小是 128 字节，这时客户端可以发送这个消息告诉服务器现在块大小最大为 131 字节，接下来就可以使用一个单独的块来发送这些音频数据了。

如果块大小的最大值是 128 字节，那么块大小区间是 `[1, 128]` 字节。块大
小的最大值每一个方向是独立维护的。

设置块大小消息的有效负载（Payload）格式如下：

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|0|                   chunk size (31 bits)                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- 0：1 比特，这个位必须（MUST）是 0。
- 块大小（chunk size）：31 比特，表示新的块大小的最大值，单位是字节。发送端随后所有的块都使用这个值直到下一个设置块大小最大值的消息。合法的大小是 `[1, 2147483647(2^31-1)]`；然而所有大于 `16777215(2^24-1)` 的大小都是等价的，因为消息头已经决定表示 Payload 的长度的字段是 3 字节，也就是说 Payload 最多只有 16777215 字节。



**2）中止消息（Abort Message (2)）**

中止消息类型（Message Type）为 2。

该消息用来通知对端中止接收块流数据。如果对端已经从块流数据中接收了一个消息的部分数据，还在等着剩下的块流数据来补全这个完整的消息，那么再收到中止消息时，就可以停止等待了，并且可以把已接收的消息的部分数据丢弃掉。该消息的 Payload 是块流 ID。

应用程序在关闭的时候发送这个消息，表明不需要继续处理消息了。

中止消息的有效负载（Payload）格式如下：

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                   chunk stream id (32 bits)                   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- 块流 ID（chunk stream ID）：32 比特，表示可以丢弃的消息的块流 ID。


**3）确认消息（Acknowledgement (3)）**

确认消息类型（Message Type）为 3。

客户端或服务器必须（MUST）在收到字节数等于窗口大小（window size）后向对方向发送确认消息。窗口大小是发送者 在收到确认消息前能发送出去的最大字节数。

确认消息中指定了序列号，这个序列号是指到目前为止接收到的字节数。

确认消息的有效负载（Payload）格式如下：

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                   sequence number (4 bytes)                   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- 序列号（sequence number）：4 字节，表示到目前为止接收到的字节数。



**4）窗口大小消息（Window Acknowledgement Size (5)）**

窗口大小消息类型（Message Type）为 5。

客户端或服务器发送这个消息来告诉对方自己的窗口大小。发送方在发送了等于窗口大小的字节数后期望收到对方的确认消息（Acknowledgement (3)）。接收方必须（MUST）在上一次发送确认消息之后接收到等于窗口大小的字节数后再发送一次确认消息。如果之前没有发送过确认消息，就从会话开始之后开始计算字节数来决定何时发送第一次确认消息。

窗口大小消息的有效负载（Payload）格式如下：

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|             Acknowledgement Window size (4 bytes)             |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- 窗口大小（acknowledgement window size）：4 字节。表示窗口大小。


**5）设置对端带宽消息（Set Peer Bandwidth (6)）**

设置对端带宽消息类型（Message Type）为 6。


客户端或服务器发送这个消息去限制对端的输出带宽。接收方收到这个消息后需要限制自己的输出带宽。除了尚未做出确认的消息外，接收方限制它的输出宽带为该消息中指定的窗口大小。

如果这个消息的接受者发送给对方的上一个窗口大小消息（Window Acknowledgement Size (5)）中指定的窗口大小与它收到的该消息中指定的窗口大小不一样，那么这个消息的接收者在限制自己的输出带宽后，应该（SHOULD）回复一个新的窗口大小消息给对方。

设置对端带宽消息的有效负载（Payload）格式如下：

```
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                 Acknowledgement Window size                   |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |   Limit Type  |
 +-+-+-+-+-+-+-+-+
```

- 窗口大小（acknowledgement window size）：4 字节。表示对方希望接收方设置的窗口大小。
- 限制类型（limit type）：1 字节。表示限制类型：
    - 0 - Hard：消息接收端应该将输出带宽限制为指定窗口大小。
    - 1 - Soft：消息接收端应该将输出带宽限制为指定窗口大小和当前窗口大小中较小值。
    - 2 - Dynamic：如果上一个消息的限制类型为 Hard，则该消息同样为 Hard，否则抛弃该消息。














#### 2.2.2、用户控制消息


用户控制消息（User Control Messages (4)）类型（Message Type）为 4。

用户控制消息应该（SHOULD）使用 `Stream ID` 为 0 的消息流（这是默认的控制流），并且当划分为块发送时，应该使用 `Chunk Stream ID` 为 2 的块流。


当接收方从流中接收到用户控制消息时，它应该立马生效；它们的时间戳可以忽略不用。

客户端或者服务器通过用户控制消息来通知对端用户控制相关的事件，该消息携带事件类型和事件数据。

用户控制消息的有效负载（Payload）格式如下：

```
+------------------------------+-------------------------+
|     Event Type (16 bits)     |       Event Data        |
+------------------------------+-------------------------+
```

- 事件类型（Event Type）：16 比特，表示事件类型。
- 事件数据（Event Data）：事件数据的的长度是可变的。然而，这种情况下消息是通过 RTMP 块流层传送的，块的大小应该（SHOULD）足够大，以满足使用单个块可以空纳这类消息。



客户端与服务器发送个用户控制消息来通知对端用户控制事件。

**用户控制消息事件（User Control Message Events）**类型有下面这些：

| 事件（Event） | 取值（Value） | 描述（Description） |
| - | - | - |
| 流开始（Stream Begin） | 0 | 服务端发送该事件，用来通知客户端一个流已经可以用来通讯了。默认情况下，该事件是在收到客户端连接指令并成功处理后发送的第一个事件。事件的数据使用 4 个字节来表示可用的流的 ID。 |
| 流结束（Stream EOF） | 1 | 服务端发送该事件，用来通知客户端其在流中请求的回放数据已经结束了。如果没有额外的指令，将不会再发送任何数据，而客户端会丢弃之后从该流接收到的消息。事件数据使用 4 个字节来表示回放完成的流的 ID。 |
| 流枯竭（Stream Dry） | 2 | 服务端发送该事件，用来通知客户端流中已经没有更多的数据了。如果服务端在一定时间后没有探测到更多数据，它就可以通知所有订阅该流的客户端，流已经枯竭。事件数据用 4 个字节来表示枯竭的流的 ID。 |
| 设置缓冲区大小（Set Buffer Length） | 3 | 客户端发送该事件，用来告知服务端用来缓存流中数据的缓冲区大小（单位：毫秒）。该事件在服务端开始处理流数据之前发送。事件数据中，前 4 个字节用来表示流 ID，之后的 4 个字节用来表示缓冲区大小（单位：毫秒）。 |
| 流已录制（Stream Is Recorded） | 4 | 服务端发送该事件，用来通知客户端指定流是一个录制流。事件数据用 4 个字节表示录制流的 ID。 |
| ping 请求（Ping Request） | 6 | 服务端发送该事件，用来探测客户端是否处于可达状态。事件数据是一个 4 字节的时间戳，表示服务端分发该事件时的服务器本地时间。客户端收到后用 ping 响应回复服务端。 |
| ping 响应（Ping Response） | 7 | 客户端用该事件回复服务端的 ping 请求，事件数据为收到的 ping 请求中携带的 4 字节的时间戳。 |









#### 2.2.3、其他消息

##### 2.2.3.1、命令消息（Command Message (20, 17)）

命令消息在客户端与服务器之间传递 AMF 编码的命令。消息类型（Message Type）为 20 用于表示 AMF0 编码，消息类型（Message Type）为 17 用于 AMF3 编码。

命令消息可以用来执行对端上的一些操作，比如：connect、createStream、publish、play、pause 等命令。命令消息还可以用于通知发送方请求的命令状态，比如：onstatus、result 等命令。一条命令消息包括：命令名、事务 ID、包含相关参数的命令对象。客户端或服务端还可以通过命令消息来实现远程过程调用（RPC）。



客户端与服务器使用 AMF 编码的命令来交互。命令的接收方在回发响应时，使用同样的事务 ID。响应串是 `_result`、`_error` 或者一个方法名（比如：verifyClient、contactExternalServer）。


命令串 `_result` 和 `_error` 是响应的标志。事务 ID 标明了响应所指向的命令。命令串中的方法名称指明发送者试图在接收端执行的方法。

下面的类对象用来发布不同的命令：

1）**NetConnection，表示服务器与客户端之间的上层的一个连接对象。**NetConnection 管理着客户端应用与服务器之间的双路连接。并且，它提供异常的远程调用。下面的命令可以通过 NetConnection 发送：
    
- connect
- call
- createStream
- close

2）**NetStream，表示发送音频流、视频流或其它相关数据的一个通道。**我们也通过它发送控制数据流的命令如 play、pause 等。NetStream 定义了通道，音频、视频、数据消息通过这个通道在客户端与服务器连接的 NetConnection 上进行流动。一个 NetConnection 对象支持多个 NetStream 数据流。下面的命令可以通过 NetStream 发送：


由客户端发送给服务端的命令：

- play
- play2
- deleteStream
- closeStream
- receiveAudio
- receiveVideo
- publish
- seek
- pause

由服务端响应客户端的命令：

- onStatus




**我们先看一下 NetConnection 各命令的细节：**

1）connect 命令

客户端发送 connect 命令给服务器去请求连接服务器上的一个应用实例。

从客户端到服务器发送的这个命令的结构如下：


| Field Name              | Type   | Description                                                |
| ----------------------- | ------ | ---------------------------------------------------------- |
| Command Name            | String | Name of the command. Set to "connect".                     |
| Transaction ID          | Number | Always set to 1.                                           |
| Command Object          | Object | Command information object which has the name-value pairs. |
| Optional User Arguments | Object | Any optional information                                   |


下面是 connect 命令中命令对象所使用到的键值对：


| Property | Type | Description | Example Value |
| - | - | - | - |
| app             | String  | The Server application name the client is connected to.      | testapp                                 |
| flashver        | String  | Flash Player version. It is the same string as returned by the ApplicationScript getversion() function. | FMSc/1.0                                |
| swfUrl          | String  | URL of the source SWF file making the connection.            | file://C:/FlvPlayer.swf                 |
| tcUrl           | String  | URL of the Server. It has the following format. protocol://servername:port/appName/appInstance | rtmp://localhost:1935/testapp/instance1 |
| fpad            | Boolean | True if proxy is being used.                                 | true or false                           |
| audioCodecs     | Number  | Indicates what audio codecs the client supports.             | SUPPORT_SND_MP3                         |
| videoCodecs     | Number  | Indicates what video codecs are supported.                   | SUPPORT_VID_SORENSON                    |
| videoFunction   | Number  | Indicates what special video functions are supported.        | SUPPORT_VID_CLIENT_SEEK                 |
| pageUrl         | String  | URL of the web page from where the SWF file was loaded.      | http://somehost/sample.html             |
| object Encoding | Number  | AMF encoding method.                                         | AMF3                                    |


audioCodecs 属性有如下标志值：

| Codec Flag          | Usage                                             | Value  |
| ------------------- | ------------------------------------------------- | ------ |
| SUPPORT_SND_NONE    | Raw sound, no compression                         | 0x0001 |
| SUPPORT_SND_ADPCM   | ADPCM compression                                 | 0x0002 |
| SUPPORT_SND_MP3     | mp3 compression                                   | 0x0004 |
| SUPPORT_SND_INTEL   | Not used                                          | 0x0008 |
| SUPPORT_SND_UNUSED  | Not used                                          | 0x0010 |
| SUPPORT_SND_NELLY8  | NellyMoser at 8-kHz compression                   | 0x0020 |
| SUPPORT_SND_NELLY   | NellyMoser compression (5, 11, 22, and 44 kHz)    | 0x0040 |
| SUPPORT_SND_G711A   | G711A sound compression (Flash Media Server only) | 0x0080 |
| SUPPORT_SND_G711U   | G711U sound compression (Flash Media Server only) | 0x0100 |
| SUPPORT_SND_NELLY16 | NellyMouser at 16-kHz compression                 | 0x0200 |
| SUPPORT_SND_AAC     | Advanced audio coding (AAC) codec                 | 0x0400 |
| SUPPORT_SND_SPEEX   | Speex Audio                                       | 0x0800 |
| SUPPORT_SND_ALL     | All RTMP-supported audio codecs                   | 0x0FFF |



videoCodecs 属性有如下标志值：

| Codec Flag                                    | Usage                               | Value  |
| --------------------------------------------- | ----------------------------------- | ------ |
| SUPPORT_VID_UNUSED                            | Obsolete value                      | 0x0001 |
| SUPPORT_VID_JPEG                              | Obsolete value                      | 0x0002 |
| SUPPORT_VID_SORENSON                          | Sorenson Flash video                | 0x0004 |
| SUPPORT_VID_HOMEBREW                          | V1 screen sharing                   | 0x0008 |
| SUPPORT_VID_VP6 (On2)                         | On2 video (Flash 8+)                | 0x0010 |
| SUPPORT_VID_VP6ALPHA (On2 with alpha channel) | On2 video with alpha channel        | 0x0020 |
| SUPPORT_VID_HOMEBREWV (screensharing v2)      | Screen sharing version 2 (Flash 8+) | 0x0040 |
| SUPPORT_VID_H264                              | H264 video                          | 0x0080 |
| SUPPORT_VID_ALL                               | All RTMP-supported video codecs     | 0x00FF |



videoFunction 属性有如下标志值：

| Function Flag           | Usage                                                       | Value |
| ----------------------- | ----------------------------------------------------------- | ----- |
| SUPPORT_VID_CLIENT_SEEK | Indicates that the client can perform frame-accurate seeks. | 1     |



object Encoding 属性有如下标志值：

| Encoding Type | Usage                                                | Value |
| ------------- | ---------------------------------------------------- | ----- |
| AMF0          | AMF0 object encoding supported by Flash 6 and later. | 0     |
| AMF3          | AMF3 encoding from Flash 9 (AS3).                    | 3     |





由服务器发送至客户端的命令结构如下：

| Field Name     | Type   | Description                                                  |
| -------------- | ------ | ------------------------------------------------------------ |
| Command Name   | String | `_result` or `_error`; indicates whether the response is result or error. |
| Transaction ID | Number | Transaction ID is 1 for connect responses.                   |
| Properties     | Object | Name-value pairs that describe the properties(fmsver etc.) of the connection. |
| Information    | Object | Name-value pairs that describe the response from the server. `code`, `level`, `description` are names of few among such information. |




connect 命令执行期间消息流动如下：

```
+--------------+                              +-------------+
|    Client    |              |               |    Server   |
+------+-------+              |               +------+------+
       |               Handshaking done              |
       |                      |                      |
       |                      |                      |
       |----------- Command Message(connect) ------->|
       |                                             |
       |<------- Window Acknowledgement Size --------|
       |                                             |
       |<----------- Set Peer Bandwidth -------------|
       |                                             |
       |-------- Window Acknowledgement Size ------->|
       |                                             |
       |<------ User Control Message(StreamBegin) ---|
       |                                             |
       |<------------ Command Message ---------------|
       |        (_result- connect response)          |
       |                                             |
```


- 1、客户端发送 `connect` 命令至服务器以请求连接至服务器端程序实例。
- 2、在收到连接命令后，服务器端发送协议消息 `Window Acknowledgement Size` 给客户端。同时，服务器端还会连接收到的 `connect` 命令中提到的应用。
- 3、服务器端发送协议消息 `Set Peer Bandwidth` 至客户端。
- 4、客户端成功处理 `Set Peer Bandwidth` 后发送协议消息 `Window Acknowledgement Size` 给服务器端。
- 5、服务器端发送用户控制消息 `StreamBegin` 给客户端。
- 6、服务器端发送命令消息以通知客户端连接状态 `success/fail`。该命令中含有处理 ID（与 1 中收到相同），该消息同时还制定了部分属性，如 Flash Media Server 版本等。除此之外，它还指定了连接响应相关的信息如 level、code、description、objectencoding 等。


2）call 命令

call 用于远程调用接收端上的程序。call 命令传递了 RPC 的名字。

发送端至接收端的命令结构如下：

| Field Name         | Type   | Description                                                  |
| ------------------ | ------ | ------------------------------------------------------------ |
| Procedure Name     | String | Name of the remote procedure that is called.                 |
| Transaction ID     | Number | If a response is expected we give a transaction Id. Else we pass a value of 0. |
| Command Object     | Object | If there exists any command info this is set, else this is set to null type. |
| Optional Arguments | Object | Any optional arguments to be provided.                       |



响应的命令结构如下：

| Field Name     | Type   | Description                                                  |
| -------------- | ------ | ------------------------------------------------------------ |
| Command Name   | String | Name of the command.                                         |
| Transaction ID | Number | ID of the command, to which the response belongs.            |
| Command Object | Object | If there exists any command info this is set, else this is set to null type. |
| Response       | Object | Response from the method that was called.                    |




3）createStream 命令

客户端发送 createStream 命令去创建一个用于消息通信的逻辑通道。音频、视频和 metadata 数据都是通过使用 createStream 命令创建的流通道进行传输的。

NetConnection 是默认的通讯通道，流 ID 为 0。协议消息和部分命令消息会使用默认通讯通道。

从客户端至服务器的 createStream 命令结构如下：

| Field Name     | Type   | Description                                                  |
| -------------- | ------ | ------------------------------------------------------------ |
| Procedure Name | String | Name of the command. Set to "createStream".                  |
| Transaction ID | Number | Transaction ID of the command.                               |
| Command Object | Object | If there exists any command info this is set, else this is set to null type. |

从服务器至客户端的 createStream 命令结构如下：

| Field Name     | Type   | Description                                                  |
| -------------- | ------ | ------------------------------------------------------------ |
| Command Name   | String | `_result` or `_error`; indicates whether the response is result or error. |
| Transaction ID | Number | ID of the command that response belongs to.                  |
| Command Object | Object | If there exists any command info this is set, else this is set to null type. |
| Stream ID      | Number | The return value is either a stream ID or an error information object. |



4）close 命令

无。



**接下来看一下 NetStream 各命令的细节：**

1）play

客户端发送个命令给服务器用以播放一个流。多次使用这个命令也可以创建一个播放列表。

如果你想创建一个在不同的直播流和录制流之间切换的动态播放列表，多次调用 `play`
并且每次 `reset` 都置为 `false`。相反地，如果你想立即播放某个流清除其它等待播放的流，置
`reset` 为 `true`。

从客户端到服务器传输的命令结构如下：

| Field Name     | Type    | Description                                                  |
| -------------- | ------- | ------------------------------------------------------------ |
| Command Name   | String  | Name of the command. Set to "play".                          |
| Transaction ID | Number  | Transaction ID set to 0.                                     |
| Command Object | Null    | Command information does not exist. Set to null type.        |
| Stream Name    | String  | Name of the stream to play. To play video (FLV) files, specify the name of the stream without a file extension (for example, "sample"). To play back MP3 or ID3 tags, you must precede the stream name with mp3: (for example, "mp3:sample"). To play H.264/AAC files, you must precede the stream name with mp4: and specify the file extension. For example, to play the file sample.m4v, specify "mp4:sample.m4v". |
| Start          | Number  | An optional parameter that specifies the start time in seconds. The default value is -2, which means the subscriber first tries to play the live stream specified in the Stream Name field. If a live stream of that name is not found, it plays the recorded stream of the same name. If there is no recorded stream with that name, the subscriber waits for a new live stream with that name and plays it when available. If you pass -1 in the Start field, only the live stream specified in the Stream Name field is played. If you pass 0 or a positive number in the Start field, a recorded stream specified in the Stream Name field is played beginning from the time specified in the Start field. If no recorded stream is found, the next item in the playlist is played. |
| Duration       | Number  | An optional parameter that specifies the duration of playback in seconds. The default value is -1. The -1 value means a live stream is played until it is no longer available or a recorded stream is played until it ends. If you pass 0, it plays the single frame since the time specified in the Start field from the beginning of a recorded stream. It is assumed that the value specified in the Start field is equal to or greater than 0. If you pass a positive number, it plays a live stream for the time period specified in the Duration field. After that it becomes available or plays a recorded stream for the time specified in the Duration field. (If a stream ends before the time specified in the Duration field, playback ends when the stream ends.)  If you pass a negative number other than -1 in the Duration field, it interprets the value as if it were -1. |
| Reset          | Boolean | An optional Boolean value or number that specifies whether to flush any previous playlist. |


命令执行期间消息流动如下：

```
     +-------------+                             +----------+
     | Play Client |             |               |  Server  |
     +------+------+             |               +-----+----+
            |        Handshaking and Application       |
            |               connect done               |
            |                    |                     |
            |                    |                     |
   ---+---- |----- Command Message(createStream) ----->|
Create|     |                                          |
Stream|     |                                          |
   ---+---- |<---------- Command Message --------------|
            |    (_result- createStream response)      |
            |                                          |
   ---+---- |------ Command Message (play) ----------->|
      |     |                                          |
      |     |<------------ SetChunkSize ---------------|
      |     |                                          |
      |     |<---- User Control (StreamIsRecorded) ----|
 Play |     |                                          |
      |     |<------ UserControl (StreamBegin) --------|
      |     |                                          |
      |     |<- Command Message(onStatus-play reset) --|
      |     |                                          |
      |     |<- Command Message(onStatus-play start) --|
      |     |                                          |
      |     |<------------ Audio Message --------------|
      |     |                                          |
      |     |<------------ Video Message --------------|
      |     |                     |                    |
                                  |
        Keep receiving audio and video stream till finishes
```

- 1、客户端在接收到来自服务器 `createStream` 成功回复后发送 `play` 命令。
- 2、收到 `play` 命令后，服务器发送一条协议消息来设置块大小。
- 3、服务器发送用户控制消息来指定事件 `StreamIsRecord` 和流 `ID`。该消息在前 2 字节中携带事件类型，并在最后 4 字节中携带流 ID。
- 4、服务器发送用户控制协议消息 `StreamBegin` 来告知客户端流状态的开始。
- 5、如客户端请求的 `play` 命令成功执行后，服务器发送一条 `onStatus` 命令消息 `NetStream.Play.Start` 和 `NetStream.Play.Reset`。只有客户端发送的 `play` 命令中包含 `reset` 标志时服务器才会回复 `NetStream.Play.Reset`。如未查找到客户端请求播放的流，服务器将在 `onStatus` 消息中返回 `NetStream.Play.StreamNotFound`。
- 6、最后服务端发送需要播放的音视频数据到客户端。

2）play2

与 play 命令不同的是，play2 命令可以在不改变播放内容的时间轴的前提下切换不同码率的流。服务器为所有支持的码率维护多个文件，这样客户端就可以请求 play2 命令。

从客户端发往服务器的命令结构如下：

| Field Name     | Type   | Description                                                  |
| -------------- | ------ | ------------------------------------------------------------ |
| Command Name   | String | Name of the command. Set to "play2".                         |
| Transaction ID | Number | Transaction ID set to 0.                                     |
| Command Object | Null   | Command information does not exist. Set to null type.        |
| Parameters     | Object | An AMF encoded object whose properties are the public properties described for the flash.net.NetStreamPlayOptions ActionScript object. |

`NetStreamPlayOptions` 对象的公共属性在 ActionScript3 Language Reference 中有详细描述。

该命令的具体消息流如下：

```
    +--------------+                          +-------------+
    | Play2 Client |           |              |    Server   |
    +--------+-----+           |              +------+------+
             |      Handshaking and Application      |
             |            connect done               |
             |                 |                     |
             |                 |                     |
    ---+---- |---- Command Message(createStream) --->|
Create |     |                                       | 
Stream |     |                                       |
    ---+---- |<---- Command Message (_result) -------|
             |                                       |
    ---+---- |------ Command Message (play) -------->|
       |     |                                       |
       |     |<---------- SetChunkSize --------------|
       |     |                                       |
       |     |<--- UserControl (StreamIsRecorded) ---| 
  Play |     |                                       |
       |     |<------ UserControl (StreamBegin) -----|
       |     |                                       |
       |     |<- Command Message(onStatus-playstart)-|
       |     |                                       |
       |     |<---------- Audio Message -------------|
       |     |                                       |
       |     |<---------- Video Message -------------|
       |     |                                       |
             |                                       |
    ---+---- |-------- Command Message(play2) ------>|
       |     |                                       |
       |     |<------- Audio Message (new rate) -----|
 Play2 |     |                                       |
       |     |<------- Video Message (new rate) -----|
       |     |                  |                    |
       |     |                  |                    |
       |  Keep receiving audio and video stream till finishes
                                |
```



3）deleteStream

当 NetStream 需要销毁 NetStream 会发送 deleteStream 命令。

从客户端发往服务器的这个命令结构如下：

| Field Name     | Type   | Description                                           |
| -------------- | ------ | ----------------------------------------------------- |
| Command Name   | String | Name of the command. Set to "deleteStream".           |
| Transaction ID | Number | Transaction ID set to 0.                              |
| Command Object | Null   | Command information does not exist. Set to null type. |
| Stream ID      | Number | The ID of the stream that is destroyed on the server. |

服务器端不会进行任何回复。


4）closeStream

无。


5）receiveAudio

NetStream 通过发送 receiveAudio 命令去通知服务器要不要发送音频给客户端。

从客户端发往服务器的这个命令结构如下：

| Field Name     | Type    | Description                                                |
| -------------- | ------- | ---------------------------------------------------------- |
| Command Name   | String  | Name of the command. Set to "receiveAudio".                |
| Transaction ID | Number  | Transaction ID set to 0.                                   |
| Command Object | Null    | Command information does not exist. Set to null type.      |
| Bool Flag      | Boolean | true or false to indicate whether to receive audio or not. |

当 receiveAudio 命令中 Bool 标志为 false 时，服务器不会进行任何响应；如该标志为 true，服务器会响应状态消息 `NetStream.Seek.Notify` 和 `NetStream.Play.Start`。


6）receiveVideo

NetStream 通过发送 receiveAudio 消息去通知服务器要不要发送视频给客户端。

从客户端发往服务器的这个命令结构如下：


| Field Name     | Type    | Description                                                |
| -------------- | ------- | ---------------------------------------------------------- |
| Command Name   | String  | Name of the command. Set to "receiveVideo".                |
| Transaction ID | Number  | Transaction ID set to 0.                                   |
| Command Object | Null    | Command information does not exist. Set to null type.      |
| Bool Flag      | Boolean | true or false to indicate whether to receive video or not. |

如 receiveVideo 命令中 Bool 标志为 false 时，服务器不进行任何响应；如该标志为 true，服务器会响应 `NetStream.Seek.Notify` 和 `NetStream.Play.Start`。

7）publish

客户端向服务器发送 publish 命令用来发布带有名字的流。任何客户端都可以使用这个名字播放这个流和接收发布的音频，视频和数据信息。

从客户端发往服务器的这个命令结构如下：

| Field Name      | Type   | Description                                                  |
| --------------- | ------ | ------------------------------------------------------------ |
| Command Name    | String | Name of the command. Set to "publish".                       |
| Transaction ID  | Number | Transaction ID set to 0.                                     |
| Command Object  | Null   | Command information does not exist. Set to null type.        |
| Publishing Name | String | Name with which the stream is published.                     |
| Publishing Type | String | Type of publishing. Set to "live", "record", or "append". `record`: The stream is published and the data is recorded to a new file. The file is stored on the server in a subdirectory within the directory that contains the server application. If the file already exists, it is overwritten. `append`: The stream is published and the data is appended to a file. If no file is found, it is created. `live`: Live data is published without recording it in a file. |

服务器响应 `onStatus` 命令以标识发布的开始。


8）seek

客户端发送 seek 命令去定位一个媒体文件或者播放列表的偏移量（毫秒为单位）。

从客户端发往服务器的这个命令结构如下：

| Field Name     | Type   | Description                                           |
| -------------- | ------ | ----------------------------------------------------- |
| Command Name   | String | Name of the command. Set to "seek".                   |
| Transaction ID | Number | Transaction ID set to 0.                              |
| Command Object | Null   | Command information does not exist. Set to null type. |
| milliSeconds   | Number | Number of milliseconds to seek into the playlist.     |

当 seek 成功后，服务器响应一条 status 消息 `NetStream.Seek.Notify`。如失败则返回一条 `_error` 消息。

9）pause

客户端发送 pause 命令去告诉服务器暂停或继续播放。

从客户端发往服务器的这个命令结构如下：

| Field Name         | Type    | Description                                            |
| ------------------ | ------- | ------------------------------------------------------ |
| Command Name       | String  | Name of the command. Set to "pause".                   |
| Transaction ID     | Number  | There is no transaction ID for this command. Set to 0. |
| Command Object     | Null    | Command information does not exist. Set to null type.  |
| Pause/Unpause Flag | Boolean | true or false, to indicate pausing or resuming play.   |
| milliSeconds       | Number  | Number of milliseconds to seek into the playlist.      |

操作成功后，如流为停止状态，服务器响应一条状态消息 `NetStream.Pause.Notify`；如流为播放状态，则返回 `NetStream.UnPause.Notify`。如操作失败，则返回 `_error` 消息。


10）onStatus

服务端通过给客户端发送 onStatus 命令来通知 NetStream 状态的更新。

onStatus 命令格式：

| Field Name     | Type   | Description                                                  |
| -------------- | ------ | ------------------------------------------------------------ |
| Command Name   | String | Name of the command. Set to "onStatus".                      |
| Transaction ID | Number | Transaction ID set to 0.                                     |
| Command Object | Null   | There is no command object for onStatus messages.            |
| Info Object    | Object | An AMF object having at least the following three properties: `level`(String): the level for this message, one of "warning", "status", or "error"; `code` (String): the message code, for example "NetStream.Play.Start"; and `description` (String): a human-readable description of the message. The Info object MAY contain other properties as appropriate to the code. |




##### 2.2.3.2、数据消息（Data Message (18, 15)）

数据消息用于客户端向对端发送 Metadata 或者任意用户数据。Metadata 包函了数据的（音频、视频等）详细信息，像创建时间、时长、主题等等。这些消息使用消息类型（Message Type）为 18 表示 AMF0，
用消息类型（Message Type）为 15 来表示 AMF3。


##### 2.2.3.3、共享对象消息（Shared Object Message (19, 16)）

共享对象是一个 Flash 对象（一个键值对的集合），用来同步多个客户端，应用实例等等。消息类型（Message Type）为 19 表示使用 AMF0，消息类型（Message Type）为 16 保留用作 AMF3 编码共享事件。每一个消息都可能包含多个事件。

共享对象消息格式如下：

```
+------+------+-------+-----+-----+------+-----+ +-----+------+-----+
|Header|Shared|Current|Flags|Event|Event |Event|.|Event|Event |Event|
|      |Object|Version|     |Type |data  |data |.|Type |data  |data |
|      |Name  |       |     |     |length|     |.|     |length|     |
+------+------+-------+-----+-----+------+-----+ +-----+------+-----+
       |                                                            |
       |<- - - - - - - - - - - - - - - - - - - - - - - - - - - - - >|
       |            AMF Shared Object Message body                  |
```

支持以下事件类型：

| 事件（Event） | 取值（Value） | 描述（Description） |
| - | - | - |
| 创建（Use） | 1 | 客户端向服务端发送，请求创建指定名称的共享对象。 |
| 释放（Release） | 2 | 客户端通知服务端，共享对象已在本地删除。 |
| 请求更新（Request Change） | 3 | 客户端请求修改共享对象的属性值。 |
| 更新（Change） | 4 | 服务端向除请求发送方外的其他客户端发送，通知其有属性的值发生了变化。 |
| 成功（Success） | 5 | “请求更新”事件被接受后，服务端向发送请求的客户端回复此事件。 |
| 发送消息（Send Message） | 6 | 客户端向服务端发送此事件，来广播一个消息。服务端收到此事件后向所有客户端广播一条消息，包括请求方客户端。 |
| 状态（Status） | 7 | 服务端发送此事件来通知客户端错误信息。 |
| 清除（Clear） | 8 | 服务端向客户端发送此事件，通知客户端清除一个共享对象。服务端在回复客户端的“创建”事件时也会发送此事件。 |
| 移除（Remove） | 9 | 服务端发送此事件，使客户端删除一个插槽。 |
| 请求移除（Request Remove） | 10 | 客户端删除一个插槽时发送此事件。 |
| 创建成功（Use Success） | 11 | 当连接成功时服务端向客户端发送此事件。 |


##### 2.2.3.4、音频消息（Audio Message (8)）

客户端或服务器使用音频消息来发送音频数据，消息类型（Message Type）为 8。



##### 2.2.3.5、视频消息（Video Message (9)）

客户端或服务器使用音频消息来发送视频数据，消息类型（Message Type）为 9。



##### 2.2.3.6、聚合消息（Aggregate Message (22)）

聚合消息是一个单独的消息，但它可以包含一系列的子消息。消息类型（Message Type）为 22。

聚合消息格式和消息体格式：

```
             +---------+-------------------------+
             | Header  | Aggregate Message body  |
             +---------+-------------------------+
                  The Aggregate Message format
+--------+-------+---------+--------+-------+---------+ - - - -
|Header 0|Message|Back     |Header 1|Message|Back     |
|        |Data 0 |Pointer 0|        |Data 1 |Pointer 1|
+--------+-------+---------+--------+-------+---------+ - - - -
                The Aggregate Message body format
```

复合消息的消息流 ID 覆盖了复合内部的子消息的消息流 ID。

复合消息与第一个子消息的时间戳之间的差值用来重新偏移的所有消息的时间刻度。该偏移量加到每一个子消息的时间戳上才是正常的流时间。第一个子消息时间戳应该（SHOULD）与聚合消息的时间戳相同，这样这个偏移量应当（SHOULD）是 0。

后向指针（Back Pointer ）字段包含了前一个消息的大小（包括它的头部）。这个字段的目的是用来匹配 FLV 文件和用来向前 seek。


使用聚合消息有几个性能优势：

- 1）在块流中，一个块最多只能发送一个消息。如果在增加发送块大小的前提下，同时使用聚合消息，就可以减少发送块的数量。
- 2）子消息在内存中很可能是连续存储的，所以这时候使用聚合消息发送这些数据在系统调用上会更高效。




### 2.3、消息交互实例


#### 2.3.1、握手

RTMP 的连接开始于握手。握手内容不同于协议的其它部分，它包含三个固定大小的块，而不是带头信息的变长块。

客户端（发起连接的端点）和服务器各自发送相同的三个块。为了演示，这三个块客户端发送的被记做 C0，C1，C2；服务发送的被记做 S0，S1，S2。

**1）握手顺序**

RTMP 协议本身并没有规定这 6 个消息的具体传输顺序，但 RTMP 协议的实现者需要保证这几点：

- 客户端发送 C0 和 C1 块开始握手。
- 客户端必须（MUST）等接收到 S1 后才能发送 C2。
- 客户端必须（MUST）等接收到 S2 后才能发送其它数据。
- 服务器必须（MUST）等接收到 C0 才能发送 S0 和 S1，也可以（MAY）等接到 C1 一起之后。
- 服务器必须（MUST）等到 C1 才能发送 S2。
- 服务器必须（MUST）等到 C2 才能发送其它数据。

**2）C0 和 S0 的格式**

C0 和 S0 包只有八个位，可以看成一个 8 位的整数。

```
 0 1 2 3 4 5 6 7
+-+-+-+-+-+-+-+-+
|    version    |
+-+-+-+-+-+-+-+-+
 C0 and S0 bits
```

下面 C0/S0 包的各个字段：

- 版本（8 bits）：在 C0 中，该字段标识了客户端请求的 RTMP 版本。在 S0 中，这个字段是服务器的 RTMP 版本。这个版本被定义成 3。0-2 用于早期的专利产品已经废弃了；4-31 被保留给未来的实现；32-255 是被禁止的（为了区分 RTMP 和基于文本的协议，基于文本的协议经常是以可打印字符开头）。如果不能识别客户端请求的版本，服务器应该（SHOULD）回复 3。客户端降低版本到 3 或者放弃连接。


**3）C1 和 S1 的格式**

C1 和 S1 包有 1536 个长节长，参考一下下面的字段：


```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        time (4 bytes)                         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        zero (4 bytes)                         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        random bytes                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        random bytes                           |
|                           (cont)                              |
|                            ....                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                        C1 and S1 bits
```

- 时间（4 字节）：这个字段包含了一个时间戳，它可能（SHOULD）作为所有本端的后续块的起始时间点。它可以是 0，也可以是任意值。为了同步多个块流，终端可能希望发送其它块流的当前时间戳的值。
- 零（4 字节）：这个字段必需是全 0。
- 随机数（1528 字节）：这个字段包含了任意值。因为每一个终端都区分该数据是回复它自己发起的握手的还是对端发起的握手，这些发送的数据应该（SHOULD）足够的随机。也不必是加密的随机数，或者是动态数据。


**4）C2 和 S2 的格式**

C2 和 S2 包有 1536 字节长，分别是 S1 和 C1 的回复，包含下面的域：


```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                       time (4 bytes)                          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                       time2 (4 bytes)                         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        random echo                            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        random echo                            |
|                          (cont)                               |
|                           ....                                |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
                      C2 and S2 bits
```

- 时间（4 字节）：这个字段必须（MUST）包含了一个时间戳，它是由对端发送过来。对于 C2 来说是 S1，对于 S2 来说是 C1。
- 时间 2（4 字节）：这个字段必须（MUST）包含前一个对端发送过来的并被读取的包（S1 或者 C1）。
- 随机数（1528 字节）：这个字段必须（MUST）包含对端发送过来的随机数对 S1 来说是 C2，对于 S2 来说是 C1。两端都可以时间字段和时间 2 字段结合当前的时间来快速宽带和（或）连接的延迟，但这个方法不太可能很有用处。

**5）握手流程示意图**

```
+-------------+                            +-------------+
|   Client    |      TCP/IP Network        |     Server  |
+-------------+             |              +-------------+
       |                    |                     |
Uninitialized               |                Uninitialized
       |        C0          |                     |
       |------------------->|           C0        |
       |                    |-------------------->|
       |        C1          |                     |
       |------------------->|           S0        |
       |                    |<--------------------|
       |                    |           S1        |
  Version sent              |<--------------------|
       |        S0          |                     |
       |<-------------------|                     |
       |        S1          |                     |
       |<-------------------|               Version sent
       |                    |           C1        |
       |                    |-------------------->|
       |        C2          |                     |
       |------------------->|           S2        |
       |                    |<--------------------|
    Ack sent                |                   Ack Sent
       |        S2          |                     |
       |<-------------------|                     |
       |                    |           C2        |
       |                    |-------------------->|
Handshake Done              |               Handshake Done
      |                     |                     |
          Pictorial Representation of Handshake
```


下面是握手图示中提到的各个阶段具体内容：

- 未初始化（Uninitialized）：协议的版本发送出去的这个阶段。客户端与服务器都是未初始化态。客户端在 C0 中发送协议版本。如果服务器支持这个版本，它将回应 S0 和 S1。如果不支持，服务器将会采取适当措施的回应。在 RTMP 中，这个措施是终止连接。
- 版本已发送（Version Sent）：客户端和服务器在未初始化态后是版本已发送态。客户端等待 S1，服务器在等待 C1。在接收到响应包后，客户端发送 C2，服务器发送 S2。状态就变成了确认已发送。
- 确认已发送（Ack Send）：客户端和服务器分别在等 S2 和 C2。
- 握手完成（Handshake Done）：客户端与服务器可以交换消息了。



#### 2.3.2、发布录制视频


这个例子演示了发布端如何能发布一个流然后将视频推送到服务器上。其它客户端可以订阅这个发布的流并且还可以播放这个视频。

```
     +--------------------+                     +-----------+
     |  Publisher Client  |        |            |   Server  |
     +----------+---------+        |            +-----+-----+
                |           Handshaking Done          |
                |                  |                  |
                |                  |                  |
       ---+---- |----- Command Message(connect) ----->|
          |     |                                     |
          |     |<----- Window Acknowledge Size ------|
  Connect |     |                                     |
          |     |<------ Set Peer BandWidth ----------|
          |     |                                     |
          |     |------ Window Acknowledge Size ----->|
          |     |                                     |
          |     |<----- User Control(StreamBegin) ----|
          |     |                                     |
       ---+---- |<-------- Command Message -----------|
                |    (_result- connect response)      |
                |                                     |
       ---+---- |--- Command Message(createStream) -->| 
   Create |     |                                     |
   Stream |     |                                     |
       ---+---- |<------- Command Message ------------| 
                |  (_result- createStream response)   |
                |                                     |
       ---+---- |---- Command Message(publish) ------>|
          |     |                                     |
          |     |<----- User Control(StreamBegin) ----|
          |     |                                     |
          |     |------ Data Message (Metadata) ----->|
          |     |                                     |
Publishing|     |------------ Audio Data ------------>| 
  Content |     |                                     |
          |     |------------ SetChunkSize ---------->|
          |     |                                     |
          |     |<--------- Command Message ----------|
          |     |      (_result- publish result)      |
          |     |                                     |
          |     |------------- Video Data ----------->|
          |     |                  |                  |
          |     |                  |                  |
                |    Until the stream is complete     |
                |                  |                  |
```




#### 2.3.3、广播共享对象

这个例子演示了共享对象的创建和改变的消息的交互。同时也演示了共享对象广播消息的过程。

```
             +----------+                       +----------+
             |  Client  |          |            |  Server  |
             +-----+----+          |            +-----+----+
                   |   Handshaking and Application    |
                   |          connect done            |
                   |               |                  |
                   |               |                  |
          ---+---- |---- Shared Object Event(Use) --->|
Create and   |     |                                  |
connect      |     |                                  |
Shared Object|     |                                  |
          ---+---- |<---- Shared Object Event---------|
                   |       (UseSuccess,Clear)         |
                   |                                  |
          ---+---- |------ Shared Object Event ------>|
Shared object|     |         (RequestChange)          | 
Set Property |     |                                  |        
          ---+---- |<------ Shared Object Event ------|
                   |            (Success)             |
                   |                                  |
          ---+---- |------- Shared Object Event ----->| 
Shared object|     |           (SendMessage)          | 
Message      |     |                                  | 
Broadcast    |     |                                  |
          ---+---- |<------- Shared Object Event -----|
                   |           (SendMessage)          |
                   |                |                 |
                                    |                 |
```






#### 2.3.4、发布来自录制流的 Metadata

这个例子描述了发布 Metadata 的消息交互。

```
      +------------------+                       +---------+
      | Publisher Client |         |             |   FMS   |
      +---------+--------+         |             +----+----+
                |     Handshaking and Application     |
                |            connect done             |
                |                  |                  |
                |                  |                  |
        ---+--- |-- Command Messsage(createStream) -->|
    Create |    |                                     |
    Stream |    |                                     |
        ---+--- |<-------- Command Message -----------|
                |    (_result - command response)     |
                |                                     |
        ---+--- |---- Command Message(publish) ------>|
Publishing |    |                                     |
  metadata |    |<----- UserControl(StreamBegin) -----|
 from file |    |                                     |
           |    |------ Data Message (Metadata) ----->|
                |                                     |
```






## 3、块（Chunk）

**在 RTMP 中，块（Chunk）是指消息的一个分片。消息在通过网络发送前被划分成更小的部分并且交叉的发送。**

**块流（Chunk Stream）是指通信的逻辑通道，它为高层的多媒体协议提供了混流和组包服务。**块流允许块在特定的方向流动，可以是从客户端到服务器或者相反方向。块流保证横跨多个流的所有消息端对端传输的时序。每一个块都有一个块流 ID（Chunk Stream ID），它用来标识块传送使用的是哪一个块流。

块流的第一个消息都包含了时间戳和载荷的类型识别，所以块流除了工作在 RTMP 协议上，也可以使用其他协议来发送消息数据。RTMP 块流和 RTMP 协议协同工作很适合于各种和样的音视频程序，从一对一和一对多的直播到视频点播服务再到互动会议程序。

当配合可靠传输协议如 TCP 时，RTMP 块流提供了有保证的时间戳序列的对端跨流消息传输。RTMP 块流并未提供任何优先级或相关形式的控制，类似功能可以由高层协议来提供，RTMP 块流可以进行配合。RTMP 块流集成了带内（in-band）协议控制消息，而且为高层协议提供了嵌入自定义控制消息的机制。例如，一个实时视频服务器可能会参考每条消息发送和响应的时间，来决定是否要丢弃部分视频消息以满足较慢客户端能够流畅地接收音频数据。



将消息分割成块用来支持混流的消息格式取决于高层协议。如果我们创建一个块，消息格式应该（SHOULD）包含下面的字段：

- 时间戳：消息时间戳，该字段为 4 个字节。
- 长度：消息负载长度，如果消息头（header）不能被省略，其长度也应计入其中。该字段占用块头中的 3 个字节。
- 类型 ID：一些类型 ID 预留用于协议控制消息，这些传递信息的消息会同时被 RTMP 块流协议和高层协议处理。所有其他的类型 ID 则由高层协议使用，RTMP 块流会直接忽略这些值。实际上，RTMP 块流并不需要类型 ID ，所有消息都可以是同一类型，有时候应用可以通过该字段来区隔同步轨。该字段占用块头中的 1 个字节
- 消息流 ID ：消息流 ID 可以是任意值，不同的消息流复用到同一块流，然后再根据各自消息流 ID 进行解复用。除此之外，别无用处。该字段占用块头中的 4 字节，使用小端格式。


### 3.1、块格式

每一个块由块头和数据组成；块头又由块基本头、消息头和扩展时间戳三个部分组成。


```
 +--------------+----------------+--------------------+--------------+
 | Basic Header | Message Header | Extended Timestamp |  Chunk Data  |
 +--------------+----------------+--------------------+--------------+
 |                                                    |
 |<------------------- Chunk Header ----------------->|
```

- 块头（Chunk Header）：
    - 块基本头（Basic Header，1 到 3 字节）：这个字段将对块流 ID 和块类型进行了编码。块类型标明了消息头的编码格式。字段的长度取决于块流 ID，因为块流是一个变长的字段。
    - 块消息头（Message Header，0、3、7 和 11 字节）：这个字段对发送的消息（部分或和全部）的信息进行了编码。这个字段的长度取决于块头的块类型。
    - 块扩展时间戳（Extended Timestamp，0 和 4 字节）：这个字段是否出现取决于块消息头中的时间戳（timestamp）或时间戳增量（timestamp delta）。
- 块数据（Chunk Data，可变大小）：这个块的有效负载，最大为配置的最大的块大小。


#### 3.1.1、块基本头（Chunk Basic Header）

块头将块类型（下图中用 fmt 表示）和块流 ID 进行了编码。块类型决定了消息头的编码格式。块基本头字段可能是 1、2 或 3 字节，具体是多少字节取决于块流 ID。为了高效，协议的实现应该（SHOULD）使用能够容纳 ID 的最小表示来节省字节。

块头协议支持块流 ID 是 3-65599 之间的 65597 个自定义值（从 3 开始，是因为 0、1 和 2 是块流 ID 的保留值）。



因此块基本头有 3 种格式：

1）块基本头格式 1：

第 2-7 位的值在 3-63 范围的块基本头表示了完整的块流 ID。

块流 ID 在 2-63 之间的可以编码成块基本头 1 字节的形式。其中，块流 ID 为 2 是保留用做协议控制消息和命令。

```
   0 1 2 3 4 5 6 7
  +-+-+-+-+-+-+-+-+
  |fmt|   cs id   |
  +-+-+-+-+-+-+-+-+
 Chunk basic header 1
```



2）块基本头格式 2：

第 2-7 位的值为 0，表明块基本头是 2 字节的形式。

块流 ID 在 64-319 之间的可以编码成块基本头的 2 字节的形式。块流 ID 的计算方法是:`第二个字节 + 64`。


```
  0                      1
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |fmt|      0    |  cs id - 64   |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      Chunk basic header 2
```




3）块基本头格式 3：

第 2-7 位的值为 1，表明块基本头是 3 字节的形式。

块流 ID 在 64-65566 之间的可以编码成块基本头为 3 字节的形式。块流 ID 的计算方法是：`(第三个字节) * 256 + 第二个字节 + 64`。

```
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |fmt|         1 |          cs id - 64           |
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
             Chunk basic header 3
```



综上所述，我们回顾一下上面 3 种格式的几个字段的含义：


- `fmt`（2 位）：这个字段标识了『块消息头』的类型，这个『块消息头』的每一种块类型将在下一节介绍。
- `第 2-7 位`（6 位）：
    - 这个字段值为 0 时，表示块基本头是 2 字节的版本。
    - 这个字段值为 1 时，表示块基本头是 3 字节的版本。
    - 这个字段值为 2-63 范围时，其值表示块流 ID。其中，值为 2 的块流 ID 是保留用做协议控制消息和命令。
- `cs id - 64`（8 或 16 位）：这个字段表示块流 ID 减去 64。比如说，块流 ID 是 365 时，会在`第 2-7 位`用 1 表示，在 `cs id - 64` 这里用 16 位表示的 301。


实际使用中，块基本头一般用 1 字节格式。


#### 3.1.2、块消息头（Chunk Message Header）

有四种不同的块消息头，其类型由块基本头中的 `fmt` 字段表示。RTMP 的实现应该（SHOULD）尽可能为每一种块消息头使用最紧凑的表示。

下面对在不同的块消息头格式中，用到的一些字段做一下解释：

- 时间戳（timestamp，3 字节）：对于类型为 0 块，这里发送的是绝对时间戳。如是时间戳大于等于 16777215（`2 ^ 24 - 1`），这个字段必须（MUST）是 16777215，表明扩展时间戳（Extended Timestamp）字段使用全 32 位对时间进行了编码。否则，这个字段应该（SHOULD）表示完整的时间戳。
- 时间戳增量（timestamp delta，3 字节）：对于类型为 1 和类型为 2 的块，这里发送的是先前块与当前块之间的差值。如果增量大于等于 16777215（`2 ^ 24 - 1`），这个字段必须（MUST）是 167777215，表明扩展时间戳（Extended Timestamp）字段使用全 32 位的时间戳增量进行了编码。否则，这个字段应该（SHOULD）表示实际的时间戳增量。
- 消息长度（message length，3 字节）：对于类型为 0 或者类型为 1 的块，这里发送的是消息的长度。注意一般情况下，这与块有效负载的长度是不一样的。所有块的有效负载长度除了最后一块外都是块的额定最大长度，最后一块长度是消息的剩余长度（对于长度很短的消息，这可能是消息的全部长度）。
- 消息类型 ID（message type id，1 字节）：对于类型为 0 和类型为 1 的块，这里表示的是消息类型。
- 消息流 ID（message stream id，4 字节）：对于类型为 0 的块，存储的是消息流 ID。消息流 ID 是以小端（little-endian）格式存储的。通常情况下，所有使用相同块流的消息应该来自于相同的消息流。将不同消息流混合到同一个的块流里是有可能的，但这样就不能有效地进行头部压缩来节省带宽。如果现有的一个消息流关闭了，随后又打开了一个新的消息流，这时我们可以通过发送块类型为 0 的块来复用之前的块流。


##### 3.1.2.1、类型 0

类型为 0 块消息头有 11 字节长。这种类型必须（MUST）在块流开始的时候使用，不管是不是流的时间戳回溯（比如回溯定位）。


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

cont 表示 continue 的意思。




##### 3.1.2.2、类型 1

类型为 1 的块消息头有 7 个字节长。没有包含消息流 ID（message stream id），这个块使用了前一个块的消息流 ID。变长消息的流（比如很多视频格式）应该（SHOULD）在流的第一个块之后每个新消息的首个块之后使用这个格式。

```
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |                   timestamp delta             | message length|
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
 |      message length (cont)    |message type id|  
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ 
                Chunk Message Header - Type 1 
```




##### 3.1.2.3、类型 2

类型为 2 的块消息头有 3 个字节长。不包含消息流 ID （message stream id）和消息长度（message length），这个块与上一个块有相同的消息流 ID 和消息长度。消息有固定长度的流（比如说，一些音频和数据格式）应该（SHOULD）在流的第一个块之后每个消息的首个块之后使用这个格式。


```
  0                   1                   2     
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ 
 |                 timestamp delta               |  
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+ 
          Chunk Message Header - Type 2 
```



##### 3.1.2.4、类型 3

类型为 3 的块没有块消息头。不包含消息流 ID（message stream id）、消息长度（message length）和时间戳（timestamp）字段。

当单个消息被分解成多个块，除了第一个块之外的后续块应当（SHOULD）使用类型 3 的块。这些块有相同消息流 ID、消息长度和时间戳。

由相同大小、消息流 ID 和时间戳增量的消息组成的流，在类型 2 的块之后所有块都应该使用该类型格式。

如果第一个消息和第二消息之间的时间戳增量与第一个消息的时间戳相同，则 0 类型的块之后可以马上发送 3 类型的块，而不必使用 2 类型的块来注册时间戳增量。如果类型 3 的块跟在类型 0 的块后面，那么 3 类型块的时间戳增量与 0 类型块的时间戳相同。





#### 3.1.3、扩展时间戳（Chunk Extended Timestamp）

扩展时间戳字段用来对大于 16777215 的时间戳或时间戳增量进行编码。

对于类型为 0，1，2 的块的时间戳或者时间戳增量超过 3 字节时，这个字段则用 4 字节来表示的时间戳或时间戳增量。

扩展时间戳占 4 个字节，能表示的最大数值就是 4294967295。当扩展时间戳启用时，`timestamp` 字段或者 `timestamp delta` 要全置为 1，表示应该去扩展时间戳字段来提取真正的时间戳或者时间戳增量。注意扩展时间戳存储的是完整值，而不是减去时间戳或者时间戳增量的值。


### 3.2、块数据示例


#### 3.2.1、示例 1

这是一个简单的音频流消息，这个示例示范了信息冗余。

```
+---------+-----------------+-----------------+-----------------+
|         |Message Stream ID| Message TYpe ID | Time  | Length  |
+---------+-----------------+-----------------+-------+---------+
| Msg # 1 |    12345        |         8       | 1000  |   32    |
+---------+-----------------+-----------------+-------+---------+
| Msg # 2 |    12345        |         8       | 1020  |   32    |
+---------+-----------------+-----------------+-------+---------+
| Msg # 3 |    12345        |         8       | 1040  |   32    |
+---------+-----------------+-----------------+-------+---------+
| Msg # 4 |    12345        |         8       | 1060  |   32    |
+---------+-----------------+-----------------+-------+---------+
           Sample audio messages to be made into chunks
```

下图展示该消息流分割成的块。从 `Chunk#3` 的块开始进行数据传输优化，从这里开始的块，块头只占用 1 字节（只有块基本头，没有块消息头）。

```
+--------+---------+-----+------------+------- ---+------------+
|        | Chunk   |Chunk|Header Data |No. of     |Total No. of|
|        |Stream ID|Type |            |Bytes After|Bytes in the|
|        |         |     |            |Header     |Chunk       |
+--------+---------+-----+------------+-----------+------------+
|Chunk#1 |    3    |  0  | delta: 1000|   32      |    44      |
|        |         |     | length: 32,|           |            |
|        |         |     | type: 8,   |           |            |
|        |         |     | stream ID: |           |            |
|        |         |     | 12345 (11  |           |            |
|        |         |     | bytes)     |           |            |
+--------+---------+-----+------------+-----------+------------+
|Chunk#2 |    3    |  2  | 20 (3      |   32      |    36      |
|        |         |     | bytes)     |           |            |
+--------+---------+-----+----+-------+-----------+------------+
|Chunk#3 |    3    |  3  | none (0    |   32      |    33      |
|        |         |     | bytes)     |           |            |
+--------+---------+-----+------------+-----------+------------+
|Chunk#4 |    3    |  3  | none (0    |   32      |    33      |
|        |         |     | bytes)     |           |            |
+--------+---------+-----+------------+-----------+------------+
         Format of each of the chunks of audio messages
```


#### 3.2.2、示例 2

该示例展示了一个超过 128 字节长度的消息，在发送的过程中被分割成了多个块。

```
+-----------+-------------------+-----------------+-----------------+
|           | Message Stream ID | Message Type ID | Time  | Length  |
+-----------+-------------------+-----------------+-----------------+
| Msg # 1   |       12346       |    9 (video)    | 1000  |   307   |
+-----------+-------------------+-----------------+-----------------+
                Sample Message to be broken to chunks
```

下图是被分割成的块：

```
+-------+------+-----+-------------+-----------+------------+
|       |Chunk |Chunk|Header       |No. of     |Total No. of|
|       |Stream| Type|Data         |Bytes After| Bytes in   |
|       | ID   |     |             |Header     | the Chunk  |
+-------+------+-----+-------------+-----------+------------+
|Chunk#1|  4   |  0  | delta: 1000 |  128      |   140      |
|       |      |     | length: 307 |           |            |
|       |      |     | type: 9,    |           |            |
|       |      |     | stream ID:  |           |            |
|       |      |     | 12346 (11   |           |            |
|       |      |     | bytes)      |           |            |
+-------+------+-----+-------------+-----------+------------+
|Chunk#2|  4   |  3  | none (0     |  128      |   129      |
|       |      |     | bytes)      |           |            |
+-------+------+-----+-------------+-----------+------------+
|Chunk#3|  4   |  3  | none (0     |  51       |   52       |
|       |      |     | bytes)      |           |            |
+-------+------+-----+-------------+-----------+------------+
                 Format of each of the chunks
```

第一个块 `Chunk#1` 的头信息指明了消息总大小为 307 字节。




从两个示例观察到，类型为 3 的块有两种不同的使用途径：

- 1）是用来指示新消息的开始，这个消息的头继承于已有的状态数据（示例 1）。
- 2）用于同一个消息的连续块（示例 2）。




## 4、问题集锦

1、RTMP 发送的 Chunk Size 大小应该如何设置？

RTMP 发送端，在将消息（Message）切割成块（Chunk）的过程中，是以 Chunk Size(默认值 128 字节) 为基准进行切割的。

Chunk Size 越大，切割时 CPU 的负担越小；但在带宽不够宽裕的环境下，发送比较耗时，会阻塞其他消息的发送。

Chunk Size 越小，利于网络发送；但切割时 CPU 的负担越大，而且服务器 CPU 负担也相对较大。不适用于高码率数据流的情况。

Chunk Size 是可以实际情况进行改变的，即通过发送控制命令 `Set Chunk Size` 的方式进行更新。

充分考虑流媒体服务器、带宽、客户端的情况，通过 `Set Chunk Size` 可动态的适应环境，从而达到最优效果。

可以参考设置为 1024。

2、推流发送 SPS、PPS 的顺序是什么？

推流发送顺序：VPS、SPS、PPS。顺序错了可能会出问题。



## 小结

RTMP 是目前在直播场景广泛使用的协议，通过上文的介绍，我们了解了 RTMP 协议的整体设计和诸多细节。


## 本文参考


- [RTMP Specification 1.0](https://www.adobe.com/content/dam/acom/en/devnet/rtmp/pdf/rtmp_specification_1.0.pdf)


