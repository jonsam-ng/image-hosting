> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/LeeQiang8023/article/details/105896531)

一、超网的概念
-------

超网 (Supernetting) 是与子网类似的概念，IP 地址根据子网掩码被分为独立的网络地址和主机地址。超网，也称无类别域间路由选择（CIDR），它是集合多个同类互联网地址的一种方法。

与子网划分（把大网络分成若干小网络）相反，它是把一些小网络组合成一个大网络，就是超网。

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

二、超网合并网段
--------

### 1、合并网段

示例：某企业有一个网段，该网段有 200 台主机，使用 192.168.0.0 255.255.255.0 网段。

后来计算机数量增加到 400 台，为后来增加的 200 台主机使用 192.168.1.0 255.255.255.0 网段，如下图：

[外链图片转存失败, 源站可能有防盗链机制, 建议将图片保存下来直接上传 (img-4srrPQOC-1588429779505)(https://s1.51cto.com/images/blog/202001/05/5d24088e7d02bd4ed72abd42b3a7da8e.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)]

在路由器配置了 192.168.0.1 的 IP 地址接口，再添加 192.168.1.1 地址后，这样 192.168.0.0 和 192.168.1.0 这两个网段内的主机就通过路由器转发来实现通信了。

那么有没有更好的办法，让这两个 C 类网段的计算机认为在一个网段？

这就需要将 192.168.0.0/24 和 192.168.1.0/24 两个 C 类网络合并。  
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS41MWN0by5jb20vaW1hZ2VzL2Jsb2cvMjAxODA1LzExL2Y0N2I4NGZmNWM2NzZjNWFiMWM3YjE2NDc3N2MzMjU1LnBuZw?x-oss-process=image/format,png)

网段合并：子网掩码向前移动 1 位，使得网络部分保持前部分相同。

注：子网掩码往左移 1 位，能够合并 2 个连续的网段，但**不是任何连续的网段都能合并**。

合并网段之后，如下图，这样所有主机相互通信就不再经过路由器转发了。  
[外链图片转存失败, 源站可能有防盗链机制, 建议将图片保存下来直接上传 (img-R4HvDBKU-1588429779508)(https://s1.51cto.com/images/blog/202001/05/aac305e52a42ee6b66c91467e700ff01.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)]

①、合并之后网段为：192.168.0.0/23，IP 分配如下图：  
②、合并之后 IP 地址 192.168.0.255/23 也是可以给计算机使用的，因为主机部分往左增加了一位 0（并不是全 1），如下图：

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS41MWN0by5jb20vaW1hZ2VzL2Jsb2cvMjAxODA1LzExL2I3YTgxMjJlM2Y0Y2I1M2RhNjE1MGQxZTY2NWVkMTVlLnBuZw?x-oss-process=image/format,png)

### 2、不是任何连续的网段都能合并

示例，如下两个连续的网段是不能合并（往前移动 1 位，网络部分不能保持相同）的。  
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS41MWN0by5jb20vaW1hZ2VzL2Jsb2cvMjAxODA1LzExL2MxYTA3Mjk0NjNhZjc0NjczNWY4YmM2ZTlmMzYzYzY5LnBuZw?x-oss-process=image/format,png)

如果非要合并，就要往前移动 2 位，这时候网络部分保持相同，这样一来，等于合并了 4 个网段，如下图：  
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS41MWN0by5jb20vaW1hZ2VzL2Jsb2cvMjAxODA1LzExLzM4NTZiZGQ1ZDkyOWNjN2EzODdjZGRlOWY4OTdkN2VhLnBuZw?x-oss-process=image/format,png)

### 3、哪些连续的网段能够合并

#### （1）判断 2 个网段是否能够合并

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS41MWN0by5jb20vaW1hZ2VzL2Jsb2cvMjAxODA1LzExLzM5ZDMxNmRmMGRjMjMyNjMxMTcwMDZkZWFmYzhjYzM1LnBuZw?x-oss-process=image/format,png)

子网掩码往左移动相应位数后，网络部分保持相同才能合并。  
** 结论：** 判断连续的 2 个网段是否能够合并，只要第一个网络号能被 2 整除，就能够通过左移 1 位子网掩码合并。

#### （2）判断 4 个网段是否能够合并

![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS41MWN0by5jb20vaW1hZ2VzL2Jsb2cvMjAxODA1LzExLzlkNmJmN2JiMzdjMTkzZTZmZjMzNGYzMTVjN2JiYWI0LnBuZw?x-oss-process=image/format,png)

** 结论：** 判断连续的 4 个网段是否能够合并，只要第一个网络号能被 4 整除，就能够通过左移 2 位子网掩码合并。

依次类推，要想判断连续的 8 个网段是否能够合并，只要第一个网络号能被 8 整除，这 8 个连续的网段就能够通过左移 3 位子网掩码合并。

### 4、网段合并的规律

子网掩码左移 1 位能够将能够合并两个网段，左移 2 位，能够合并四个网段，左移 3 位，能够合并 8 个网段。  
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9zMS41MWN0by5jb20vaW1hZ2VzL2Jsb2cvMjAxODA1LzExLzhhMGRiZDYwZjBkNGRiMWE3MDllMzE2YzdiMjYyOGM0LnBuZw?x-oss-process=image/format,png)

### 5、判断一个网段是超网还是子网

①、通过左移子网掩码合并多个网段，右移子网掩码将一个网段划分成多个子网，使得 IP 地址打破了传统的 A 类、B 类、C 类的界限。

②、判断一个网段到底是子网还是超网，就要看该网段是 A 类网络、还是 B 类网络、还是 C 类网络，默认 A 类子网掩码 / 8，B 类子网掩码是 / 16，C 类子网掩码是 / 24。

③、如果该网段的子网掩码比默认子网掩码长，就是子网，如果该网段的子网掩码比默认子网掩码短，则是超网。

.

**© 著作权归作者所有：来自 51CTO 博客作者 MrHuaZi 的原创作品 https://blog.51cto.com/6930123/2111068，如需转载，请与作者联系，否则将追究法律责任**