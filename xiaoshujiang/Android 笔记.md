---
title: Android 笔记
renderNumberedHeading: false
grammar_cjkRuby: true
---

# ThreadLocal
###  作用
线程隔离
### 线程隔离原理：
  
### 内存泄漏：
- 原因：
ThreadLocal 中存储 threadlocal 与 looper 的 map，ThreadLocalMap 是使用 ThreadLocal 作为 key，looper 作为 value 的。而其中的 key threadlocal 又是一个弱引用，所以当某些时候弱引用被回收，map 中的 key 变成了 null，value 还有值未被回收就导致了内存泄漏。
- 解决办法：
使用完 threadlocal 后调用他的 remove 方法，释放资源
# java 中的各种引用关系
- 强引用：直接的引用，可以直接使用对象资源。只要引用村子 GC 永远不会回收，即使导致 OOM
- 软引用：SoftReference，只有当堆栈使用接近阈值时才会对软引用对象进行回收，GC 回收前 get 返回引用对象，GC 回收后 get 返回 null。一般用于缓存，将导致空指针的尽量不要用
- 弱引用：只要执行 GC 回收，系统发现了弱引用就会对其进行回收。
- 虚引用：随时都可能被系统回收，几乎相当于没引用。get 获取总是会返回 null
- 被系统回收优先级 
`虚引用`>`弱引用`>`软引用`>`强引用`
# java 虚拟机 GC 机制

### 垃圾回收算法
