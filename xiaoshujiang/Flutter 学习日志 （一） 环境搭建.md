---
title: Flutter 学习日志 （一） 环境搭建
grammar_cjkRuby: true
---

本文以 win10 为例，说一说从零开始到跑起来第一个 flutter 应用过程中的坑和需要注意的地方。本文默认已安装好 java 和 android 开发环境，请先配置好相关环境再参照此文配置环境
# 环境搭建
环境搭建的基本步骤官方文档都详细列出来了。以 win10 为例大概说一下步骤：
## 墙外网络：
- 1、从 github 克隆 flutter 工程
`git clone -b beta https://github.com/flutter/flutter.git`
- 2、配置 flutter 系统环境变量
克隆完成后要在命令行中使用 flutter 需要配置 windows 的系统环境变量，在系统环境变量的 `path` 后面加上 `...\flutter\bin` 前面省略部分为第一步中克隆文件夹的路径
- 3、运行 flutter doctor 命令，安装相关依赖
运行 flutter 命令需要在 widows 自动命令行或者 win10 powershell 中运行，git bush 之类的第三方命令行是无效的，第一次运行的时候 flutter 会安装相关的依赖，需要一定时间。安装完成最终会显示下图内容。已安装好的前面会打钩，这里使用 AndroidStudio 作为开发 IDE 说以 Intellij IDEA 那个不用管

![初始化 flutter](http://oddbiem8l.bkt.clouddn.com/flutterdoctor.png)
- 4、AndroidStudio 安装 flutter 插件
打开 AndroidStudio Settings -> Plugins 输入 Flutter 搜索如下图，点击安装后会把 flutter 和 dart 的插件都装上。重启 AndroidStudio 就可以直接创建 Flutter 项目了

![flutter 插件](http://oddbiem8l.bkt.clouddn.com/flutter_plugin.png)
## 墙内网络：
墙内网络环境需要先配置镜像地址，避免下载依赖时反复掉线超时等导致环境初始化失败。
```cmd
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
```
然鹅 windows 命令行压根识别不鸟 export 命令，解决办法是在 windows 环境变量里面配置这两个环境变量。配置在用户环境变量里就行不用配置到系统环境变量。后续的操作跟墙外网络的操作一毛一样
# Run First Flutter App
打开 AndroidStudio 跟以往创建 Android 工程一样，new project 选项多了一个 new Flutter project 选项。选择创建 flutter project 选择 flutter application 。一路点到底，最后一步可选择 kotlin （Android）或者 swift （ios）支持。直接点击运行或者在 AndroidStudo 终端输入 flutter run 都可以在设备上运行 flutter 应用。

![example flutter](http://oddbiem8l.bkt.clouddn.com/S80712-00123214.jpg)
运行阶段对 flutter 工程中的代码修改后直接在命令行按 `r` 可直接热更生效。debug 模式下 flutter 界面的右上角会有一个 debug 的标记，debug 模式下的运行性能会比 release 模式低很多。在已有 Android/ios 项目中将 flutter 以 FlutterView 添加到界面时，debug 模式下界面初始化时会明显黑屏，release 模式则比较流畅。
# 总结
flutter 的环境配置相对来说还是比较简单的，只要处理好初始化时网络环境的问题，按照官方文档的步骤（`windows 镜像配置注意通过环境变量配置`）还是比较顺利的。直接从头到尾创建一个 flutter 应用非常简单，但是纯 flutter 应用并不能满足所有的需求。项目中期如果想集成 flutter 总不可能推掉重来吧，所以在现有的 Android/ios 项目中集成使用 flutter 是很有必要的。这几天了解下来，感觉相关的文档还不是太完善，拼拼凑凑用起来坑也比较多，后面也会以文章的方式记录下来。希望官方后面会加强这方面的支持吧，把在现有项目中集成混合使用的成本降低开发者才更容易接受啊。

