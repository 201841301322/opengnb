[OpenGNB](https://github.com/gnbdev/opengnb "OpenGNB")是一个开源的去中心化的具有极致内网穿透能力的通过P2P进行三层网络交换的虚拟组网系统。

一些运行GNB的节点处于的网络可能非常复杂，虽然GNB尽最大可能让这些节点通过NAT穿透互连，但是总是难以避免会遇到一些节点在NAT时会失败，这在P2P通讯中是不可避免的。

造成一些节点无法互连的原因还可能有：

节点之间由于没有正确的交换公钥，如果节点采用时钟同步更新密钥但一些节点的时钟没有和其他节点的时钟同步也会导致互连失败。

为了诊断当前GNB与其他节点互连的情况，提供了一个叫`gnb_ctl`的工具来查看当前GNB进程内部的状态。

在介绍`gnb_ctl`之前，要先介绍一下gnb内部的一个机制，gnb在设计之初就考虑到如何将gnb进程内部大部分的核心的状态暴露出来，这样用户不需要通过观察gnb的日志就能了解到gnb进程的工作状况，例如当前节点与哪些节点是成功建立了连接，也可以根据这些信息判断某些节点没有互连成功的原因。

这个机制就是共享内存，gnb内部几乎所有的重要的数据结构所使用的内存都不是从堆上获得，而是通过mmap一块内存与 `gnb_ctl` `gnb_es` 共享，通常情况是gnb进行负责更新这块共享内存，`gnb_ctl` `gnb_es` 多数时候是读取这块内存。

如果启动gnb时不通过参数指定，这块mmap内存对应的文件 gnb.map 将在节点配置目录中创建。

Unix系的mmap系统调用几乎都是一样，除了个别平台特性相关的参数，唯独Windows创建共享内存的API是独有的，因此gnb特地对Unix系和Windows的共享内存做了一个封装，具体代码在 `gnb_mmap.h gnb_mmap.c`，现在这部分代码已经公开。

由于启动 gnb 通常是root用户，因此通过`gnb_ctl`打开这块共享内存也需要用root用户。

当gnb启动后，执行下面的命令就能看到gnb内部的状态，

`./gnb_ctl -b ../../conf/1001/gnb.map -c`

输出的信息中会列出所有节点的连通状态，如果一个节点的信息中有
`ipv6 REACHABL`
或在
`ipv4 REACHABL`
就说明这个节点与当前节点当然成功连通的。

用户可以通过这些信息排查节点无法连通的原因，其中包括当前节点与其他节点通讯时使用的密钥，这些信息涉及到通讯的安全不应当透露给无关的他人。

当确实遇到无法连通的节点，这样可能就需要通过`forward`节点去为这些节点提供数据中转服务，这样这些节点就从点对点网络转变为中心化网络。

如名字的含义，`gnb_ctl`将来还可以做更多的事情。

需要了解更多细节可以执行`gnb_ctl -h` 了解。
