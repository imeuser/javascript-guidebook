---
nav:
  title: 计算机网络
  order: 8
group:
  title: 计算机网络体系
  order: 2
title: 传输层协议
order: 2
---

# 传输层协议

传输层（Transport Layer）的主要任务就是负责向两台主机进程之间的通信提供通用的 <strong style="color:red">数据传输服务</strong>。应用进程利用该服务传送应用层报文。

网络协议族中有两个具有代表性的传输层协议，分别是 TCP 和 UDP。

- 传输控制协议 TCP：提供面向连接的，可靠的数据传输服务
- 用户数据协议 UDP：提供无连接的，尽最大努力的数据传输服务（不保证数据传输的可靠性）

## TCP

**传输控制协议**（Transmission Control Protocol，简称 TCP）是一种 **面向连接**（连接导向）的、可靠的、 基于 IP 协议的传输层协议。

- **面向连接**：每条 TCP 连接只能有两个端点（亦即点对点，不可广播、多播），每一条 TCP 连接只能是一对一
- **可靠的传输服务**：通过 TCP 连接传送的数据，无差错、不丢失、不重复、并且按序到达，丢包时通过重传机制进而增加时延实现可靠性
- **全双工通信**：TCP 允许通信双方的应用进程在任何时候都能发送数据。TCP 连接的两端都设有发送缓存和接收缓存，用来临时存放双方通信的数据
- **字节流**：面向字节流，TCP 中的 **流**（Stream）指的是流入进程或从进程流出的字节序列
- **流量缓冲**：解决速度不匹配问题

```jsx | inline
import React from 'react';
import img from '../../assets/transport-layer-protocol/tcp-status-machine.jpeg';

export default () => <img alt="TCP 状态机" src={img} width={400} />;
```

### 数据包结构

