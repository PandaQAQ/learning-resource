---
title: Android 笔记
renderNumberedHeading: ture
grammar_cjkRuby: true
---
# Java内存结构
 ![Java内存结构](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1601281207183.png)
 ## 两栈一个堆：
 - **虚拟机栈**
   线程栈帧
 - **本地方法栈**
 - **GC堆**
   
分为新生代、老年代，存放数组、对象等。
# java 虚拟机 GC 机制
## 什么是垃圾
![可达性分析](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1601281610890.png)
**GC Root 对象:**
- Java 虚拟机栈（局部变量表）中的引用的对象
- 方法区中静态引用指向的对象
- 仍处于存活状态中的线程对象
- Native 方法中 JNI 引用的对象
 
 与 GC Root 对象存在直接或间接的引用关系的对象视为非垃圾对象，不存在引用关系的视为垃圾对象，将会在 GC 时被回收
## 什么时候回收
1、堆内存中进行分配时，当内存空间不足以分配给新的对象时将会触发 GC
2、应用层，开发人员调用 System.gc() 主动进行进行一次 GC
## 怎样回收 
### 回收算法
- 标记清除算法
 1、根据可达性，将垃圾对象与存活对象标记出来
 2、将标记出来的垃圾对象清除
 优点：不需要移动对象
 确点：会产生内存碎片、执行 GC 频繁
 
 **图示**
 ![标记清除算法](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1601283576505.png)
- 复制算法
 1、将内存一分为二 （A、B区域）
 2、根据可达性将垃圾对象与存活对象标记出来
 3、将存活对象复制到另一半未使用的内存中
 优点：不需要考虑内存碎片问题，运行高效
 确点：可用内存只有一半，会提高 GC 的频率
 
  **图示**
 ![enter description here](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1601283792712.png)
- 标记-压缩算法
 1、根据可达性标记垃圾对象与存活对象
 2、清除回收垃圾对象
 3、压缩存活对象
 优点：这种方法既避免了碎片的产生，又不需要两块相同的内存空间，因此，其性价比比较高
 确点：需要压缩移动对象，降低了效率
  **图示**
  ![enter description here](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1601283870514.png)
### JVM分代回收策略
JVM 会根据对象生命周期不同，将堆内存分为几块使用
![GC 堆](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1601283042614.png)
1、新对象放在新生代 Eden 区域，GC 一次后存活对象将放在 S0中
2、第二次 GC 会将 S0 中任然活跃的对象放入 S1
3、经过多次 GC 后仍然存活的对象，将被系统视为长生命周期对象，放入到 老年代区域
# ThreadLocal
##  作用
线程隔离
## 线程隔离原理：
  
## 内存泄漏：
- 原因：
ThreadLocal 中存储 threadlocal 与 looper 的 map，ThreadLocalMap 是使用 ThreadLocal 作为 key，looper 作为 value 的。而其中的 key threadlocal 又是一个弱引用，所以当某些时候弱引用被回收，map 中的 key 变成了 null，value 还有值未被回收就导致了内存泄漏。
- 解决办法：
使用完 threadlocal 后调用他的 remove 方法，释放资源
# java 中的各种引用
- **强引用**：直接的引用，可以直接使用对象资源。只要引用村子 GC 永远不会回收，即使导致 OOM
- **软引用**：SoftReference，只有当堆栈使用接近阈值时才会对软引用对象进行回收，GC 回收前 get 返回引用对象，GC 回收后 get 返回 null。一般用于缓存，将导致空指针的尽量不要用
- **弱引用**：只要执行 GC 回收，系统发现了弱引用就会对其进行回收。
- **虚引用**：随时都可能被系统回收，几乎相当于没引用。get 获取总是会返回 null
- **被系统回收优先级**：`虚引用`>`弱引用`>`软引用`>`强引用`
# WebView 白名单验证
```java
private static boolean checkDomain(String inputUrl) throws  URISyntaxException {
    if (!inputUrl.startsWith("http://")&&!inputUrl.startsWith("https://"))
    {
        return false;
    }
    String[] whiteList=new String[]{"site1.com","site2.com"};
    java.net.URI url=new java.net.URI(inputUrl);
    String inputDomain=url.getHost(); //提取host
    for (String whiteDomain:whiteList)
    {
        if (inputDomain.endsWith("."+whiteDomain)) //www.site1.com      app.site2.com
            return true;
    }
    return  false;
}
```
# Apk 打包过程
- **1** aapt 打包资源文件 阶段
- **2** aidl 转 java 文件 阶段
- **3** Java 编译（Compilers）生成.class文件 阶段
- **4** dex（生成dex文件）阶段
- **5** apkbuilder（生成未签名apk）阶段
- **6** Jarsigner（签名）阶段
- **7** zipalign（对齐） 阶段（减少运行时使用内存）

