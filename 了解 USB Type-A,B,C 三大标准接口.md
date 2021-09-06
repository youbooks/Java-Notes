# 了解 USB Type-A,B,C 三大标准接口

原文链接：http://sdman.tech/2019/07/22/%E5%88%9D%E6%AD%A5%E4%BA%86%E8%A7%A3USB-Type-A-B-C%E4%B8%89%E5%A4%A7%E6%A0%87%E5%87%86%E6%8E%A5%E5%8F%A3/

## 1. USB 是什么

通用串行总线 (Universal Serial Bus，USB) 是一种新兴的并逐渐取代其他接口标准的数据通信方式，由 Intel、Compaq、Digital、IBM、Microsoft、NEC 及 Northern Telecom 等计算机公司和通信公司于 1995 年联合制定。

## 2. USB Type-A、Type-B 的区别与联系

Type-A 接口的英文名称 “Standard Type-A USB” 意味着它是一个标准的 USB 接口，在它之后的多数 USB 设备都按照此标准执行，比如 **type-A 接口主要作为数据和电源的下行端口**，拥有 type-A 接口的设备从电源方面来讲输入供电设备，从数据来讲输入 Host（主机）。

而 Type-B 接口与 Type-A 接口相反，**不属于供电方或者数据接收方，而是仅作为用电方和数据的上行端口**，多用于外部设备，例如打印机等；

