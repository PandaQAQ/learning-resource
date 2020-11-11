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
# 对象引用
- **强引用**：直接的引用，可以直接使用对象资源。只要引用存在 GC 永远不会回收，即使导致 OOM
- **软引用** `SoftReference`，只有当堆栈使用接近阈值时才会对软引用对象进行回收，GC 回收前 get 返回引用对象，GC 回收后 get 返回 null。一般用于缓存，将导致空指针的尽量不要用
- **弱引用**：`WeakReference`只要执行 GC 回收，系统发现了弱引用就会对其进行回收。
- **虚引用**：`PhantomReference`随时都可能被系统回收，几乎相当于没引用。get 获取总是会返回 null
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
# 线程池
## 通过 Executors 创建线程池
- newSingleThreadExecutor ，创建一个只有一个线程的线程池。使用唯一的工作线程执行任务，因此可以保证进入线程池的任务按顺序执行。
- newFixedThreadPool ，创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
- newScheduledThreadPool，创建一个可定时或周期执行的指定最大线程个数的线程池
- newCachedThreadPool，创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
 ## 线程池创建回收规则
1、创建线程池，设置各个参数， 此时没有一个线程。
2、execute或submit传入一个任务，同时会唤醒休眠的线程。
3、先判断线程数达到`corePoolSize`没有， 如果没有达到就新建一个线程来运行任务。
4、如果线程数达到`corePoolSize`且未达到 `maximumPoolSize`，就开始会把任务加入队列中。
5、如果任务队列也被加满且当前线程数未达到 `maximumPoolSize`，则任然会创建新的线程来处理。
6、如果线程池线程数达到了允许的最大线程数，则会拒绝执行任务。
7、当某个线程运行完任务后， 会再次从队列中获取新的任务运行。
8、如果队列中没有任务，线程会休眠，休眠时间是传入的时间
9、某个线程休眠结束后，会再次从任务队列中获取任务，如果任务队列是空的， 则判断当前存活线程数是否大于核心线程数， 如果大于则这个线程就会死亡。
10、如果小于或者等于最小核心线程， 就会继续休眠。
## 阿里不推荐使用 Excutors 创建线程池原因
1、newCachedThreadPool，创建的最大线程数限制为 `Integer.MAX_VALUE`,队列使用 `SynchronousQueue`,来者不拒，如果任务积压过多有导致 OOM 的风险
```java
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
```
2、newFixedThreadPool，任务队列使用无界的 `LinkedBlockingQueue`，当线程满载时，任务会无限制的加入到队列中，有导致 OOM 的风险
```java
    public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }
```
3、newSingleThreadExecutor，任务队列使用无界的 `LinkedBlockingQueue`，任务会无限制的加入到队列中，有导致 OOM 的风险
```java
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```
4、Executors.newScheduledThreadPool(2)
当任务周期执行时，如果任务时间比周期时间还长，即上一次任务还在执行的时候下一次任务又开始了，刚好执行的任务中代码又有锁，则可能造成线程锁
# Java 中的线程锁
## synchronized 
### 锁范围
- 修饰实例方法：锁对象是当前实例对象，因此只有`同一个实例对象`调用此方法才会产生`互斥`效果，不同实例对象之间不会有互斥效果
- 修饰静态方法：锁对象是当前类的 Class 对象。因此即使在`不同线程`中调用`不同实例`对象，也会有`互斥`效果
- 修饰代码块：锁对象就是后面括号中的对象如下：
```java
class Test{

	private Object object;
	...
	// 对象锁，仅仅对 Test 的同一对象实例互斥，作用域同实例方法
	synchronized(this){
	
	}
	
	// 类锁，对所有的 Test 实例互斥，作用域同静态方法
	synchronized(Test.class){
	
	}
	
	// 对象锁，同一实例互斥。只要拿到 object 的对象锁即可运行
	synchronized(object){
		
	}
	...
}
```
### 锁概念
- 锁自旋：
JVM 对 `synchronized` 的优化，线程阻塞和唤醒 CPU 切换是会消耗性能的。因此当线程去申请锁对象时如果锁对象被其他线程占用，则线程会执行一段无意义的循环等待一段时间而不是直接挂起。如果这段时间内锁被释放则可以直接获得，不用经历挂起唤醒的过程。弊端是，如果锁一直未释放出来，自旋过程也是要消耗性能的。
- 轻量级锁：
 一块同步代码，是由多个线程交替请求。即不存在同一时间多个线程竞争锁的情况。这种状态下锁会保持轻量级锁状态。当多个线程同一时间竞争锁时轻量锁会膨胀成重量级锁。
- 偏向锁
 轻量级锁是在没有锁竞争情况下的锁状态，但是在有些时候锁不仅存在多线程的竞争，而且总是由同一个线程获得。因此为了让线程获得锁的代价更低引入了偏向锁的概念。偏向锁的意思是如果一个线程获得了一个偏向锁，如果在接下来的一段时间中没有其他线程来竞争锁，那么持有偏向锁的线程再次进入或者退出同一个同步代码块，不需要再次进行抢占锁和释放锁的操作。
## ReentrantLock 