![打包流程图](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1602467733146.png)

# Activity 启动模式
## Standard 标准模式
默认的启动模式，每次都会创建一个新的 Activity 压入栈内
## SingleTop 栈顶复用模式
如果当前任务的栈顶是这个 Activity 则复用调用 `onNewIntent()` 方法，否则与 Standard 模式一样。例如文章、商品的详情页反复打开关联推荐的详情页
## SingelTask 栈内复用模式
会在系统中查找属性值 `affinity` 等于它的属性值 `taskAffinity` 的任务栈，如果存在则在该这个任务栈中启动，否则就在建新任务栈（affinity 值为它的 taskAffinity）启动（FLAG_ACTIVITY_NEW_TASK 同样的情况） 如果在任务栈中已经有该 Activity 的实例，就重用该实例(会调用实例的 onNewIntent() )。重用时，会让该实例回到栈顶，因此在它上面的实例将会被移出栈(即 singleTask 有 clearTop 的效果)。如果栈中不存在该实例，将会创建新的实例放入栈中。
## SingleInstance 单例模式
Activity 放入一个单独的栈内，系统不会往这个栈内放入其他的 Activity。只能在 Manifest 中指定启动模式，无法在代码中通过 Intent 动态指定
## Intent 的 Flags 
- FLAG_ACTIVITY_NEW_TASK 如果 taskAffinity 一样则与标准模式一样新启动一个 Activity,如果不一样则新建一个 task 放该 Activity
- FLAG_ACTIVITY_SINGLE_TOP 与 SingleTop 效果一致
- FLAG_ACTIVITY_CLEAR_TOP 销毁目标 Activity 和它之上的所有 Activity，重新创建目标 Activity + FLAG_ACTIVITY_SINGLE_TOP 效果与 SingleTask 效果一致
- FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS 具有此标记位的 Activity 不会出如今历史 Activity 
## taskAffinity
- 每个 Activity 都有自己所归属的 task。Manifest 中可以通过 `taskAffinity` 指定某个 Activity 所归属的 task，也可以在 `Application` 节点下指定全局的默认 task 的 `taskAffinity`。 
- Android 手机的任务列表就是根据不同 task 弹出的，我们可以根据任务管理器有几个 item 图标，来知道我们开启了几个 task。
- `taskAffinity` 必须与代码中 `Intent.FLAG_ACTIVITY_NEW_TASK` 或者配置 `allowTaskReparenting` 属性组合使用，否则并不会去创建新的 `task`，因为 Acvity 打开时默认的 task 为启动他的 Activity 所在的 task。（Ps:`ARouter` 跳转 Activity 只配置 `taskAffinity`会生效,因为 ARouter 中跳转时默认添加了 `Intent.FLAG_ACTIVITY_NEW_TASK` ）
## allowTaskReparenting
允许 Activity 进行任务栈迁移，迁移的规则是：从一个与该Activity TaskAffinity属性不同的任务栈中迁移到与它TaskAffinity相同的任务栈中。
例如：A 应用中打开 B 应用的 `ActivityB`，由于 ActivityB 被 TaskA 启动，所以其归属的任务栈是 TaskA，当应用 B 启动后，B 应用创建了自己的任务栈，此时 `ActivityB` 将会迁移到 B 的任务栈中。看到的效果即为，A 打开 B 应用的 ActivityB，回到桌面打开 B 应用，此时 B 应用显示的页面为 ActivityB，而不是 B 应用的启动页。