![2021-09-01-NKr5ja](https://image.ldbmcs.com/2021-09-01-NKr5ja.jpg)

Type-A 和 Type-B 的命名区别仅仅是物理形态的不同；查看资料知道，A 和 B 两种接口的电气特性一致，只是形状不同，原因是为了防止使用者把一台设备连接到另一台设备，导致短路（也许是为了区分不同用途的设备？）。

USB 接口分为 USB 1.0 标准、USB 1.1 标准、USB 2.0 标准和 USB 3.0 标准，但因后来 USB1.1 演变为 USB2.0 以及两者本身对应的接口形态不存在差别，所以在这里都笼统的归为 USB 2.0; 下面为 USB 的全家福（除 Type-C 外）：

![2021-09-01-87PJXS](https://image.ldbmcs.com/2021-09-01-87PJXS.jpg)

## 3. USB 2.0 时期

USB2.0 标准当时存在**标准 USB-A、标准 USB-B、Mini USB-A、Mini USB-B、Micro USB-A 和 Micro USB-B 接口**，不难看出当时接口都分为 A、B 端，树形图如下所示：

![2021-09-01-z7aSBc](https://image.ldbmcs.com/2021-09-01-z7aSBc.jpg)

所谓的 Mini 接口和 Micro 接口就是这两个：

1. 分为 A 和 B 的两种 Mini 接口：

    ![2021-09-01-siE76X](https://image.ldbmcs.com/2021-09-01-siE76X.jpg)

2. 分为 A 和 B 的两种 Micro 接口：

    ![2021-09-01-IdtIVj](https://image.ldbmcs.com/2021-09-01-IdtIVj.jpg)

## 4. USB 3.0 时期

USB3.0 标准则只有四种接口，**标准 USB-A、标准 USB-B、Micro USB-A 和 Micro USB-B**。

树形图如图：

![2021-09-01-YoTT3Z](https://image.ldbmcs.com/2021-09-01-YoTT3Z.jpg)

在这里说一下 Mini USB 弃除的原因，首先 Mini USB 防呆性差，厚度比 Micro USB 大，而移动设备追求轻薄的设计不能容纳 Mini USB 接口，过厚的机身会牺牲移动设备的手感，所以便不纳入 USB3.0 标准中。

由于 3.0 标准比 2.0 标准接口多出了 5 个针脚，所以 Micro USB 也迫于生存压力多出了一个口，如下图，也是为了多出的针脚和向下兼容性而做出的妥协

![2021-09-01-yOnUih](https://image.ldbmcs.com/2021-09-01-yOnUih.jpg)

可能有人说为什么标准 USB 接口外形并没有改变，但实际上标准 USB-A 的插片上的针脚由 2.0 的 4 个增加为 3.0 的 9 个，而标准 USB-B 由于先天设计性障碍只能增大自身体积来容纳多出的 5 个针脚：

![2021-09-01-spn44X](https://image.ldbmcs.com/2021-09-01-spn44X.jpg)

## 5. Type-C 接口

说完 Type-A 和 Type-B 接口，是时候了解一下 2014 年前后正式亮相的 Type-C 接口了，相信现在每个人生活都与 Type-C 接口密不可分，除了苹果和安卓的百元机外，所有手机无一例外用上 Type-C 接口，说到这个不能不说全球首款使用 Type-C 接口的乐视超级手机 1，当时三星这个巨头还在使用 Micro USB，可惜乐视开了个好头、却结了个烂尾，现在乐视早已破产，贾跃亭还在美国 ppt 造车呢。

![2021-09-01-BG3TIU](https://image.ldbmcs.com/2021-09-01-BG3TIU.jpg)

虽然 USB 3.1 标准早在 Type-C 接口出来之前早已制定，而且 Type-C 接口也是按照 USB 3.1 标准制定的，但并不是全部 Type-C 接口都是 USB 3.1 标准，有的只支持 USB 3.1 gen1（USB IF 协会改名了的 USB 3.0），有的支持 gen2，有的甚至只支持 USB 2.0，相对应的插头针数也有满针与不满针的区别

![2021-09-01-8DkIzi](https://image.ldbmcs.com/2021-09-01-8DkIzi.jpg)

说了这么多，好像早早脱离了这篇博客原来想写的 —— 研究 Type-C 插头的内部结构以及各针脚的作用，完全是好奇心驱使我研究 C 口结构的，所以，我们就开始吧，首先看看 C 口母座和公头的引脚图：

![2021-09-01-R1bLkl](https://image.ldbmcs.com/2021-09-01-R1bLkl.jpg)

注：母座引脚图

![2021-09-01-w1Oxea](https://image.ldbmcs.com/2021-09-01-w1Oxea.jpg)

注：公头引脚图

TX 代表发送（transport），RX 代表接收（receive）

RX- 和 RX+ 代表超高速接收差分对信号

TX- 和 TX+ 代表超高速发射差分对信号

D+ 和 D- 为通用差分对信号，为了兼容 USB2.0 而设

（何谓差分信号？这两个信号的振幅相同，相位相反即为差分信号。信号接收端比较这两个电压的差值来判断发送端发送的是逻辑 0 还是逻辑 1）

GND 为接地信号

VBus 为总线电源，即 USB 设备的 5V 电压

SBU1 和 SBU2：复用引脚，在不同模式下可作为不同通道使用

CC 引脚：是传统 Type-B 和 Type-A 接口所没有的，也是 Type-C 接口所特有的，并且具有 plug configuration detection，

plug configuration detection 翻译是插头配置检测，能检测 USB 连接、检测正反插、USB 设备间数据与 VBus 的连接建立与管理等

![2021-09-01-IKclyv](https://image.ldbmcs.com/2021-09-01-IKclyv.jpg)

注：C 口正反插的检测

在没有芯片的 cable 上实际上只有一根 cc 线，但在含有芯片的 cable 也不是两根 cc 线，而是一根 cc，一根变成 Vconn 用来给芯片供电。

工作主从机连接如下。DFP 为主，UFP 为从，DRP 可为主也可为从，取决于接什么。DFP 的 CC 脚有上拉电阻 Rp,UFP 有下拉电阻 Rd。未连接时，DFP 的 VBUS 无输出。当 CC 端相连，DFP 的 CC 脚会检测到 UFP 的下拉电阻 Rd，说明连接上，DFP 打开 VBus 开关开始供电。而哪个 CC 脚 (CC1，CC2) 检测到下拉电阻就确定接口插入的方向，顺便切换 RX/TX。

![2021-09-01-C204eo](https://image.ldbmcs.com/2021-09-01-C204eo.jpg)

注：上拉：将不确定的信号通过一个电阻钳位在高电平，电阻同时起限流作用。下拉同理

Type-C 插头分为 Host 主机和 Device 设备，而对应充电过程分为 source 源和 sink，DFP 只能做 source，UFP 只能做 sink，但是 DRP 合二为一，而手机就是 DRP。

最后，正是由于手机这种设备的 Type-C 接口能实现 DRP（Dual Role port），从数据传输来看既能做 Host 也能做 Device，从电源传输来看既能做 source 也能做 sink，未来手机即是主机，为人类生活带来更加轻便的未来

—–短时间内未能完全了解工作主从机工作原理，希望了解的同学加微信交流 —–

## 6. Lightning接口

## 7. 说明

图源：

USB Type-C 学习点滴 https://blog.csdn.net/Fybon/article/details/78115198

技术控必读 从 Type-A 到 Type-C 发展历程 https://blog.csdn.net/liyuzh552200/article/details/83897755

百度经验











