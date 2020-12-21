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
DELETE 方法就是让服务器删除请求 URL 所指定的资
## 请求码
- 1xx ：101 告诉浏览器支持 http2，100 表示客户端传的东西太大时分段传直到发送完毕
- 2xx ：请求成功
- 3xx ：301 站点永久迁移浏览器会重定向，302 表示临时迁移，304 表示网站内容未改变（直接刷新当前页会返回 304）
- 4xx ：客户端错误，400 请求参数之类的有错误，404 路径不正确，401 未登录等原因
- 5xx ：服务器错误
## Header
host ：因为虚拟地址的存在，一个 ip 会对应多个域名地址。所以需要用 host 告诉服务器自己来自哪个虚拟地址
Content-Type：数据类型
Content-Length：内容长度，避免二进制内容丢失
Location：重定向地址，当返回 301 时，会把新地址放在 Location 中
User-Agent：用户代理（某某浏览器，某某手机厂商）
Range / Accept-Range ：支持分段下载时，下载的长度
Cache-Control：`no-cahce` 可以缓存但是需要请求服务器是否过期，`no-store` 不能缓存，`max-age` 规定时间内可缓存
## 请求优化
### 减少 http 请求的次数
建立 http 连接需要三次握手，因此每一次请求的建立需要额外的开销。因此将接口融合、小文件合并传输能减少 http 连接的创建次数达到优化请求的目的
### 静态资源使用 CDN
内容分发网络（CDN）是一组分布在多个不同地理位置的 Web 服务器。我们都知道，当服务器离用户越远时，延迟越高。CDN 就是为了解决这一问题，在多个位置部署服务器，让用户离服务器更近，从而缩短请求时间。
### 善用缓存
配置缓存策略，避免每次进入都要拉取新的数据显示
### 压缩文件
Okhttp3 默认支持 Gzip 压缩，只需要服务端添加对应 Header 即可。如压缩需求更苛刻，可以采用 Zlib（Deflate） 进行压缩，可自动配置压缩率，平衡压缩大小和速度

## 编码和字符集
详看 https://www.cnblogs.com/skynet/archive/2011/05/03/2035105.html。
### 字符集
是一个系统支持的所有抽象字符的集合。字符是各种文字和符号的总称，包括各国家文字、标点符号、图形符号、数字等。常见的字符集有`ASCII字符集`、`GB2312字符集`、`BIG5字符集`、`Unicode字符集`。
  #### ASCII 字符集
 ASCII（American Standard Code for Information Interchange，美国信息交换标准代码）是基于拉丁字母的一套电脑编码系统。它主要用于显示现代英语，而其扩展版本EASCII则可以勉强显示其他西欧语言。它是现今最通用的单字节编码系统（但是有被Unicode追上的迹象），并等同于国际标准ISO/IEC 646。
#### GB2312 字符集
 GB2312或GB2312-80是中国国家标准简体中文字符集，全称《信息交换用汉字编码字符集·基本集》，又称GB0，由中国国家标准总局发布
#### BIG5 字符集
 Big5，又称为大五码或五大码，是使用繁体中文（正体中文）社区中最常用的电脑汉字字符集标准，共收录13,060个汉字。
#### Unicode 字符集
 当计算机传到世界各个国家时，为了适合当地语言和字符，设计和实现类似GB232/GBK/GB18030/BIG5的编码方案。这样各搞一套，在本地使用没有问题，一旦出现在网络中，由于不兼容，互相访问就出现了乱码现象。因此就产生了通用的计算机编码字符集 `Unicode` 编码，也称`通用码`、`万国码`
 ##### Unicode 字符集与 UTF 编码的关系
 Unicode 是字符集，而 UTF-32、UTF-16、UTF-8 是编码的实现方式，定义了自然语言文字字符与字符集中的字符的对应方式。
