> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/LeeQiang8023/article/details/105896390)

> 在学习IP地址和子网划分前，必须对进制计数有一定了解，尤其是二进制和十进制之间的相互转换，对于我们掌握IP地址和子网的划分非常有帮助，可参看如下目录详文。IP地址和子网划分学习笔记相关篇章：1、IP地址和子网划分学习笔记之《预备知识：进制计数》2、IP地址和子网划分学习笔记之《IP地址详解》3、IP地址和子网划分学习笔记之《子网掩码详解》4、IP地址和子网划分学习笔记之《子网划分详解》...

在学习 IP 地址和子网划分前，必须对进制计数有一定了解，尤其是二进制和十进制之间的相互转换，对于我们掌握 IP 地址和子网的划分非常有帮助，可参看如下目录详文。

> **IP 地址和子网划分学习笔记相关篇章：**
> 
> **[1、IP 地址和子网划分学习笔记之《预备知识：进制计数》](https://blog.csdn.net/LeeQiang8023/article/details/105896270)**
> 
> **[2、IP 地址和子网划分学习笔记之《IP 地址详解》](https://blog.csdn.net/LeeQiang8023/article/details/105896390)**
> 
> **[3、IP 地址和子网划分学习笔记之《子网掩码详解》](https://blog.csdn.net/LeeQiang8023/article/details/105896584)**
> 
> **[4、IP 地址和子网划分学习笔记之《子网划分详解》](https://blog.csdn.net/LeeQiang8023/article/details/105896492)**
> 
> **[5、IP 地址和子网划分学习笔记之《超网合并详解》](https://blog.csdn.net/LeeQiang8023/article/details/105896531)**

一、IP 地址和 MAC 地址
---------------

### 1、MAC 地址

MAC（Media Access Control，介质访问控制）地址，或称为物理地址，也叫硬件地址，用来定义网络设备的位置，MAC 地址是网卡出厂时设定的，是固定的（但可以通过在设备管理器中或注册表等方式修改，同一网段内的 MAC 地址必须唯一）。MAC 地址采用十六进制数表示，长度是 6 个字节（48 位），分为前 24 位和后 24 位。

> 1、前 24 位叫做组织唯一标志符（Organizationally Unique Identifier，即 OUI），是由 IEEE 的注册管理机构给不同厂家分配的代码，区分了不同的厂家。  
> 2、后 24 位是由厂家自己分配的，称为扩展标识符。同一个厂家生产的网卡中 MAC 地址后 24 位是不同的。

MAC 地址对应于 OSI 参考模型的第二层数据链路层，工作在数据链路层的交换机维护着计算机 MAC 地址和自身端口的数据库，交换机根据收到的数据帧中的 “目的 MAC 地址” 字段来转发数据帧。

### 2、IP 地址

IP 地址（Internet Protocol Address），缩写为 IP Adress，是一种在 Internet 上的给主机统一编址的地址格式，也称为网络协议（IP 协议）地址。它为互联网上的每一个网络和每一台主机分配一个逻辑地址，常见的 IP 地址，分为 IPv4 与 IPv6 两大类，当前广泛应用的是 IPv4，目前 IPv4 几乎耗尽，下一阶段必然会进行版本升级到 IPv6；如无特别注明，一般我们讲的的 IP 地址所指的是 IPv4。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS41MWN0by5jb20vaW1hZ2VzL2Jsb2cvMjAxODA1LzA0LzAxYjkzYTFkMGFjYWM1MmNjMGJkNDg3ODY5NmQ0MDk4LnBuZw?x-oss-process=image/format,png)

IP 地址对应于 OSI 参考模型的第三层网络层，工作在网络层的路由器根据目标 IP 和源 IP 来判断是否属于同一网段，如果是不同网段，则转发数据包。

### 3、IP 地址格式和表示

> 在计算机二进制中，1 个字节 = 8 位 = 8bit（比特）

#### ①IP 地址格式和表示

IP 地址 (IPv4) 由 32 位二进制数组成，分为 4 段（4 个字节），每一段为 8 位二进制数（1 个字节）  
每一段 8 位二进制，中间使用英文的标点符号 “.” 隔开

由于二进制数太长，为了便于记忆和识别，把每一段 8 位二进制数转成十进制，大小为 0 至 255。  
IP 地址的这种表示法叫做 “**点分十进制表示法**”。  
IP 地址表示为：xxx.xxx.xxx.xxx  
举个栗子：210.21.196.6 就是一个 IP 地址的表示。

#### ②理解 2 的指数幂

2 的幂也称为 2 的指数，还可以称为 2 的次方，如 2 的 2 次方、2 的 3 次方等等，任何数的 0 次方都等于 1。  
在 IP 地址中，0 次方到 7 次方刚好为 8 位，这对于 IP 地址二进制转换为十进制非常方便。  
举个栗子：11010010 = 1×27+1×26+0×25+1×24+0×23+0×22+1×21+0×20 = 128+64+0+16+0+0+2+0 = 210

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS41MWN0by5jb20vaW1hZ2VzL2Jsb2cvMjAxODA1LzAzL2ZlYTNmYWM4YmE4OTA4NTRkYmZkZGUzNDkyNmUwNGFjLnBuZw?x-oss-process=image/format,png)

我们需要记住上图的 2 的幂的结果，不需要死记硬背，这个是有技巧的，从上图来看，很容易发现，由于是 2 的幂，所有相邻的幂的前后都是相差 2 倍，所以只要知道其中一个幂值，就知道相邻的幂的值。

### 4、IP 地址的组成

IP 地址 = 网络地址 + 主机地址，比如：  
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS41MWN0by5jb20vaW1hZ2VzL2Jsb2cvMjAxODA1LzAzL2FjM2M2NTk4ZGQyNGIyNWRjOWIwMWJiNjBiMTVkNzI1LnBuZw?x-oss-process=image/format,png)  
计算机的 IP 地址由两部分组成，一部分为网络标识，一部分为主机标识，同一网段内的计算机网络部分相同，主机部分不同同时重复出现。路由器连接不同网段，负责不同网段之间的数据转发，交换机连接的是同一网段的计算机。通过设置网络地址和主机地址，在互相连接的整个网络中保证每台主机的 IP 地址不会互相重叠，即 IP 地址具有了唯一性。  
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS41MWN0by5jb20vaW1hZ2VzL2Jsb2cvMjAxODA1LzA0LzYzMTc0NjEyZWUxYWQ0YjQ0NDgwOTY1MDM0YTU2MTg3LnBuZw?x-oss-process=image/format,png)

### 5、IP 地址与 MAC 地址区别

*   长度不同：IP 地址为 32 位（二进制），MAC 地址为 48 位（十六进制）。
*   分配依据不同：IP 地址的分配是基于网络拓扑，MAC 地址的分配是基于制造商。
*   寻址协议层不同：IP 地址应用于 OSI 第三层（网络层），而 MAC 地址应用在 OSI 第二层（数据链路层）。

### 6、IP 地址与 MAC 地址的作用和关系

IP 和 MAC 两者之间分工明确，默契合作，完成通信过程。在数据通信时，IP 地址专注于网络层，网络层设备（如路由器）根据 IP 地址，将数据包从一个网络传递转发到另外一个网络上；而 MAC 地址专注于数据链路层，数据链路层设备（如交换机）根据 MAC 地址，将一个数据帧从一个节点传送到相同链路的另一个节点上。IP 和 MAC 地址这种映射关系由 ARP（Address Resolution Protocol，地址解析协议）协议完成，ARP 根据目的 IP 地址，找到中间节点的 MAC 地址，通过中间节点传送，从而最终到达目的网络。  
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS41MWN0by5jb20vaW1hZ2VzL2Jsb2cvMjAxODA1LzA0LzNjOTc3MGI3OTM2YjI3ZDJiOTU1YjE3MDNkMTNkYmJiLnBuZw?x-oss-process=image/format,png)

