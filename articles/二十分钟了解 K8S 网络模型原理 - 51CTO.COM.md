> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.51cto.com](https://www.51cto.com/article/710970.html)

> ​对于好多刚接触 K8S，甚至是接触 K8S 很长时间的同学，K8S 网络模型可以说是个很神秘的东西。那么对于这部分同学，恭喜你发现了本文，只要你花二十分钟的时间，就保证你能轻松掌握 K8S 网络模型原理。

​对于好多刚接触 K8S，甚至是接触 K8S 很长时间的同学，K8S 网络模型可以说是个很神秘的东西。那么对于这部分同学，恭喜你发现了本文，只要你花二十分钟的时间，就保证你能轻松掌握 K8S 网络模型原理。

作者 ｜ 中国移动云能力中心 PaaS 产品部 张永曦

​对于好多刚接触 K8S，甚至是接触 K8S 很长时间的同学，K8S 网络模型可以说是个很神秘的东西。那么对于这部分同学，恭喜你发现了本文，只要你花二十分钟的时间，就保证你能轻松掌握 K8S 网络模型原理。

01 知识储备
-------

首先，我们提前热身一下，学一点网络基础知识。

### 1.1 网络命名空间

维基百科的定义是这样的 “Network namespaces virtualize the network stack”，意思就是说 linux network namespace 对 network stack 做了虚拟化。什么是 network stack 呢？网络栈包括了网卡（network interface），回环设备（loopback device），路由表（routing tables）和 iptables 规则。打个比方，当你登录一台 linux 服务器时，你默认用的就是 Host 网络栈。

### 1.2 网桥设备

网桥是在内核中虚拟出来的，可以将主机上真实的物理网卡（如 eth0，eth1），或虚拟的网卡桥接上来。桥接上来的网卡就相当于网桥上的端口，端口收到的数据包都提交给这个虚拟的 “网桥”，让其进行转发。

### 1.3 对设备

Veth Pair 设备被创建出来后，总是以两个虚拟网卡（Veth Peer）的形式成对出现，其中一张 “网卡” 发出的数据包可以直接在出现在对应的 “网卡” 上。Veth Pair 常用作连接不同 Network Namespace 的网线。

### 1.4 VXLAN

VXLAN（Virtual extensible Local Area Network，虚拟扩展局域网），是由 IETF 定义的 NVO3（Network Virtualization over Layer 3）标准技术之一，是对传统 VLAN 协议的一种扩展。VXLAN 的特点是将 L2 的以太帧封装到 UDP 报文（即 L2 over L4）中，并在 L3 网络中传输。

VXLAN 本质上是一种隧道技术，在源网络设备与目的网络设备之间的 IP 网络上，建立一条逻辑隧道，将用户侧报文经过特定的封装后通过这条隧道转发。从用户的角度来看，接入网络的服务器就像是连接到了一个虚拟的二层交换机的不同端口上，可以方便地通信。

### 1.5 BGP（边界网关协议）

边界网关协议（Border Gateway Protocol，缩写：BGP）是互联网上一个核心的去中心化自治路由协议。它通过维护 IP 路由表或 “前缀” 表来实现自治系统（AS）之间的可达性，属于矢量路由协议。BGP 不使用传统的内部网关协议（IGP）的指标，而使用基于路径、网络策略或规则集来决定路由。![](https://s7.51cto.com/oss/202206/07/13e7e9768b5af389362216b6090048f13cf561.gif)

02 什么是微服务的可观测性单机容器网络
--------------------

好了，经过上一章的热身，大家对网络基础知识应该有了大致的了解。那么咱们接下来先尝试探索一下单机容器网络模型，这也是 docker 默认使用的单机容器网络模型。

![](https://s8.51cto.com/oss/202206/07/048982d84f67caffeee142e424e5cc19837b41.png)

图 1 宿主机上不同容器通过网桥进行互通

单机容器网络的示意图如图 1，图中给出了如下几个关键点：

*   每个容器（Container）分别拥有自己的 Network Namespace。
*   容器通过对设备，连接到宿主机的 Host Network Namespace。对设备在容器 Network Namespace 这一端的 “网卡” 是 eth0，eth0 配置的 ip 即容器的 ip。对设备连接 Host Namespace 的那一端挂载到网桥设备 docker0。
*   网桥设备 docker0，挂载着所有容器的对设备的 Host Namespace 这一端。并且，挂载在网桥上的设备，会被降级成网桥上的一个端口，端口的唯一作用就是转发网桥或另一端对设备的数据包。
*   从 Container1 发送到 Container2 的数据包，首先经过 Container1 中的 eth0，到达 docker0 网桥，docker0 网桥经过二层转发，将数据包发送到 Container2 对应的端口（Container2 对设备的 docker0 网桥这一端），这样数据包就被直接送到 Container2 中了。

上述就是单机容器网络的基本原理，在对单机容器网络模型有了初步的认识后，接下来我们进入正题，开始对 K8S 网络模型的探索。![](https://s3.51cto.com/oss/202206/07/617b6c11766a10059cd528026b4e9c8b2290af.gif)

03K8S 网络模型
----------

接触过 K8S 的同学，大致都听说过 Flannel 和 Calico 两种网络模型。但是，这两种网络模型具体是什么样子，工作原理是什么，可能很多同学就比较困惑了。没关系，接下来我们就开始对 k8S 网络模型的探索吧！

### 3.1 Flannel 网络模型

在上一章中，介绍了单机容器网络的原理。那么，要理解容器如何 “跨主机通信” 的原理，一定要从 Flannel 这个项目说起。Flannel 项目是 CoreOS 公司推出的容器网络方案，目前它支持三种后端实现：

*   UDP
*   VXLAN
*   Host-gw

#### 3.1.1 Flannel-UDP

UPD 模式 Flannel 最早实现的一种方式，也是性能最差的，目前已被弃用。但是这种方式也是最直接，最容易理解的方式，所以我们从这种方式开始介绍。

![](https://s9.51cto.com/oss/202206/07/397ed638979fce7b35a543c0f4afd8c5ae5775.png)

图 2 Flannel-UDP 模式跨主机通信示意图

图 2 是 Flannel-UDP 模式的原理图。与单机容器网络相比，这里新增了一个 flannel0 设备和 flanneld 进程。flannel0 设备是一个 TUN 设备，它的作用非常简单，就是在系统内核和用户应用程序之间传包；flanneld 进程的职责，就是封装和解封装。数据包是如何从 Node1 中的 container-1 容器发送到 Node2 的 container-2 容器的呢？

*   数据包从 container-1，来到了网桥 docker0 上，由于数据包的目的地址不属于网桥的网段，所以数据包经由 docker0 网桥，出现在宿主机上。
*   在宿主机的路由表中，去往 100.96.0.0/16 网段的包经由 flannel0 处理。flannel0 收到数据包之后，将数据包送到 flanneld 进程，flanneld 进程会对数据包封装成一个 UDP 数据包，src 和 dst 地址分别为两个容器对应的宿主机的地址。这样，数据包就可以到达 Node2 了。
*   数据包到达 Node2 的 8285 端口，即 Node2 上的 flanneld 进程，会被执行解封装操作，之后数据包被发送到 TUN 设备，即 flannel0 设备。剩下的事情就简单了，数据包经过 docker0 网桥到达 container-2。

#### 3.1.2 Flannel-VXLAN

经过上一小节的介绍，大家对 Flannel-UDP 模式大致了解了吧，那聪明的你们已经猜到为什么 Flannel-UDP 被弃用了吧？没错，因为效率太低了，数据包每次经过 flannel0 设备，都会经过内核态 - 用户态 - 内核态的这一顿折腾。

![](https://s2.51cto.com/oss/202206/07/0585ba6865347984b872243892d2c48bb50152.png)

图 3  TUN 设备示意图

那有没有办法不要这么折腾呢？有，Flannel-VXLAN 方案就解决了这个问题。Flannel-VXLAN 方案用 VXLAN 技术替代了 flannel0 设备，让数据包能够在内核态上实现数据包的封装和解封装。

![](https://s7.51cto.com/oss/202206/07/a52d68638a16ef261110425b73a87b0acdba00.png)

图 4  Flannel-VXLAN 网络模型示意图

Flannel-VXLAN 网络模型的原理如图 4 所示，你会发现，这和 Flannel-UDP 基本上的是一样。事实也的确如此，Flannel-VXLAN 是 Flannel-UDP 的升级版。这里需要交代一下他们之间的不同点。

*   Flannel-UDP 的 TUN 设备 flannel0，升级成了 VXLAN 的 VTEP 设备。数据包的封装和解封装在内核态就能完成。
*   数据包的格式中，增加了 VXLAN Header，这个 Header 的作用和 Flannel-UDP 的数据包中的 dport:8285 的作用是一样的，当数据包来到 Node2 时，操作系统能根据 VXLAN Header，把数据包直接给到 flannel.1 设备。

#### 3.1.3 Flannel-host-gw

此时，聪明的你肯定会说，Flannel-VXLAN 虽然效率提高了，但是还是用到了隧道技术，效率还是会受到影响，能不能不用隧道技术呢？答案是能。接下来我们继续探索 Flannel-host-gw 网络模型，一个基于三层的网络方案。老规矩，上图。

![](https://s4.51cto.com/oss/202206/07/316349535ff5ceab31f32024cc946c5186a319.png)

图 5  Flannel-host-gw 网络模型示意图

图 5 是 Flannel-host-gw 网络模型，相比较之前的两个网络模型，隧道设备确实没有了，取而代之的是一堆路由规则。那，数据包又是怎么从 container1 到 container2 的呢？

*   当数据包从 container1 到了网桥之后，通过 Host 网络栈的路由表，发现去 container2 的路已经指明，经由 eth0，达到 Node2（10.168.0.3/24）即可。
*   当数据包到了 Node2 之后，通过 Host 网络栈的路由表，找到 cni0 网桥，container2 自然也就找到了。

肉眼可见，Flannel-host-gw 的性能确实提高了很多，那为什么还要用 Flannel-VXLAN 呢？原因很明显，Flannel-host-gw 只支持宿主机在二层连通的网络，并且，K8S 的规模不能太大，否则每台机器的路由表就太多了。

### 3.2 Calico 网络模型

经过上一小节的介绍，大家对 Flannel 应该有个大致的了解了。可能有人会问，除了 Flannel，K8S 还有别的网络模型么。当然有了，下面我们开始探索 Calico 网络模型。

#### 3.2.1 Calico（非 IPIP 模式）

实际上 Calico 网络模型的解决方案，几乎和 Flannel-host-gw 是一样的。不同的是 Flannel-host-gw 使用 etcd 来维护主机的路由表，而 Calico 则使用 BGP（边界网关协议）来维护主机的路由表。BGP 协议的定义看着有点高深，换成通俗的说法，大家可以理解为在每个边界网关都会都运行着一个小程序，它们会交换各自的路由信息，将需要的信息更新到自己的路由表里。BGP 这个能力正好可以取代 Flannel-host-gw 利用 Etcd 维护主机上路由表的功能，并且更为强大。

除了 BGP 之外，Calico 另外一个不同之处就在于它不需要维护一个网桥，Calico 网络模型如 6 所示：

![](https://s4.51cto.com/oss/202206/07/c3e2d32137ffe72c32822701b196fd1c69c780.png)

图 6  Calico 网络模型示意图

图 6 是 Calico 网络模型示意图，其中 BGP Client 和 Felix 的作用是和 K8S 集群其他节点交换路由信息，并更新 Host 网络栈的路由信息。

由于没有了网桥设备，每个对设备 Host 网络栈的这一端，需要配置一条路由规则，将目的地址为对应 Container 的数据包转入该对设备。对应的路由如下所示：

```
10.233.1.2 dev cali9c02e56 scope link
```

数据包是如何从 Container1 走到 Container3 的呢？过程基本上和 Flannel-host-gw 无异了。唯一区别就是数据包进出容器，不再依赖网桥，而是直接通过宿主机路由表找到容器的另一端对设备。

#### 3.2.2 Calico（IPIP 模式）

Calico 听着挺强大的，实则和 Flannel-host-gw 一样，只支持宿主机二层联通的情况。假设 Container1 和 Container3 的宿主机在不同的子网，那通过二层网络是无法将数据包传到下一跳的地址的。如图 7 所示，Calico 会在 Node1 创建这样一条路由规则：

```
10.233.2.0/16 via 192.168.2.2 eth0
```

此时问题就出现了，下一跳是 192.168.2.2，和 Node1 不在一个子网里，根本就找不到。

![](https://s4.51cto.com/oss/202206/07/363ee7d453b584febcb190682e861e3f1ebf8b.png)

图 7  Calico（IPIP 模式）网络模型示意图

Calico 的 IPIP 模式解决了上述问题，在每一台宿主机上，都会增加一个 tunl0 设备（IP 隧道设备），并且会对应增加如下一条路由策略：

```
10.233.2.0/16 via 192.168.2.2 tunl0
```

这样一来，Container1 去往 Container3 的数据包就会经过 tunl0 设备的处理，tunl0 设备会在源 IP 报头之外新增一个外部 IP 报头，拿本例来说，这个外部 IP 报头的 src 和 dst 分别为 Node1 和 Node2 的 IP，这样，数据包就伪装成了从 Node1 发到 Node2 的数据包。当数据包到达 Node2 之后，Node2 上的 tunl0 会把外部 IP 报头拿掉，从而拿到原始的 IP 包。

我知道，聪明的你此时肯定会有一个更好的想法，为什么不在 Router1 和 Router2 上也用 BGP 协议的方式，同步容器的 IP 路由信息呢？这样宿主机上不就可以不用 tunl0 设备了么。这个方法确实很好，并且在一些场景也得到了应用。

03CNI 网络插件
----------

最后，我再介绍一下 CNI 网络插件。CNI（Container Network Interface）顾名思义，是 K8S 的网络接口。这个接口的作用就是当 K8S 的 kubelet 创建 Pod 时，dockershim 会预先调用 Docker API 创建并启动 Infra 容器，执行 SetUpPod 的方法。这个方法会为 CNI 网络插件准备参数和环境变量，然后调用 CNI 插件为 Infra 容器配置网络。CNI 网络插件仅需实现 ADD 和 DEL 两种方法，分别对应 Pod 加入 K8S 网络，以及 Pod 移出 K8S 网络。

用大白话再解释一遍，就是当 Pod 创建时，需要对网络进行一些设置，包括前边的提到的创建对设备，把对设备的一端挂载到网桥上，添加 Pod 以及主机的 Network Namespace 的路由规则等，这些操作当然可以由运维人员手动完成（不嫌累的话），CNI 网络插件就是一个脚本，自动对网络进行设置。

Flannel 和 Calico 各自都有专门的 CNI 插件，大家可以去 Github 上研究一下源码，并亲自部署一下试试。这里就不多介绍了，毕竟看再多的资料，都不如自己动手实践一遍理解得深刻。​