# TCP 协议介绍



### 基础

#### 分组窗口和滑动窗口

![窗口](/img/network/tcp/windows.png)


上图显示了，当前三个分组窗口，整个窗口的大小是 3。

左侧分组`(编号为3)`是已经发送且确认的；

右侧分组`(编号为7)`已经准备好但未发送；

中间分组`(编号4,5,6)`代表的已经发送，但还未确认；

假设发送方下一步收到 `编号4` 的 `ACK`, 则窗口向右滑动一格，即窗口的范围变成 `(编号5,6,7)`, 

这个过程就叫  `滑动窗口`。

&gt; 发送方和接收方都各自维护滑动窗口。
&gt; 
&gt; 发送方用它记录哪些分组可被释放,哪些分组正在等待ACK,以及哪些分组还不能发送；
&gt; 
&gt; 接收方用它记录哪些分组已经被接收和确认，哪些分组是下一步期望的，以及哪些分组要被丢弃的;


#### 变量窗口： 流量控制和拥塞控制

`流量控制`：防止发送方发送过快，超出接收方的处理能力，在接收方跟不上时强迫发送方慢下来的方法。

1. 速率： 给发送方指定某个速率，同时确保数据永远不能超过这个速率发送；

2. 窗口： 窗口大小不固定，随时间而变动，接收方通过`窗口通告`来通知发送方动态调整窗口大小，限制发送方的速率不会超过接收方的处理能力，可以更好的保护接收方。

但发送和接收中间的网络就复杂了，很可能存在有限内存的路由器且能力低于窗口的大小，从而导致丢包，这种情况需要特殊的控制方法：拥塞控制。


#### TCP 头部结构

`TCP` 在 `IP 数据报`中的封装:
![ip_datagram_with_tcp_segment](/img/network/tcp/ip数据报_tcp.png)

----

头部数据结构：
![tcp_header](/img/network/tcp/head.png)


- `Source Port`和`Destination Port`：标识了发送方和接收方的端口号；
- `Sequence Number`：标识了发送方发送的字节流的起始位置；
- `Acknowledgment Number`：标识了接收方期望收到的下一个字节的序列号；

- `Flags`：标志字段，用于控制 TCP 连接的状态；
  - `Header Length`：表示头部的长度，单位为 4 字节；
  - `Resv`：保留字段, 暂时未使用, `0`;
  - `CWR`: 拥塞窗口减少;
  - `ECE`: 显式拥塞通知;
  - `URG`: 紧急指针，很少使用;
  - `ACK`：确认号有效，表示接收方期望收到的下一个字节的序列号；
  - `PSH`：推送，表示接收方应该尽快将数据交给应用层(没被可靠地实现或用到);
  - `RST`：重置连接，表示连接中出现严重错误；
  - `SYN`：初始化一个连接的同步序列号；
  - `FIN`：结束向对方发送数据；

- `Window Size`：窗口大小，表示发送方的缓冲区大小， 16 位字段,上限是 65535 字节 (`Options`中有`Window Scale`来适应高速或者低速网络)
- `Checksum`：校验和，用于检测数据是否损坏；
- `Urgent Pointer`：紧急指针，用于通知接收方有紧急数据需要处理；
- `Options`：可选字段，用于扩展 TCP 功能， 常见的就是 `MSS`, 还可能用到 `SACK`、时间戳、窗口缩放等；



### 连接管理

#### 三次握手

之所以要发送 3 个报文段来完成连接的建立，目的不仅在于让通信双方了解一个连接正在建立，还在于利用数据包的来交换`初始序列号 (Initial SEquence Number, ISN)`

![TCP头](/img/network_proto_tcp_handlshark.png)

- 第一次握手

    1. 客户端发起，设置标志位:`SYN=1, seq=J`并发送；
    2. 发送完毕后，自身进入`SYN_SEND`状态，等待服务器确认；

- 第二次握手

    1. 服务器端收到数据包后, 设置标志位`SYN=1, ACK=1, seq=K, ack=J&#43;1`并发送；
    2. 发送完毕后，自身进入`SYNC_RCVD`状态，等待客户端确认；

- 第三次握手

    1. 客户端收到确认后，校验 `ack` `ACK` 字段是否正确；
    2. 如果正确, 设置标志位`ACK=1,ack=K&#43;1`并发送；
    3. 发送完毕后，客户端进入`ESTABLISHED`状态；
    4. 服务端在收到这个确认包，也进入`ESTABLISHED`；

&gt; 整个过程可以想象一下打电话的过程，很类似.

完成了三次握手后，双方都进入 `ESTABLISHED` 状态，可以开始传输数据。


#### 四次挥手

由于TCP连接是全双工模式，需要客户端和服务端总共发送4个包以确认连接的断开，每个方向都必须要单独进行关闭；

![TCP头](/img/network_proto_tcp_close.png)

双方都可以主动发起挥手请求，假设由客户端发起请求：
- 第一次挥手
  
    1. 客户端主动发起挥手请求，发送`FIN=1, ACK=1, seq=u, ack=v`；
    2. 发送完毕后，自身进入`FIN_WAIT_1`状态, 等待服务器确认；