> 计算机在和其他计算机通信之前，首先要判断目标 IP 地址和自己的 IP 地址是否在一个网段，这决定了数据链层的目标 MAC 地址是目标计算机的还是路由器接口的 MAC 地址。数据包的目标 IP 地址决定了数据包最终到达哪一个计算机，而目标 MAC 地址决定了该数据包下一跳由哪个设备接收，不一定是终点。

二、IP 地址的分类
----------

### 1、IP 地址分类详解

IP 地址分 A、B、C、D、E 五类，其中 A、B、C 这三类是比较常用的 IP 地址，D、E 类为特殊地址。

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS41MWN0by5jb20vaW1hZ2VzL2Jsb2cvMjAxODA1LzA0L2Q4ZWRhZmViY2E1YmJiZjFkNWJiMzVjZWY0MTU2MDI2LnBuZw?x-oss-process=image/format,png)

#### ①、A 类地址

1.  A 类地址第 1 字节为网络地址（最高位固定是 0），另外 3 个字节为主机地址。
2.  A 类地址范围：1.0.0.0 - 126.255.255.255，其中 0 和 127 作为特殊地址。
3.  A 类网络默认子网掩码为 255.0.0.0，也可写作 / 8。
4.  A 类网络最大主机数量是 256×256×256-2=166777214（减去 1 个主机位为 0 的网络地址和 1 个广播地址）。

> 在计算机网络中，主机 ID 全部为 0 的地址为网络地址，而主机 ID 全部为 1 的地址为广播地址，这 2 个地址是不能分配给主机用的。

#### ②、B 类地址

