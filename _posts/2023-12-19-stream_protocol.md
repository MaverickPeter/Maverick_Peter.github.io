---
layout:     post
title:      "流媒体传输协议"
subtitle:   " \"RTP/RTSP/RTMP\""
date:       2023-12-19 13:00:00
author:     "xxc"
header-img: "img/black_bg.jpg"
catalog: true
tags:
    - 知识分享
    - 日常学习

---
<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>

## 联系与区别
rtp/rtcp/rtsp/rtmp/mms/hls
RTP传输流媒体数据、RTCP对RTP进行控制，同步；RTSP发起/终止流媒体RTP和RTCP互为姐妹关系，RTSP可以使用RTP来传输数据，但并没有绑定关系也可以使用TCP/UDP；RTSP、RTMP、HLS都可以做直播和点播，它们是三种不同的应用层协议
[Reference](https://xie.infoq.cn/article/e35bb0d6b7d6943df6fff509d)

![relation](/img/stream_protocol/bg.png)

## RTSP和RTMP如何选择
RTSP 实时性最好，但是实现复杂，适合视频聊天和视频监控
RTMP 强在浏览器支持好，加载 flash 插件后就能直接播放，非常火，相反在浏览器里播放 rtsp 就很困难了
- IP 摄像机选择RTSP：几乎所有 IP 摄像机都支持 RTSP，这是因为 IP 摄像机早在 RTMP 协议创建之前就已经存在，与 RTSP 和 IP 摄像机结合使用时，IP 摄像机本身充当 RTSP 服务器，这意味着要将摄像机连接到 IP 摄像机服务器并广播视频。
- 物联网设备选择RTSP：RTSP 通常内置在无人机或物联网软件中，从而可以访问视频源，它的好处之一是低延迟，确保视频中没有延迟，这对于无人机来说至关重要。
- 流媒体应用程序选择RTMP：比如各种短视频软件、视频直播软件等都内置了RTMP，RTMP 是为满足现代流媒体需求而设计的。

## RTP/RTCP
RTP协议是建立在UDP协议上的。RTP协议详细说明了在互联网上传递音频和视频的标准数据包格式。RTP协议常用于流媒体系统（配合RTCP协议）、视频会议和视频电话系统（配合H.263或SIP）。


RTP本身并没有提供按时发送机制或其他服务质量（QoS）保证，它依赖于底层服务去实现这一过程。RTP并不保证传送或防止无序传送，也不确定底层网络的可靠性。RTP实行有序传送，RTP中的序列号允许接收方重组发送方的包序列，同时序列号也能用于决定适当的包位置，例如：在视频解码中，就不需要顺序解码。

实时传输控制协议（Real-time Transport Control Protocol,RTCP）是实时传输协议（RTP）的一个姐妹协议。RTCP为RTP媒体流提供信道外控制。RTCP定期在流多媒体会话参加者之间传输控制数据。RTCP的主要功能是为RTP所提供的服务质量提供反馈。RTCP收集相关媒体连接的统计信息，例如：传输字节数，传输分组数，丢失分组数，时延抖动，单向和双向网络延迟等等。网络应用程序可以利用RTCP所提供的信息试图提高服务质量，比如限制信息流量或改用压缩比较小的编解码器。RTCP本身不提供数据加密或身份认证，其伴生协议SRTCP（安全实时传输控制协议）则可用于此类用途。

## 实时流协议RTSP
RTSP协议定义了一对多应用程序如何有效通过IP网络传送多媒体数据。RTSP在体系结构上位于RTP和RTCP之上，它使用TCP或RTP完成数据传输。HTTP与RTSP相比，HTTP传送HTML，而RTSP传送的是多媒体数据。HTTP请求由客户机发出，服务器做出响应；RTSP可以是双向的，即客户机和服务器都可以发出请求。
RTSP与RTP最大的区别在于：RTSP是一种双向实时数据传输协议，它允许客户端向服务器端发送请求，如回放、快进、倒退等操作。当然RTSP可基于RTP来传送数据，还可以选择TCP、UDP、组播UDP等通道来发送数据，具有很好的扩展性。它是一种类似于HTTP协议的网络应用协议。

## 实时消息传输协议RTMP
RTMP（Real Time Messaging Protocol）是Adobe Systems公司为Flash播放器和服务器之间音频、视频和数据传输开发的开放协议。它有三种变种：
（1）工作在TCP之上的明文协议，使用端口1935；

（2）RTMPT封装在HTTP请求之中，可穿越防火墙；

（3）RTMPS类似RTMPT，但使用的是HTTPS连接。

### RTMP视频播放的特点：
（1）RTMP协议是采用实时的流式传输，所以不会缓存文件到客户端，这种特性说明用户想下载RTMP协议下的视频是比较难的；

（2）视频流可以随便拖动，既可以从任意时间点向服务器发送请求进行播放，并不需要视频有关键帧。相比而言，HTTP协议下视频需要有关键帧才可以随意拖动。

（3）RTMP协议支持点播/回放（通俗点将就是支持把flv,f4v,mp4文件放在RTMP服务器，客户端可以直接播放），直播（边录制视频边播放）。

## HLS
HTTP Live Streaming(HLS)是苹果公司实现的基于HTTP的流媒体传输协议，可实现流媒体的直播和点播，主要应用于iOS系统。HLS点播是分段HTTP点播，不同在于它的分段非常小。要实现HLS点播，重点在于对媒体文件分段，目前有不少开源工具可以使用。
相对于常见的流媒体直播协议，HLS直播最大的不同在于，直播客户端获取到的并不是一个完整的数据流，HLS协议在服务器端将直播数据流存储为连续的、很短时长的媒体文件（MPEG-TS格式），而客户端则不断的下载并播放这些小文件，因为服务器总是会将最新的直播数据生成新的小文件，这样客户端只要不停的按顺序播放从服务器获取到的文件，就实现了直播。由此可见，基本上可以认为，HLS是以点播的技术方式实现直播。由于数据通过HTTP协议传输，所以完全不用考虑防火墙或者代理的问题，而且分段文件的时长很短，客户端可以很快的选择和切换码率，以适应不同带宽条件下的播放。不过HLS的这种技术特点，决定了它的延迟一般总是会高于普通的流媒体直播协议。