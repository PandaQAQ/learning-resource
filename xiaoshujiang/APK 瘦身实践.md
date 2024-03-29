---
title: APK 瘦身实践
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---
撸完代码打包发现应用没太多复杂功能的应用打包出的 apk 安装包大小居然达到了感人的 62M 之大！显然这个这个大小是不能接受的。那么要瘦身首先要知道胖在哪儿，对症下药才能出奇效。
# 分析“肥胖原因”
Android Studio 自带的 Apk 分析工具就能处理这个问题，`Build` ->`AnalyzeAPK` 选中待分析的 APK 即可得到安装包中各个大类别的体积大小：

![apk](http://oddbiem8l.bkt.clouddn.com/APKmax.png)
从上图可以看出总共 62M 大小的安装包 lib 目录占了 43M 之大。这些都是应用中使用到的针对不同类型 CPU 的 .so 依赖库文件，如果能想办法减少这部分的体积，APK 大小应该会有明显的缩小。这么多的 "ABIs" 如果无特殊需要是不用全部支持的，去掉其中的部分则可以压缩安装包的体积。

# jniLibs 的取舍
| ABI 类型   |   备注   |         |
| ----------- | ------------------------ | ------------------------------------------ |
| arm64-v8a   | 第8代、64位ARM处理器     | 很少设备，三星 Galaxy S6是其中之一         |
| armeabi-v7a | 第7代及以上的 ARM 处理器 | 现在市面上的主流安卓机型 CPU 架构          |
| armeabi     | 第5代、第6代的ARM处理器  | 早期设备，v7a 及 v8a 能兼容 armeabi 指令集 |
| x86         |                          | 平板及模拟器，少部分华硕的机型                               |
| x86_64      |                          | 64 位平板                                  |

上表中可以看出，如果不是做的平板应用，也不需要兼容平板及模拟器运行。那么 `X86` 的两个包是可以去掉的。剩下的三个 `armeabi` 的依赖库则可以根据需求做取舍。需要注意的是：
`如果这三个包都存在时需要每个包类型中都有对应的 .so 文件。因为向下兼容的原因，高架构版本 CPU 的手机在加载依赖库时如果无高版本 lib 目录则会以低版本兼容方式运行，如果有高版本目录但是资源不完整则会报错。比如在 arm64-v8a 类型手机上运行应用时，如果 armeabi-v7a 目录中有 a.so、b.so， arm64-v8a 目录中只有 a.so 在运行时则会出现 b.so 找不到的问题。如果删除掉 arm64-v8a 的目录则应用会以低版本兼容模式运行，不会产生问题，同理 armeabi 也一样。`
从上面的内容来看，保留哪些类型的依赖库也很明朗了：
 
 - 需要保留 64 位的优化特性则保留 64位 包
 - 需要兼容 5、6 代 arm 机型的保留 armeabi
 - 其他情况使用 armeabi-v7 包即可
  
如果应用是专用设备分发安装，不是应用市场自由下载。使用多渠道打包每种类型打一个包给对应平台使用才是最佳解决方式。
### 最终选择
摸着石头过河，咱也可以摸着大佬们过坑。论机型覆盖，国内的主流应用`微信`、`支付宝`应该是妥妥的第一梯队。那我们来看一下大佬们保留了哪些依赖库：
- 支付宝

![enter description here](http://oddbiem8l.bkt.clouddn.com/AliPay.png)
- 微信

![enter description here](http://oddbiem8l.bkt.clouddn.com/WX.png)
通过 AndroidStudio 分析发现微信和支付宝都是单个 lib `armeabi` 包。因为我的应用也没有特别需求的骚操作，因此也跟两位大佬一样最终只保留了`armeabi`包，此外因为使用了腾讯 X5 浏览服务，根据他的官方文档，要在 64 位手机上正常运行是需要将应用以 32 0位模式运行的，即只能使用 32 位包。
# 资源优化
经过前面的 lib 过滤，在只使用 `armeabi` 包的情况下打包出来的 APK 大小已经从原来的 62M 缩减到了23M 的大小

![enter description here](http://oddbiem8l.bkt.clouddn.com/fixlib.png)
优化 jni 依赖包后 APK 的大小大幅缩减了，现在安装包最大的部分变成了 res 目录，点进去看发现 drawable 目录下居然有张超过 1M 的大图，后来找 UI 重新要了张小一些的图，从而将 APK 大小进一步缩小了。除了给的资源图片太大会导致没必要的占用 APK 体积外，将大量的未使用图标资源打包进 APK 也会增加体积。解决这个问题的办法是在 gradle 中配置 `shrinkResources=true`，在打包时剔除未使用资源。混淆打包也能在一定程度上压缩安装包体积，所以不要因为加固所以就不配置混淆，他还能帮你减小安装包体积呢。
- 工具推荐
大的 png 图片，本着自己动手丰衣足食的原则。可以通过 [TinyPNG](https://tinypng.com/)这个网站将图片进行高保真压缩，传上去的 1.9M 的 png 图片帮我压缩到了 680 kb，这个压缩效果还是比较明显的。当然能找到 UI 统一处理一下还是交给 UI 处理方便一些（手动滑稽）
# 总结一哈
APK 瘦身基本就干下面几件事
- 根据自己的需求确定使用哪些版本的 JNI 库。不要全都打进去了，三方是全平台都提供了但是你不一定要全平台都用啊
- 检查资源图片中是否有太大图片，试着处理一下再放进去。资源包括 res 和 assets 目录
- 配置打包时过滤未使用文件 `shrinkResources=true`
- 一定要混淆代码打包
- 图片换成 WEBP 格式，不用 PNG 格式（这个不是我操作的，同事试了效果还不错）