1.  B 类地址第 1 字节（最高位固定是 10）和第 2 字节为网络地址，另外 2 个字节为主机地址。
2.  B 类地址范围：128.0.0.0 - 191.255.255.255。
3.  B 类网络默认子网掩码为 255.255.0.0，也可写作 / 16。
4.  B 类网络最大主机数量 256×256-2=65534。

#### ③、C 类地址

1.  C 类地址第 1 字节（最高位固定是 110）、第 2 字节和第 3 个字节，另外 1 个字节为主机地址。
2.  C 类地址范围：192.0.0.0 - 223.255.255.255。
3.  C 类网络默认子网掩码为 255.255.255.0，也可写作 / 24。
4.  C 类网络最大主机数量 256-2=254。

#### ④、D 类地址

1.  D 类地址不分网络地址和主机地址，它的第 1 个字节的最高位固定是 1110。
2.  D 类地址用于组播（也称为多播）的地址，无子网掩码。
3.  D 类地址范围：224.0.0.0 - 239.255.255.255。

#### ⑤、E 类地址

1.  E 类地址也不分网络地址和主机地址，它的第 1 个字节的最高位固定是 11110。
2.  E 类地址范围：240.0.0.0 - 255.255.255.255。
3.  其中 240.0.0.0-255.255.255.254 作为保留地址，主要用于 Internet 试验和开发，255.255.255.255 作为广播地址。

### 2、IP 地址分类思维导图

### 2、IP 地址分类思维导图

IP 地址总结学习思维导图如下：  
![](https://img-blog.csdnimg.cn/20200504103518349.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0xlZVFpYW5nODAyMw==,size_16,color_FFFFFF,t_70)

三、保留的特殊 IP 地址
-------------

以下这些特殊 IP 地址都是不能分配给主机用的地址：

*   主机 ID 全为 0 的地址：特指某个网段，比如：192.168.10.0 255.255.255.0，指 192.168.10.0 网段。
*   主机 ID 全为 1 的地址：特指该网段的全部主机，比如：192.168.10.255，如果你的计算机发送数据包使用主机 ID 全是 1 的 IP 地址，数据链层地址用广播地址 FF-FF-FF-FF-FF-FF。
*   127.0.0.1：是本地环回地址，指本机地址，一般用来测试使用。回送地址 (127.x.x.x) 是本机回送地址(Loopback Address)，即主机 IP 堆栈内部的 IP 地址。
*   169.254.0.0：169.254.0.0-169.254.255.255 实际上是自动私有 IP 地址。
*   0.0.0.0：如果计算机的 IP 地址和网络中的其他计算机地址冲突，使用 ipconfig 命令看到的就是 0.0.0.0，子网掩码也是 0.0.0.0。

保留的特殊 IP 地址思维导图如下：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS41MWN0by5jb20vaW1hZ2VzL2Jsb2cvMjAxODA1LzA0L2U2NjRkMTgxYmU4Njk3MjI5YjUyZjJhZjIzNmFmYWU3LnBuZw?x-oss-process=image/format,png)

四、公网和私网 IP 地址
-------------

**公网 IP 地址**  
公有地址分配和管理由 Inter NIC（Internet Network Information Center 因特网信息中心）负责。各级 ISP 使用的公网地址都需要向 Inter NIC 提出申请，有 Inter NIC 统一发放，这样就能确保地址块不冲突。

**私网 IP 地址**  
创建 IP 寻址方案的人也创建了私网 IP 地址。这些地址可以被用于私有网络，在 Internet 没有这些 IP 地址，Internet 上的路由器也没有到私有网络的路由表。

*   A 类：10.0.0.0 255.0.0.0，保留了 1 个 A 类网络。
*   B 类：172.16.0.0 255.255.0.0～172.31.0.0 255.255.0.0，保留了 16 个 B 类网络。
*   C 类：192.168.0.0 255.255.255.0～192.168.255.0 255.255.255.0，保留了 256 个 C 类网络。

PS：私网地址访问 Internet 需要做 NAT 或 PAT 网络地址转换  
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS41MWN0by5jb20vaW1hZ2VzL2Jsb2cvMjAxODA1LzA0LzhhNTNlNmI5YzEwNTFiZDViZGEwODM2NGRkMWVhNGIxLnBuZw?x-oss-process=image/format,png)

公网和私网 IP 地址思维导图如下：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS41MWN0by5jb20vaW1hZ2VzL2Jsb2cvMjAxODA1LzA0L2IwMDYzOGZlZjc5ODYzYzBlMTk1OGJiOTEyYTUyZDdkLnBuZw?x-oss-process=image/format,png)

本篇 end，收工~

**© 著作权归作者所有：来自 51CTO 博客作者 MrHuaZi 的原创作品 https://blog.51cto.com/6930123/2111068，如需转载，请与作者联系，否则将追究法律责任**