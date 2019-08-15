---
title: RxJava2 中的异常处理问题
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---
众所周知，RxJava2 中当链式调用中抛出异常时，如果没有对应的 Comsumer 去处理异常，则这个异常会被抛出到虚拟机中去，Android 上的直接表现就是 crash，程序崩溃。

# 异常处理
RxJava2 中 `Observable` 订阅方法 `subscribe()` 接受两种类型的参数

- subscribe()