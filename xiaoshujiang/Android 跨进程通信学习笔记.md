---
title: Android 跨进程通信学习笔记
tags: 新建,模板,小书匠

grammar_cjkRuby: true
---
# 为什么需要跨进程通信
Android 系统工作时并不是一个应用就独立完成了所有的工作，而是需要多个应用协作进行工作。而 Android 中一个应用至少包含一个进程。多个应用协调完成工作时就需要在各个进程之间交换数据，这就需要跨进程进行通信。举个栗子，之前公司开发 POS 机应用，上层应用几乎都会去调用 POS 设备的读卡、刷卡、扫码、NFC等等硬件功能。我们开发了一个中间服务应用，系统启动后自动拉起服务，服务中提供了 POS 机所有的功能方法。上层应用通过跨进程绑定服务，间接调用 POS 机的各种功能方法。将与 POS 机硬件功能交互的工作分为上层应用与中间服务应用协作完成。这样将硬件功能服务独立出来维护能保证上层各个应用对底层硬件操作的一致性，也能降低应用开发人员的开发难度，不用再去与 `native` 层交互。
# 跨进程通信方案
Android 中交换数据的方式有哪些呢：
- Bundle
  通过 Bundle 传值的方式，将数据传递给接受 Intent 的目标。期数据发送是单向的，只能发起者将数据传给接受者。
- 文件共享
  硬件上的文件系统是跟进程无关的，可以通过 A 进程将数据序列化写入文件 B 进程将数据读出来再反序列化的方式交换数据，跟上课传纸条的方式类似，文件即是我们传的纸条。
- AIDL
  Android 特有的远程接口定义语言，定义好 AIDL 接口方法后，插件将会自动生成对应的模板代码。其实现方式分为以下几个步骤：
  **Server 端（提供服务方法的进程）**
  1、定义 AIDL 接口并 build 让插件自动生成模板代码
  2、自定义一个 Service，并让`onBind()`方法返回模板 `Stub`的接口实现类
  **Client 端（调用服务方法的进程）**
  1、复制 Server 端的 aidl 文件，并保持文件的路径与 server 端一致
  2、如 Server 端的 Service 有配置签名权限，则 Client 端需要申请对应权限
  3、远程绑定服务，`onServiceConnected` 中调用 `IWebViewAidlInterface.Stub.asInterface(binder)` 获取接口对象
- Messenger
  Messenger 内部也是使用 AIDL 实现的，与 AIDL 方式相比他有以下几点不通：
  1、不通编写 aidl 文件，更便捷
  2、只能一次一个串行交换数据，AIDL 支持多个进程并行处理
  3、不支持 RPC 调用，只能通过 message 传递消息
- ContentProvider
  
- Socket
  
- BroadcastReceiver
# Android 中的 Binder 机制
上面提到的通信方式`bundle`、`messenger`、`AIDL`都离不开 `Binder`机制。可以说 Android 系统中处处都离不开 `Binder`。系统的启动，应用进程的创建，Activity 的跳转管理处处可见 `Binder` 的身影。
Android 进程模型如下：

![进程模型](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1588932905226.png)
由于每个进程控件包含 `用户空间`、`内核空间`，而其中`用户空间`为进程的私有空间，其他进程无法直接访问，因此跨进程通信需要通过一些能够公共的访问空间来交换数据，Binder 则充当了这个中间桥梁的角色。

