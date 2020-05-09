---
title: Android 跨进程通信——Binder 机制
tags: 新建,模板,小书匠
renderNumberedHeading: true
grammar_cjkRuby: true
---

Android 系统是一个多进程的复杂系统，各式各样的系统服务提供了对应的系统服务。这些系统进程互相协调来完成各项工作，其工作流程简图如下：

![系统服务调用简图](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1588996421213.png)
可通过[Android 源码在线预览](https://www.androidos.net.cn/sourcecode)在线搜索查看对应的 `ServiceManager`。
