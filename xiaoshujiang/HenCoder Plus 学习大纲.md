---
title: HenCoder Plus 学习大纲
renderNumberedHeading: true
grammar_cjkRuby: true
---
# HTTP
HTTP 超文本传输协议是位于 TCP/IP 体系结构中的应用层协议，它是万维网数据通信的基础。当我们访问一个网站时，需要通过统一资源定位符（uniform resource locator，URL）来定位服务器并获取资源。
```java
// 默认端口号是 80 端口，通常可以省略
<协议>://<域名>:<端口>/<路径>
```
## 工作机制
- 1、先通过域名系统（Domain Name System，DNS）查询将域名转换为 IP 地址。即将 test.com 转换为 221.239.100.30 这一过程
- 2、通过三次握手（稍后会讲）建立 TCP 连接。
- 3、发起 HTTP 请求。
- 4、服务器接收到 HTTP 请求并响应请求。
- 5、目标服务器往客户端发回 HTTP 响应。
- 6、客户端接收到响应数据做对应的数据展示。

![http 请求过程](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1606097913343.png)

## 数据格式和 REST 
- 1、tcp 包含 32 比特的序号字段和确认号字段。tcp 字节流每一个字节都按顺序编号，确认号是指接收方期望从对方收到的下一字节的序号。
- 2、ACK 标志位，用于标识确认字段中的值是否有效。ACK=1 有效，ACK =0 无效。
- 3、SYN 标志位，用于连接建立，SYN = 1 表示是一个请求建立连接报文。
- 4、FIN 标志位，用于连接拆除，FIN = 1 表示数据已发送完毕，请释放连接。

![tcp 报文首部格式](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1606098033900.png)
### TCP 三次握手和四次挥手

#### TCP 三次握手建立连接
TCP 标准规定，ACK 报文段可以携带数据，但不携带数据就不用消耗序号。
- 1、客户端发送一个不包含应用层数据的报文，首部的 SYN 设置为 1 ，随机选取一个初始报文序号（一般为 0），放在 tcp 报文的序号字段中。（SYN 为 1 时不携带数据，但是会消耗掉一个序号）。
- 2、TCP 报文到达服务端，服务端接收并提取报文并为该 TCP 分配缓存和变量。然后向客户端回复允许连接的 ACK 报文段（包含 SYN=1,ACK=1,确认号置为客户端报文序号+1）
- 3、接收到服务端回复后，客户端也为 TCP 连接分配缓存和变量。然后向服务端发送一个 ACK 报文段，这个报文段中将服务端的报文序号 +1 设置为报文序号。连接建立成功后 SYN 置位 0 ，开始携带数据进行传输。
 ```java
example:
客户端：喂，我要建立连接，能收到吗？（SYN = 1，报文序号 0）
服务端：收到，分配了专人（分配 tcp 缓存和变量）接待，可以进行连接。（SYN =1 ,报文序号 1）
客户端：好的，我这里也是专人负责（分配 tcp 缓存和变量），我们开始通信吧。(SYN = 0 ,报文序号 2)
````
 #### TCP 四次挥手拆除连接
 FIN 报文段即使不携带数据，也要消耗序号。
- 1、客户端发送一个 FIN 置为 1 的报文段。
- 2、服务器回送一个确认报文段。
- 3、服务器发送 FIN 置为 1 的报文段。
- 4、客户端回送一个确认报文段。
```java
example:
客户端：我接收结束了，可以关闭了。
服务端：我收到你的消息了。
服务端：是传输结束了，关闭连接吗？
客户端：是的，可以关闭连接。
````
#### TCP 为什么是四次挥手，而不是三次？
四次挥手是为了让客户端与服务端双方都确认通信完毕，避免报文丢失。
- 1、客户端发送 FIN 报文后，表示我不再发送报文，但是仍然可以接收报文
- 2、服务端收到 FIN 报文后，可能还有正在发送的报文。因此先发送 ACK 报文告知客户端，我知道你想断开连接了。此时客户端收到服务端的应答后就不再向服务端发送断开 FIN 报文了。
- 3、待服务端发送完毕报文后，再发送 FIN 报文通知客户端我刚才发送的报文已经发送完毕了，然后进入  LAST_ACK （超时等待）阶段。
- 4、客户端发送 ACK 报文，双方断开连接。

### REST
一种设计风格：
- 看Url就知道要什么
- 看http method就知道干什么
- 看http status code就知道结果如何
  
