---
title: FreeRTOS-TCP/IP-相关基本概念
date: 2023/4/2
categories: 
- TCP/IP
---

<center>
FreeRTOS-TCP/IP-相关基本概念
</center>

<!--more-->

***

**Ethernet Address**：
在同一个以太网中，通过MAC地址（media access control address，硬件地址）来识别不同的节点。MAC地址为 6 字节，由设备厂商配置。

数据以以太网帧（ethernet frames）数据格式，从本地以太网的一个节点传输到另一个节点。
![测试图片描述](./TCPIP-basic-conception-draft/640px-Ethernet_Type_II_Frame_format.svg.png)
<center>

图片来自 [wikipedia](https://en.wikipedia.org/wiki/Ethernet_frame)
</center>
通过目标地址指明数据接收方，源地址表明数据发送方。
ether type指明负载使用（payload）使用的具体协议。例如，0x0800表明负载数据为IP协议数据，0x0806表明负载数据为ARP协议数据。

<br/>

**MTU**（Maximum Transmission Unit）：
上图以太网帧数据中，头部字段为固定的14字节，尾部校验为固定4字节。
负载数据部分长度为 46-1500 字节。 其中，46字节最小数据，是由于以太网使用的CSMA/CD中冲突检测机制的需要。 而1500最大长度，又出于发送效率和时间的平衡（负载数据长度越大，头部固定长度占的比例越小，传输效率更高。但负载数据长度越大，传输时间越长，由于共享总线，发送期间其它节点无法发送数据，可能造成其它节点数据的拥堵）。
不同的硬件支持的 MTU 可能不同（可能小于1500），实际使用需要根据系统具体情况配置参数。

当上层协议数据（封包后的）长度大于链路层的 MTU 时，数据需要拆分为多个包发送。

**IP**（Internet Protocol）：
IP 协议是用于跨网络（inter-networking）通信的协议，例如互联网（Internet）。IP 网络中的节点使用 IP地址来标识自己。
IP数据包在以太网帧中传输，并且其自身可以传输TCP/UDP数据。

**IP Address**：
IP 网络中的不同节点，通过 IP地址来标识。
Static IP Address：手动配置的IP地址，设备启动时使用设定好的地址。
动态 IP地址：设备启动时，向DHCP（Dynamic Host Configuration Protocol）服务器请求 IP地址。

**ARP**（Address Resolution Protocol）：
IP数据包是基于 IP地址进行数据发送/接收的，但IP数据包最终是作为以太网帧的负载来传输的，而以太网帧是基于MAC地址在网络节点间传输的。因此，在以太网上实际传输 IP数据包前，需要知道目的IP地址所对应的MAC地址。
地址解析协议（ARP）就用通过IP地址获得MAC地址的协议，FreeRTOS-TCP协议栈在ARP表中存储了IP地址到MAC地址的映射。







<br/>
<br/>

#### 参考连接：
【1】[https://en.wikipedia.org/wiki/Ethernet_frame](https://en.wikipedia.org/wiki/Ethernet_frame)









