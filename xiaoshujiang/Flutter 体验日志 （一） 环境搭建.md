---
title: Flutter 体验日志 （一） 环境搭建
grammar_cjkRuby: true
---

本文以 win10 为例，说一说从零开始到跑起来第一个 flutter 应用过程中的坑和需要注意的地方。本文默认已安装好 java 和 android 开发环境，请先配置好相关环境再参照此文配置环境
# 一、环境搭建
环境搭建的基本步骤官方文档都详细列出来了。以 win10 为例大概说一下步骤：
## 墙外网络：
- 1、从 github 克隆 flutter 工程
`git clone -b beta https://github.com/flutter/flutter.git`
- 2、配置 flutter 系统环境变量
克隆完成后要在命令行中使用 flutter 需要配置 windows 的系统环境变量，在系统环境变量的 `path` 后面加上 `...\flutter\bin` 前面省略部分为第一步中克隆文件夹的路径
- 3、运行 flutter doctor 命令，安装相关依赖
运行 flutter 命令需要在 widows 自动命令行或者 win10 powershell 中运行，git bush 之类的第三方命令行是无效的，第一次运行的时候 flutter 会安装相关的依赖，需要一定时间。安装完成最终会显示下图内容。已安装好的前面会打钩，这里使用 AndroidStudio 作为开发 IDE 说以 Intellij IDEA 那个不用管

![enter description here](http://oddbiem8l.bkt.clouddn.com/flutterdoctor.png)
- 4、AndroidStudio 安装 flutter 插件
打开 AndroidStudio Settings -> Plugins 输入 Flutter 搜索如下图，点击安装后会把 flutter 和 dart 的插件都装上。重启 AndroidStudio 就可以直接创建 Flutter 项目了
## 墙内网络：
墙内网络环境需要先配置镜像地址，避免下载依赖时反复掉线超时等导致环境初始化失败。
