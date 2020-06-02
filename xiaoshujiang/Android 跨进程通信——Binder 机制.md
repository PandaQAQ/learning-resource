---
title: Android 跨进程通信——Binder 机制
---

Android 系统是一个多进程的复杂系统，各式各样的系统服务提供了对应的系统服务。这些系统进程互相协调来完成各项工作，其工作流程简图如下：

![系统服务调用简图](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1588996421213.png)
用户 APP 中的 `Activity 管理`、`窗口管理`、`消息管理`等等都是通过 SDK 中的对应 Manager 来进行的。而这些 Manager 则是通过 IPC 进程通信去调用对应的系统服务处理相应的事件，系统服务再通过跨进程通信通知执行结果。可通过[Android 源码在线预览](https://www.androidos.net.cn/sourcecode)在线搜索查看对应的 `ServiceManager`。

# IPC 通信基本原理
一个进程的虚拟地址空间分为两部分，`用户空间`和`内核空间`。其中用户空间为各个进程独享的，不同的进程间是不能直接互相访问的。内核空间一般用户进程是无法直接访问的，都是由操作系统代为操作，内核空间各个进程是共享数据的。如下图所示：

![进程空间结构图](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/images/1590305354866.png)
在进程的虚拟空间中注意如下几点：
- 进程间：
1、用户空间独立，数据互不共享
2、内核空间共享，所有的进程公用一个内核空间
- 进程内：
用户空间与内核空间数据不能直接进行访问，需要系统代办操作。`copy_from_user（）`将用户空间数据拷贝到内核空间，`copy_to_user（）`将内核空间数据拷贝到用户空间。

## 跨进程交换数据基本流程
如图所示：

![enter description here](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/images/1590306309943.png)

传统的跨进程通信完成一次 CS 进程数据发送需要经过两次数据复制
  - 1、系统调用`copy_from_user（）`将数据从 Client 进程用户空间复制到内核空间（第一次复制）
  - 2、系统调用`copy_to_user（）` 将数据从内核空间复制到 Server 进程（第二次复制）
    
除了数据需要两次复制，这种传统 IPC 方式，因为接收进程的缓存区分配是由接收进程完成的。它并不知道数据具体需要多大的空间，所以只能尽可能大的分配空间，会造成内存的浪费。这在移动设备上的代价还是比较大的，尤其是硬件设备较差的时代。
## Android 中的跨进程
因为上文提到的传统的跨进程通信方式存在的问题，移动端为了更好的性能体验采用了另一种实现方式来达到进程间交换数据的目的，即 Binder 机制。
通过 Binder 方式实现跨进程通信，原理还是没改变，仍然是利用各个进程间共享内核空间，通过内核空间中转数据。binder 机制中，利用 linux 内核动态加载模块的特性在系统内核中加载了 `binder 驱动` 模块来处理跨进程通信。
在 Android 系统中，Binder 驱动和 ServiceManager 属于 Android 系统层，Android 平台已经实现好了，不需要应用开发人员再去编写，应用开发人员只需要编写不同进程的应用层代码即可：

按用户空间与内核空间分，Binder 驱动属于内核空间部分，应用进程属于用户空间，包括 ServiceMnanger 进程也属于用户空间，各个应用进程通过 Binder 驱动进行进程通信。所以 Android 系统中我们做两个应用进程的通信时，Binder 与 ServiceManager 也进行了数据交换。

### 一次拷贝
如果 binder 驱动仅仅是提供了在内核空间中转数据的功能，那相对于传统的方式也并没有性能上的优势。所以显然事情没有这么简单。