参考：[数据包结构](https://zh.wikipedia.org/wiki/%E4%BC%A0%E8%BE%93%E6%8E%A7%E5%88%B6%E5%8D%8F%E8%AE%AE)

TCP 首部标志比特有 6 个：URG、ACK、PSH、RST、SYN、FIN

| 控制位 | 名称             | 说明                             |
| :----- | :--------------- | :------------------------------- |
| URG    | Urgent Flag      | 紧急指针                         |
| ACK    | Acknowledge Flag | 确认序号有效                     |
| PSH    | Push Flag        | 尽可能快地将数据送往接收进程     |
| RST    | Reset Flag       | 可能需要重现创建建 TCP 连接      |
| SYN    | Synchronize      | 同步序号来发起一个连接           |
| FIN    | Finish           | 发送方完成发送任务，要求释放连接 |
| Seq    | Sequance number  | 序列号                           |

### 三次握手

TCP 提供 <strong style="color:red">面向连接</strong> 的通信传输。面向有连接是指在数据通信开始之前先做好两端之间的准备工作，也就是说无论哪一方向另一方发送数据之前，都必须先在双方之间建立一条连接。

三次握手是指建立一个 TCP 连接时需要客户端和服务器端总共发送三个包以确认连接的建立。

#### 握手的目标

三次握手的目的：

- **同步连接双方的 Sequence 序列号和确认号**
  - 初始序列号 ISN（Initial Sequence Number）
- **交换 TCP 窗口大小信息**
  - 如 MSS、窗口比例因子、选择性确认、指定校验和算法

在 Socket 编程中，这一过程由客户端执行 `connect` 来触发。

#### 三次握手流程图

```jsx | inline
import React from 'react';
import img from '../../assets/transport-layer-protocol/handshake.jpg';

export default () => <img alt="三次握手流程图" src={img} width={800} />;
```

1. **第一次握手**：**建立连接**。客户端发送连接请求报文段，将标志比特位 SYN 置为 1，随机产生一个序列号码 Sequence Number 值为 X（由操作系统动态随机选取一个 32 位长的序列号），并将该数据包发送给服务端，客户端进入 `SYN_SENT` 状态，等待服务端确认。
2. **第二次握手**：**服务端收到 SYN 报文段**。服务端收到数据包后需要对标志位 SYN 报文段进行确认，确认后设置确认号码 Acknowledgment Number 为 X+1（Sequence Number+1）；同时，自己还要发送 SYN 请求信息（以建立服务端对客户端的连接），将 SYN 设置为 1，设置 Sequence Number 值为 Y（由操作系统动态随机选取一个 32 位长的序列号），服务端将上述所有信息放到一个报文段（即 SYN+ACK 报文段）中，一并发送给客户端**以确认建立连接请求**，服务端进入 `SYN_RCVD` 状态。
3. **第三次握手**：**客户端收到服务端的 SYN+ACK 报文段**。确认后，然后将 Acknowledgment Number 设置为 Y+1，向服务端发送 ACK 报文段，这个报文段发送完毕后，客户端和服务器端进入 ESTABLISHED 状态，完成三次握手，随后客户端与服务器端之间可以开始传输数据了。

握手过程中传送的包里<span style="font-weight:bold;color:red">不包含数据</span>，只有三次握手完毕后，客户端与服务器才正式开始传送数据。理想状态下，TCP 连接一旦建立，在通信双方中的任何一方主动关闭连接之前，TCP 连接都将被一直保持下去。

#### 握手报文

SYN 报文

```jsx | inline
import React from 'react';
import img from '../../assets/transport-layer-protocol/tcp-handshaking-1.jpg';

export default () => <img alt="三次握手-1" src={img} width={640} />;
```

SYN/ACK 报文

```jsx | inline
import React from 'react';
import img from '../../assets/transport-layer-protocol/tcp-handshaking-2.jpg';

export default () => <img alt="三次握手-2" src={img} width={640} />;
```

ACK 报文

```jsx | inline
import React from 'react';
import img from '../../assets/transport-layer-protocol/tcp-handshaking-3.jpg';

export default () => <img alt="三次握手-3" src={img} width={640} />;
```

#### 其他问题

**未连接队列**

在三次握手协议中，服务器维护一个未连接队列，该队列为每个客户端的 SYN 包（syn=j）开设一个条目，该条目表明服务器已收到 SYN 包，并向客户端发出确认，正在等待客户端的确认包。这些条目所标识的连接在服务器处于 `SYN_RECV` 状态，当服务器收到客户端的确认包时，删除该条目，服务器进入 `ESTABLISHED` 状态。

**为什么建立 TCP 连接需要三次握手？**

主要是为了防止服务端开启无用的连接。

因为我们知道网络传输是有延时的，因为终端间隔了非常远的距离，数据包通过光纤以及各种中间代理服务器进行传输，但是在服务端和客户端的传输过程中，往往由于网络传输的不稳定原因丢失了数据包，客户端一直没有收到服务端返回的数据包，客户端可能设置了超时时间关闭了连接创建，那么就会再发起新的请求。如果没有第三次握手，服务端是不知道客户端到底有没有接收到服务端返回给他的数据的，客户端也没有一个确认说要关闭还是要创建这个请求，服务端的端口就一直开着，等着客户端发送实际的请求数据，那么这个时候开销就浪费了，服务端不知道这个连接已经创建失败了，可能客户端已经创建别的连接去了。

所以我们需要三次握手来确认这个过程，让服务端和客户端能及时察觉到网络原因导致的网络连接的关闭的问题，从而规避网络传输中因为延时导致导致的服务器开销问题。

**三次握手中的第一次握手可以携带数据吗？**

不可以，因为三次握手还没完成。

**第三次握手可以发送数据吗？为何？**

可以。因为能够发出第三次握手报文的主机，肯定接收到第二次（来自服务端）的握手报文。因为伪造 IP 的主机不会收到第二次报文。

**对方难道不可以将数据缓存下来，等握手成功后再提交给应用程序？**

这样会放大 SYN FLOOD 攻击。如果攻击者伪造了成千上万的握手报文，携带了 1K+ 字节的数据，而接收方会开辟大量的缓存来容纳这些巨大数据，内存会很容易耗尽，从而拒绝服务。

### 四次挥手

四次挥手即终止 TCP 连接，就是指断开一个 TCP 连接时，需要客户端和服务端总共发送 4 个包以确认连接的断开。在 Socket 编程中，这一过程由客户端或服务端任一方执行 `close` 来触发。

由于 TCP 连接是<span style="font-weight:bold;color:red">全双工</span>的，因此，每个方向都必须要**单独进行关闭**，这一原则是当一方完成数据发送任务后，发送一个 `FIN` 来终止这一方向的连接，收到一个 `FIN` 只是意味着这一方向上没有数据流动了，即不会再收到数据了，但是在这个 TCP 连接上仍然能够发送数据，直到这一方向也发送了 `FIN`。首先进行关闭的一方将执行主动关闭，而另一方则执行被动关闭。

**四次挥手流程图**

```jsx | inline
import React from 'react';
import img from '../../assets/transport-layer-protocol/wave.jpg';

export default () => <img alt="四次挥手流程图" src={img} width={800} />;
```

1. **第一次挥手**：客户端设置 Sequence Number，发送一个 `FIN` 报文段，用于关闭客户端到服务器端的数据传送，客户端进入 `FIN_WAIT_1` 状态。意思是说「我客户端没有数据要发给你了」，但是如果你服务器端还有数据没有发送完成，则不必急着关闭连接，可以继续发送数据。
2. **第二次挥手**：服务器端收到 `FIN` 报文段，回复 `ACK` 报文段，Acknowledgment Number 为 Sequence Number 加 1，告诉客户端，你的请求我收到了，我同意你的关闭请求。这个时候客户端就进入 `FIN_WAIT_2` 状态。
3. **第三次挥手**：当服务器端确定数据已发送完成，则向客户端发送 `FIN` 报文段，告诉客户端，好了，我这边数据发完了，准备好关闭连接了。服务器端进入 `LAST_ACK` 状态。
4. **第四次挥手**：客户端收到 `FIN` 报文段后，就知道可以关闭连接了，但是他还是不相信网络，怕服务器端不知道要关闭，所以发送 `ACK` 报文段回复服务端，然后进入 `TIME_WAIT` 状态，如果服务端端没有收到 `ACK` 则可以重传。服务器端收到 `ACK` 后，就知道可以断开连接了。客户端等待了 2MSL（通常是两分钟）后依然没有收到回复，则证明服务器端已正常关闭，那好，我客户端也可以关闭连接了。最终完成了四次握手。

> MSL（Maximum Segment Lifetime）报文最大生存时间。维持 2MSL 时长的 TIME-WAIT 状态，保证至少一次报文的往返时间内端口是不可复用。

- 第一次挥手是**服务端确认客户端需要断开连接**
- 第二次挥手是**客户端确认服务器接收断开请求**
- 第三次挥手是**客户端确认服务器数据发完，断开连接**
- 第四次挥手是**服务端确认客户端断开连接，断开连接**

所以如果服务端的数据全部发送完，是没有第三次挥手，直接进入第四次挥手。

**为什么断开 TCP 连接需要四次挥手？**

由于 TCP 连接采取全双工的通信方式，因此每个方向都必须单独进行关闭，这个原则是当一方完成它的数据发送任务后就能发送一个 `FIN` 来终止这个方向的连接。收到一个 `FIN` 只意味着这一方向上没有数据流动，一个 TCP 连接在收到一个 `FIN` 后仍能发送数据。首先进行关闭的一方将执行主动关闭，而另一方执行被动关闭。

**为什么基于 TCP 的程序往往都有个应用层的心跳检测机制？**

TCP 建立链接后，只是在两端的内核里面维持 TCP 信息，实际上并没有一个物理的连接通路，对端这个时候挂了，谁也不知道。

### 重传机制

### 拥塞控制机制

### 流量控制机制

### 可靠传输机制

## UDP

用户数据报协议（User Datagram Protocol，UDP），又称使用者资料包协定，是一个简单的面向数据包的传输层协议，正式规范为 RFC 768。

在 TCP/IP 模型中，UDP 为网络层以上和应用层以下提供了一个简单的接口。UDP 只提供数据的不可靠传递，它一旦把应用程序发给网络层的数据发送出去，就不保留数据备份（所以 UDP 有时候也被认为是不可靠的数据报协议）。UDP 在 IP 数据报的头部仅仅加入了复用和数据校验（字段）。

### 特点

- <strong style="color:red">无需建立连接</strong>（减少延迟）
- 尽最大努力交付，即不保证可靠交付，因此主机不需要维持复杂的链接状态
- UDP 的首部开销小，只有 8 个字节，比 TCP 的 20 个字节的首部要短
- 没有拥塞控制，因此网络出现拥塞不会使源主机的发送速率降低（对实时应用很有用，如直播，实时视频会议等）
- 支持一对一、一对多、多对一和多对多的交互通信

### 实践

**基于 UDP 协议的有：**

- 域名系统（DNS）
- 简单网络管理协议（SNMP）
- 动态主机配置协议（DHCP）
- 路由信息协议（RIP）
- 自举协议（BOOTP）
- 简单文件传输协议（TFTP）

## 数据通信形式

- 单工数据传输只支持数据在一个方向上传输
- 半双工数据传输允许数据在两个方向上传输，但是，在某一时刻，只允许数据在一个方向上传输，它实际上是一种切换方向的单工通信
- 全双工数据通信允许数据同时在两个方向上传输，因此，**全双工通信是两个单工通信方式的结合，它要求发送设备和接收设备都有独立的接收和发送能力**

## TCP 与 UDP

|          | TCP                   | UDP                      |
| :------- | :-------------------- | :----------------------- |
| 连接性   | 面向连接              | 无连接                   |
| 双工性   | 全双工（1:1）         | `n:m`                    |
| 可靠性   | 可靠（重传机制）      | 不可靠（丢包后数据丢失） |
| 有序性   | 有序（通过 SYN 排序） | 无序                     |
| 有界性   | 无，有沾包情况        | 有消息边界，无沾包       |
| 拥塞控制 | 有                    | 无                       |
| 传输速度 | 慢                    | 快                       |
| 量级     | 低                    | 20-60 字节               |
| 头部大小 | 高                    | 8 字节                   |

## 参考资料

- [📖 维基百科：传输控制协议](https://zh.wikipedia.org/wiki/%E4%BC%A0%E8%BE%93%E6%8E%A7%E5%88%B6%E5%8D%8F%E8%AE%AE)
- [📝 TCP 协议详解](https://juejin.im/post/5ba895a06fb9a05ce95c5dac)
- [📝 一篇文章带你熟悉 TCP/IP 协议](https://juejin.im/post/5a069b6d51882509e5432656)
- [📝 前端必须懂的计算机网络知识](https://juejin.im/post/5ba3b68c6fb9a05d287345ab)
- [📝 就是要你懂 TCP](http://jm.taobao.org/2017/06/08/20170608/)
- [📝 TCP / IP 协议没有哪篇文章能讲的这么明明白白！](https://segmentfault.com/a/1190000024571707)