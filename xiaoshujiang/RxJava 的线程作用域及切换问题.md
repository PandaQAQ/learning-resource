---
title: RxJava 的线程作用域及切换问题 
tags: 新建,模板,小书匠
renderNumberedHeading: true
grammar_cjkRuby: true
---

1、数据流向是自上向下的流向，订阅是自下向上的订阅
2、数据流产生必定是在所有的订阅之后，这也就是为什么 subscribeOn 不管怎样设置订阅线程，只要一遇到 observeOn 数据流的线程就会被切换到 observeOn 定义的线程上的原因。
3、综上所述，subscribeOn 每次订阅都会切换上级的订阅线程，但是事件回来后只要遇到 observeOn 就会把数据流换到 observeOn 的线程
