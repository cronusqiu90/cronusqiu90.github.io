# 网络协议


### 协议三要素
• 语法，就是这一段内容要符合一定的规则和格式。例如，括号要成对，结束要使用分号等。

• 语义，就是这一段内容要代表某种意义。例如数字减去数字是有意义的，数字减去文本一般来说就没有意义。

• 顺序，就是先干啥，后干啥。例如，可以先加上某个数值，然后再减去某个数值。


### 常用的网络协议
应用层: DHCP、HTTP、HTTPS、RTMP、P2P、DNS、GTP、RPC

传输层: TCP、UDP

网络层：ICMP、IP、OSPF、BGP、IPSec、GRE

链路层: ARP、VLAN、STP、RARP

物理层: ~~

### 数据如何传输
在浏览器中输入URL (www.baidu.com) 到显示页面之间，做了哪些事情？
1. 浏览器只知道名字是 `www.baidu.com`，但是不知道具体的地点，所以不知道应该如何访问。于是，它打开地址簿去查找。可以使用一般的地址簿协议DNS去查找，还可以使用另一种更加精准的地址簿查找协议HTTPDNS。
   
2. 无论用哪一种方法查找，最终都会得到这个地址：`36.155.132.3`。这个是IP地址，是 baidu 在互联网世界的地址。
   
3. 浏览器就开始打包它的请求。在应用层使用`HTTP` 或者 `HTTPS` 协议,无论是什么协议，里面都会写明“你要搜索什么？”
```
HTTP  -------------------------
     | POST, URL, HTTP 1.1   |  
     | 格式:JSON，长度:1111   |
     -------------------------
     | 你要搜索什么?          |
     -------------------------
```

4. 传输层有两种协议，UDP和TCP。主要封装浏览器的端口和目标服务器的端口：
```
TCP  -------------------------
     | 浏览器端口: 19000      |  
     | 百度端口： 443         |
HTTP  -------------------------
     | POST, URL, HTTP 1.1   |  
     | 格式:JSON，长度:1111   |
     -------------------------
     | 你要搜索什么?          |
     -------------------------
```

5. 传输层使用的是 IP 协议。
```
IP   -------------------------
     | 本机IP地址: 192.168.1.100 |  
     | 百度IP地址： 172.16.17.32 |
TCP  -------------------------
     | 浏览器端口: 19000      |  
     | 百度端口： 443         |
HTTP  -------------------------
     | POST, URL, HTTP 1.1   |  
     | 格式:JSON，长度:1111   |
     -------------------------
     | 你要搜索什么?          |
     -------------------------
```

6. MAC层。 网卡再将包发出去。由于这个包里面是有 MAC 地址的，因而它能够到达网关。
```
MAC  -------------------------
     | 本机MAC:  xxx.xxx.xxx.xxx |  
     | 网关MAC： xxx.xxx.xxx.xxx |
IP   -------------------------
     | 本机IP地址: 192.168.1.100 |  
     | 百度IP地址： 172.16.17.32 |
TCP  -------------------------
     | 浏览器端口: 19000      |  
     | 百度端口： 443         |
HTTP  -------------------------
     | POST, URL, HTTP 1.1   |  
     | 格式:JSON，长度:1111   |
     -------------------------
     | 你要搜索什么?          |
     -------------------------
```

7. 网关收到包之后，会根据自己的知识，判断下一步应该怎么走。网关往往是一个路由器，到某个 IP 地址应该怎么走，这个叫作路由表。
路由器有点像高速公路的收费站。每个收费站都连着不同城市，每个城市相当于一个局域网，在局域网内部通过MAC地址进行通行。

8. 一旦到达一个收费站，就需要拿出IP头，里面写了我来自哪里，我要去哪里，路过此地，请问接下来怎么走？
```
    去IP段1， 走网口1，下一站为路由器A
    去IP段2， 走网口2，下一站为路由器B
    去IP段3， 走网口3，下一站为路由器C
```
收费站往往知道这些，因为临近的收费站之间也会经常沟通。到哪里应该怎么走，这种沟通的协议成为路由协议，也就是 OSPF 和 BGP。

9. 网络包出了收费站后，进入一个城市，通过 MAC 地址来找到目标服务器。 
MAC 地址对上后，取下MAC头，发送给操作系统的网络层；
IP 也对上了，就取下 IP 头。IP 头里会写上一层封装的是 TCP 协议，然后将其交给传输层，即TCP层；


### 网络为什么要分层
因为，是个复杂的程序都要分层。就像你设计一个系统，搞数据库层、缓存层、Compose、Controller、Gateway，每层就只做自己负责的事情。

### 程序如何工作
![](/img/network/program_howto_work.png)

1. 当一个网络包从一个网口经过的时候，你看到了，首先先看看要不要请进来，处理一把。 

2. 收进来后调用`process_layer2(buffer)` （假设有这么一个函数）,摘掉二层的头，看看里面的`MAC`地址是不是自己，如果是说明是给自己的；  

3. 调用`process_layer3(buffer)`，摘掉三层的头，看看里面的 `IP` 地址是不是自己，如果是说明是发给自己的，不然就是转发出去的；根据三层的头来判断，是`TCP` 还是 `UDP`;   

4. 如果是 `TCP`, 调用 `process_tcp(buffer)`， 摘掉四层的头，看是如果是发起或者应答，发送一个回复包，如果是正常数据包，根据头里的端口，传递给上层应用处理；  

5. 浏览器获取到数据包，解析HTML，显示页面；   

6. 用户在浏览器页面点击了一个按钮，浏览器就又发起一个新的 HTTP 请求，于是：   ---&gt;
```text
send_tcp(buffer) 加HTTP头 
-&gt; send_layer3(buffer)  加TCP头 
-&gt; send_layer2(buffer) 加MAC头 
-&gt; send_network(buffer) 发送出去。
```

**与现实世界中的例子来比喻，就很像 Boss 说了一句话定了个基调， 然后总经理补充框架，然后各部门经理补充细节，员工最后感慨一下；**

**还有一个原则很重要：只要是在网络上跑的包，都是完整的。可以有下层没上层，绝对不可能有上层没下层。**


---

> 作者: 迷途小狼崽  
> URL: http://localhost:1313/posts/netword/%E5%8D%8F%E8%AE%AE/  

