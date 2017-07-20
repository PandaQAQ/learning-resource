---
title: Picasso 图片下载框架相关 
---

# 杂记
- 默认实现的 Picasso LRU 内存占用为应用所分配的总可用内存的 15%
- 磁盘缓存区大小为总存储量的 2% 最大 50MB 最小 5MB （只有再 API 14 以上或者使用一个单独的提供磁盘缓存的库比如 okhttp）
- 默认三线程加载图片
- 如果需要使用一些自定义的设置通过 Picasso.Builder 来进行
- 如果需要全局改变的话将 Picasso.Builder 得到的自定义对象通过 setSingletonInstance() 设置给全局
- Builder 中设置 RequestTransformer 来修改每次请求的信息，比如说加 header 验证什么的
- 当一个加载目标是弱可及的但是加载请求又没被取消时将会被加入到引用队列。Picasso 有一个 CleanupThread 可手动调用清除队列并取消请求
- 调用 load 的时候会返回一个 RequestCreator 类来创建请求
- 