## 常见请求方式的区别
### POST & GET
- GET 是幂等的每次获取的结果都是一样的，POST 是非幂等的每次都是一个新的资源
- GET 与 POST 请求 HTTP 协议并未限制其长度，GET 请求长度限制是浏览器限制导致的，tomcat POST 默认长度限制在了 2M，可以通过配置更改。 
### GET & HEAD
- GET 与 HEAD 的请求是安全的幂等（无论请求多少次，资源未改变时返回的结果是一样的）请求。
- GET = HEAD + content，即 HEAD 请求除了不返回文件内容其他响应信息与 GET 一致，因此可以通过 HEAD 请求只获取文件的大小信息，而不下载文件。
### POST
通常用于向服务端提交数据，提交数据可以是表单、raw、json body
### PUT
与 GET 方法从服务器读取文档相反，PUT 方法会向服务器写入文档。PUT 方法的语义就是让服务器用请求的主体部分来创建一个由所请求的 URL 命名的新文档。 如果那个文档已存在，就覆盖它，并不会产生一个新的数据对象。
### DELETE
DELETE 方法就是让服务器删除请求 URL 所指定的资源

## 请求优化
### 减少 http 请求的次数
建立 http 连接需要三次握手，因此每一次请求的建立需要额外的开销。因此将接口融合、小文件合并传输能减少 http 连接的创建次数达到优化请求的目的
### 静态资源使用 CDN
内容分发网络（CDN）是一组分布在多个不同地理位置的 Web 服务器。我们都知道，当服务器离用户越远时，延迟越高。CDN 就是为了解决这一问题，在多个位置部署服务器，让用户离服务器更近，从而缩短请求时间。
### 善用缓存
配置缓存策略，避免每次进入都要拉取新的数据显示
### 压缩文件
Okhttp3 默认支持 Gzip 压缩，只需要服务端添加对应 Header 即可。如压缩需求更苛刻，可以采用 Zlib（Deflate） 进行压缩，可自动配置压缩率，平衡压缩大小和速度
## 编码、加密、Hash、序列化和字符集


## 登录和授权
## TCP/IP 协议族
## https
## 从 Retrofit 的原理来看 http
## 从OkHttp 的原理来看 http

# Kotlin
## kotlin 基础
## kotlin 进阶

# 自定义 View
## View 的绘制流程

## 自定义 View 的绘制

### 图形的位置和尺寸测量
#### 绘制的节本要素
#### 图形的位置、尺寸、角度的计算
#### Path 的方向一级封闭图形的内外判断

### 文字的测量
#### 文字（对齐居中）

### 范围裁切和几何变换
#### Canvas 的范围裁切和几何变换
#### 使用 Canvas 多重变换
#### Matrix 的几何变换
#### 使用 Camera 做三维旋转

### 属性动画和硬件加速
#### ViewPropertyAnimator
#### ObjectAnimator
#### Interpolator
#### PropertyValuesHolder
#### AnimatorSet
#### TypeEvaluator
#### 用字符串做动画
#### 硬件加速
- 是什么
- 为什么
- 有什么弱点
- 离屏缓冲
  
### Bitmap 和 Drawable
#### 概念定义（是什么）
#### 互相转换的方法和本质
#### 自定义 Drawable 的方式和应用场景

### 手写 MaterialEditText 

## 自定义 View 布局
### 布局流程解析
### 自定义布局尺寸定义
### 自定义布局 Layout 自定义
### View 绘制流程源码解析

## 自定义 View 触摸反馈
### 触摸反馈原理全解析
#### onTouchEvent()
#### MotionEvent 的基本 Action
#### 自定义 ViewGroup
### 双向滑动的 ScalableImageView
### 多点触控原理和写法
### ViewGroup 的触摸反馈
### 自定义触摸算法(OnDragListener 和 ViewDragHelper)
### 嵌套滑动
- 关键点
- 嵌套滑动 API
 
 # ConstraintLayout、MotionLayout
 
 # Java 多线程
 ## 多线程和线程同步
 ## 线程间通信
 ## Android 多线程机制
 ## RxJava 的原理全解析
 
 # I/O
 ## Java 的 I/O、NIO、Okio
 含义
 ### Java I/O
 ### NIO 的原理及对比
 ### Okio 的介绍
 
 # Git
 //建议直接看廖雪峰
 ## 核心概念
 ## Feature Branching
 ## 常用指令及其本质
 ## Git Flow
 
 # Gradle
 ## 配置文件解析
 ## Gradle Plugin
 ## Android 构建流程解析
 
 # 插件化、热更新
 
 # Annotation 
 
 # 泛型
 ## 创建
 ## 上界与下界
 ## 类型推断
 ## 使用场景
 ## 重复与嵌套
 ## 类型擦除
 
 # 面试常问源码解析
 ## HashMap 源码解析
 ### 理解 HashMap 的数据结构
 ### HashMap 如何存放元素
 ### HashMap 如何扩容
 ### HashMap 1.7 和 1.8 的区别
 - Java7 数组+链表的方式存储数据
 ## LeakCanary 
 ## BlockCanary
 ## 深入理解 JVM
 
 # 数据结构
 ## 数组
 ## 链表