- 第二次分手
  
    1. 服务端收到后，回复`ACK=1, seq=v, ack=u&#43;1`；
    2. 发送完毕后，自身进入`CLOSE_WAIT`状态，等待客户端确认；
    3. 客户端收到确认报文段，进入`FIN_WAIT_2`状态；

- 第三次分手
  
    1. 服务端发送`FIN=1, ACK=1, seq=w, ack=u&#43;1`；
    2. 发送完毕后，服务端进入`LAST_ACK`状态，等待客户端确认；

- 第四次分手
  
    1. 客户端收到后，回复`ACK=1, seq=u&#43;1, ack=w&#43;1`；
    2. 发送完毕后，客户端进入`TIME_WAIT`状态。
    3. 服务端收到客户端的确认报文段，关闭自身连接。
    4. 客户端开始等待 2 个 `MSL` (大概4分钟)的时间， 期间如果没有收到任何回复，客户端自身关闭。

&gt; **为什么要设置等待 2 个 `MSL` 时间呢 ?**

    客户端回复`确认`报文段后，直接进入`CLOSED`状态，而不是进入`TIME_WAIT`状态，此时
    如果`确认`报文段丢失了，服务端会触发**超时重传**，此时服务端仍处于`LAST_ACK`；
    当重传的`连接释放`报文段到达客户端，但客户端已经处于`CLOSED`状态，因此不理睬，
    服务端就会一直处于 `LAST_ACK`状态，并反复重传`连接释放`的报文段，无法进入`CLOSED`状态;
    因此进入`TIME_WAIT`状态并等待 `2MSL` 的时长，是为了确保服务端可以收到最后一个`确认`报文段，
    从而进入`CLOSED`状态;
    
    等待 `2MSL`的时长，更多的是从工程上考虑，但就目前的网络情况，等待 `2MSL` 确实有点长；


----

Wireshark 抓包截图:

终端执行命令:
```shell
C:\Users\Administrator&gt; telnet 36.155.132.3 80  # 这个是baidu的IP地址哈
Trying 36.155.132.3...
Connected to 36.155.132.3.
Escape character is &#39;^]&#39;.

# wait for a seconds and press `ctrl&#43;]`

telnet&gt; quit
Connection closed.
```


![tcp连接断开抓包](/img/network/tcp/tcp_conn_close_cap.png)

----


### 超时与重传

`TCP` 拥有两套独立机制来完成重传，一是`基于时间`，二是`基于确认信息的构成`，后者会比较高效。


`TCP` 在发送数据时会设置一个计时器，若至计时器超时仍未收到数据确认信息，则会引发相应的`超时`或`基于计时器的重传操作`, 计时器超时称为 `重传超时 (Retransmission Timeout, RTO)`。另一种方式的重传称为`快速重传`，通常发生在没有时延的情况下，即在`3个重复ACK` (ACK阈值, `dupthresh`，通常设置为 3) 后，会触发快速重传。

### 流量控制

每个`TCP`头部的`Window Size`字段表明接收端可用缓存空间的大小，发送端根据这个字段来决定发送速率。

采用可变滑动窗口来实现流量控制。

`TCP` 连接双方都可收发数据，连接的两段都维护一个 `发送窗口结构 (send window strucure)` 和 `接收窗口结构 (receive window structure)`。

#### 发送窗口结构

![发送窗口](/img/network/tcp/send_window_structure.png)

由接收端通告的窗口称为`提供窗口 (offered window)`，包含编号 4 ~ 9 字节。接收端已成功确认包括第 3 字节在内的之前的数据，并通告了一个 6 字节大小的窗口。随着时间推移，当接收到返回的数据 `ACK`， 滑动窗口也随之向右移动。

每个`TCP`报文段都包含`ACK`号和窗口通告信息，发送端可以据次调节窗口大小。
- 窗口左边界不能左移；
- `ACK`号增大，窗口大小保持不变，则称为窗口向前滑动;
- `ACK`号增大，窗口却缩小，则左右边界距离减小，当左右边界相等时，称为`零窗口`，发送端就不能在发送数据了，转变成开始`探测`对方窗口，来增大窗口大小。


#### 接收窗口结构

![接收窗口](/img/network/tcp/recv_window_structure.png)

接收端的窗口结构就简单多了，记录了已接收并确认的数据，以及它能够接收的最大序列号。

- 序列号 &lt; 左边界，认为是重复数据，丢弃；
- 序列号 &gt; 右边界，超出处理范围，丢弃；
- 序列号 == 左边界，数据才会被处理，窗口开始滑动；

#### TCP 持续计时器

当出现零窗口后，窗口更新的 `ACK` 丢失了，双方又处于一直等待状态等情况： 接收方等待接收数据，发送方等待窗口更新告知。

为了防止出现这种死锁情况，发送端会启动一个`持续计时器`来不断查询接收端，看窗口是否已经增长。`持续计时器`会触发`窗口探测 (window probe)` 的传输，强制要求接收端返回 `ACK`。

### 拥塞控制

后面单独介绍吧!



---

> 作者: 迷途小狼崽  
> URL: https://cronusqiu90.github.io/posts/network/tcp/  