### 编码
我们在屏幕上看到的英文、汉字等字符是二进制数转换之后的结果。通俗的说，按照何种规则将字符存储在计算机中，如'a'用什么表示，称为"编码"；反之，将存储在计算机中的二进制数解析显示出来，称为"解码"，如同密码学中的加密和解密。在解码过程中，如果使用了错误的解码规则，则导致'a'解析成'b'或者乱码。
### 字符编码
字符编码是一套规则，这个规则能将自然语言的一个集合与另一个集合（如数字编码、电脉冲信号）相映射。通常人是使用`文字`来表达信息，而计算机需要使用计算机的文字`二进制数据`来表达信息，因此字符编码即是把符号转化成计算机所能接收的数字代码。
### hash
![enter description here](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/images/1608303873151.png)

## 加密
加密即是把可阅读理解的明文信息经过加密算法变成不可直接阅读的无明显意义的字符。加密可分为可逆加密与不可逆加密。密码的两个关键因素为 `算法`和`秘钥`
### 可逆加密
可逆加密，加密后可通过解密还原出原始数据，可逆加密又分为对称加密和非对称加密。
#### 对称加密（AES、DES）
AES 加密时一个 SK 扩散成多个 SK,轮加密。 DES 秘钥太短容易被破解所以被弃用了

![AES 加、解密过程](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1606294649950.png)
DES 共用一个 SK，可多次迭代使用 DES 加密，如 3DES 加密即是把结果加密再加密。

![3DES 加、解密过程](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1606294592228.png)
#### 非对称加密(RSA)
非对称加密过程有一对公钥和私钥，使用公钥加密的数据使用对应的私钥才能解密。

![RSA 加、解密过程](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1606295510300.png)
为了避免第三方截获公钥使用公钥伪造信息，一般还会使用私钥发送签名数据，接收方使用发送方的公钥进行身份验证
![enter description here](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/images/1608298034176.png)
### 不可逆加密（MD5、SHA1）
不可逆加密在加密后包含的信息不再完整，无法通过算法还原出原始数据。不存在保存秘钥的问题，输入明文后经过加密系统输出不可逆的密文。MD5 和 SHA1 都是基于 Hash  散列实现的。
## HTTPS
Https 即是加了 SSL 层的的 http，保证了数据传输的安全性。数据加密方式可以看下面几组图：
- 1、对称加密
![对称加密](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1606380547317.png)
秘钥泄漏，中间人截获消息
![被第三方截获](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1606380649967.png)
- 2、非对称加密
 ![非对称加密](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1606380708870.png)
 公钥泄漏，中间人截获消息
 ![截获伪造数据](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1606380848570.png)
- 3、HTTPS 引入 CA 机构颁发证书
 https 中的加密，引入了 CA 机构证书。相当于把服务端的公钥交给一个有公信力的第三方，客户端需要时从证书中获取对应服务端的公钥，这样就避免了任意的第三方截获、伪造公钥。
 
 ![HTTPS 加密数据过程](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1606380926951.png) 
  Https 加密过程如下：
 - 1、客户端明文请求 https 地址。
 - 2、服务端返回公钥信息。
 - 3、客户端验证 CA 证书的合法性（机构、版本、时效等）。
 - 4、客户端生成一个随机的对称加密秘钥。
 - 5、使用验证过的证书中的公钥加密生成的随机秘钥并发送给服务端。
 - 6、服务端使用私有钥解密得到客户端的秘钥。
 - 7、服务端使用得到的客户端秘钥加密数据并返回给客户端。
 ![http 加密过程](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1606444215309.png)
## Cookie 和 Authorization
两种方式都是http 中的授权（登录）方式
### Cookie 
主要用在前端与后端交互时的`会话管理`、`用户偏好，主题`、`用户型为分析`等地方，相当于给一个未知的用户打上一个标签。因为是公开传输的，所以他不能拿来做身份认证。
### Authorization


## 从 Retrofit 的原理来看 http

## 从OkHttp 的原理来看 http

# Binder 机制

# Handler 机制
## 概述
Handler 机制中有三个重要的角色 `Handler`、`Looper`、`MessageQueen`、`Message`。整个机制的工作流程为，在线程中 Handler 负责将 Message 发送到 MessageQueen 中， Looper 从 MessageQueen 中不断地去取得消息，并根据消息类型

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
