---
title: RxJava2 中的异常处理问题
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---
众所周知，RxJava2 中当链式调用中抛出异常时，如果没有对应的 Consumer 去处理异常，则这个异常会被抛出到虚拟机中去，Android 上的直接表现就是 crash，程序崩溃。

# 订阅方式
说异常处理前咱们先来看一下 RxJava2 中 `Observable` 订阅方法 `subscribe()` 的几种订阅方式：

```java 

// 1
subscribe()
// 2
Disposable subscribe(Consumer<? super T> onNext)
// 3
Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError)
// 4
Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError,Action onComplete)
// 5
Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError,Action onComplete, Consumer<? super Disposable> onSubscribe)
// 6
void subscribe(Observer<? super T> observer)

```
无参和以 `Consumer`为参数的几种方法内部都是以默认参数补齐的方式最终调用第 `5` 个方法，而方法 `5` 内部通过 LambdaObserver 将参数包装成 Observer 再调用第 `6` 个方法

``` java
    public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError,
            Action onComplete, Consumer<? super Disposable> onSubscribe) {
        ObjectHelper.requireNonNull(onNext, "onNext is null");
        ObjectHelper.requireNonNull(onError, "onError is null");
        ObjectHelper.requireNonNull(onComplete, "onComplete is null");
        ObjectHelper.requireNonNull(onSubscribe, "onSubscribe is null");

        LambdaObserver<T> ls = new LambdaObserver<T>(onNext, onError, onComplete, onSubscribe);

        subscribe(ls);

        return ls;
    }
```
所以使用 `Consumer` 参数方式和  `Observer` 参数方式进行订阅除了观察回调来源不一样其他没有任何差别。但就是因为这种差别，在异常情况发生时的处理结果上也会产生差别
# 异常处理
我们分别进行一下几种方式模拟异常：

- 1、Observer onNext 中抛出异常
``` kotlin
                apiService.newJsonKeyData()
                    .doOnSubscribe { t -> compositeDisposable.add(t) }
                    .compose(RxScheduler.sync())
                    .subscribe(object : Observer<List<ZooData>> {
                        override fun onComplete() {

                        }

                        override fun onSubscribe(d: Disposable) {

                        }

                        override fun onNext(t: List<ZooData>) {
                            throw RuntimeException("runtime exception")
                        }

                        override fun onError(e: Throwable) {
                            Log.d("error", e.message)
                        }

                    })
```
`结果：不会触发 onError，App 闪退`

- 2、Observer map 操作符中抛出异常
```java
                apiService.newJsonKeyData()
                    .doOnSubscribe { t -> compositeDisposable.add(t) }
                    .map {
                        throw RuntimeException("runtime exception")
                    }
                    .compose(RxScheduler.sync())
                    .subscribe(object : Observer<List<ZooData>> {
                        override fun onComplete() {

                        }

                        override fun onSubscribe(d: Disposable) {

                        }

                        override fun onNext(t: List<ZooData>) {

                        }

                        override fun onError(e: Throwable) {
                            Log.d("error", e.message)
                        }

                    })
```
`结果：会触发 Observer 的 onError，App 未闪退`

- 3、Consumer onNext 中抛出异常
`结果 A：有 errorConsumer 触发 errorConsumer，App 未闪退`
`结果 B：无 errorConsumer，App 闪退`

- 4、先取消订阅再抛出异常
`结果：不会触发错误回调，App 闪退`

那么为什么会出现这些不同情况呢？我们从源码中去一探究竟