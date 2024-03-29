---
title: RxJava 观察绑定和事件发送流程及其中的线程切换分析 
grammar_cjkRuby: true
---
**本文的所有分析都是基于 RxJava2 进行的。以下的 RxJava 指 RxJava2**
阅读本文你将会知道：
- RxJava 的观察绑定和事件发送过程
- RxJava 观察绑定和事件发送过程中的线程切换

从 RxJava1.0 到 RxJava2.0，在项目开发中已经使用了很长时间这个库了。链式调用，丝滑的线程切换很香，但是如果没弄清楚其中的奥妙很容易掉进线程调度的坑里。这篇文章我们就来对 RxJava 的订阅过程、时间发送过程、线程调度进行分析
# 订阅和事件流
先说结论
- 按着代码书写顺序，事件自上向下发送
- 订阅从 `subscribe()` 开始自下向上订阅，这也是整个事件流的起点，当订阅开始整个操作才会生效执行
- 订阅完成后才会发送事件
### 图解
**为了更便于理解订阅的流转方向,我将Observable调用 `subscribe()` 订阅描述为了 Observer `beSubscribed()`**

![订阅及数据发送](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1585807148894.png)
## 源码分析
### Observabe 创建过程
此过程对应图中`黑色箭头`部分，以操作符中的`map()`操作为例：
>```java
>    @CheckReturnValue
>    @SchedulerSupport(SchedulerSupport.NONE)
>    public final <R> Observable<R> map(Function<? super T, ? extends R> mapper) {
>        ObjectHelper.requireNonNull(mapper, "mapper is null");
>        return RxJavaPlugins.onAssembly(new ObservableMap<T, R>(this, mapper));
>    }
>```
调用`map`操作符时，RxJavaPliguns 会注册一个新的 `ObservableMap` 对象，查看其它操作符会发现都有对应的 `Observable` 对象产生。同时，上游的 `Observabe`会作为 `source` 参数传入赋值给这个新的 `Observable` 的 `source`属性。层层向下，可以对这个新生成的 `Observable`又可以继续使用操作符。
### 订阅过程：
当调用最后一个  `Observable` 的 `subscribe（）` 方法时，即开始订阅过程。此过程对应图中`红色箭头`部分
>```java
>    @SchedulerSupport(SchedulerSupport.NONE)
>    @Override
>    public final void subscribe(Observer<? super T> observer) {
>        ObjectHelper.requireNonNull(observer, "observer is null");
>        try {
>            observer = RxJavaPlugins.onSubscribe(this, observer);
>
>            ObjectHelper.requireNonNull(observer, "The RxJavaPlugins.onSubscribe hook returned a null Observer. Please change the handler provided to RxJavaPlugins.setOnObservableSubscribe for invalid null returns. Further reading: https://github.com/ReactiveX/RxJava/wiki/Plugins");
>
>            subscribeActual(observer);
>        } catch (NullPointerException e) { // NOPMD
>            throw e;
>        } catch (Throwable e) {
>            Exceptions.throwIfFatal(e);
>            // can't call onError because no way to know if a Disposable has been set or not
>            // can't call onSubscribe because the call might have set a Subscription already
>            RxJavaPlugins.onError(e);
>
>            NullPointerException npe = new NullPointerException("Actually not, but can't throw other exceptions due to RS");
>            npe.initCause(e);
>            throw npe;
>        }
>    }
>```
在调用`subscribe(Observer)` 时实际上会去调用各个 `Observable `实现子类中的 `subscribeActual()` 方法:
>```java
>    @Override
>    public void subscribeActual(Observer<? super U> t) {
>        source.subscribe(new MapObserver<T, U>(t, function));
>    }
>```
而在这个`subscribeActual()` 方法也很简单，调用了 `source` 去订阅一个新生成的 `Observer` 对象，同时这个新的`MapObserver`会将调用`subscribe()`时传入的 `observer`,赋值给`downstream`属性。这样每一级订阅都会将上级的 `Observable`、本级生成的  `Observer`、订阅下级传入的`Observer`联系起来，直到达到 Observable 最初创建的地方整个订阅过程结束。
### 事件发送过程：
此过程对应图中`绿色箭头`部分Observable 事件起点创建有很多中操作符，他们都会创建出最初发送的事件/数据，以 `ObservableCreate`为例：
 >```java
>    @Override
>    protected void subscribeActual(Observer<? super T> observer) {
>        CreateEmitter<T> parent = new CreateEmitter<T>(observer);
>        observer.onSubscribe(parent);
>
>        try {
>            source.subscribe(parent);
>        } catch (Throwable ex) {
>            Exceptions.throwIfFatal(ex);
>            parent.onError(ex);
>        }
>    }
>```
订阅时会调用`source.subscrebe(parent)`,而这个`source` 又是从哪儿来的呢？
>```java
>    public ObservableCreate(ObservableOnSubscribe<T> source) {
>        this.source = source;
>    }
>```

>```java
>    Observable.create(object : ObservableOnSubscribe<String> {
>           override fun subscribe(emitter: ObservableEmitter<String>) {
>                emitter.onNext("data")
>           }
>
>    })
>```
从代码中我们可以看出，这个 source 即为我们创建时传入的 `ObservableOnSubscribe`,因此` emitter.onNext("data")`即是事件发送的起点。我们再继续看`emitter`的 `onNext()` 做了什么：
```java
        @Override
        public void onNext(T t) {
            if (t == null) {
                onError(new NullPointerException("onNext called with null. Null values are generally not allowed in 2.x operators and sources."));
                return;
            }
            if (!isDisposed()) {
                observer.onNext(t);
            }
        }
```
源码中现实调用了`observer.onNext()`,而这个`observer` 则是前面订阅过程中 `source.subscribe(new MapObserver<T, U>(t, function))` 传入的那个 `observer`，从而将事件发送到了下一级，下一级的 Observer 同样在 `onNext()` 将事件发送到更下一级，一直到最终我们 `subscribe()`时传入的那个`Observer` 实例完毕。

# 线程调度
事件订阅发送流程通过上面的文章基本已经能够摸清了，我们接下来关注另一个重点 `线程调度`问题。
## 调度方式
RxJava 中线程变换通过 `subscribeOn()`和 `observeOn()`两个操作来进行。其中 `subscribeOn()`改变的是订阅线程的执行线程，即事件发生的线程。`observeOn()`改变的是事件结果观察者回调所在线程，即 `onNext()`方法所在的线程。

![举个栗子](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1585809124336.png)
使用 RxJava + Retrofit 进行网络请求时，用 RxJava 管理网络请求过程的线程切换。`subscribeOn()`指定的是网络请求的线程，`observeOn()`指定的是网络请求后事件流的执行线程。
## 源码分析
前面说过，每次操作符的使用，RxJava 都会生成一个对应的新的 `Observable`对象。`observeOn()`与 `subscribeOn()`也不例外。线程调度的核心逻辑都在 `ObservableSubscribeOn` 与 `ObservableObserveOn`两个类中
### subscribeOn()过程
>```java
>   @CheckReturnValue
>   @SchedulerSupport(SchedulerSupport.CUSTOM)
>   public final Observable<T> subscribeOn(Scheduler scheduler) {
>       ObjectHelper.requireNonNull(scheduler, "scheduler is null");
>        return RxJavaPlugins.onAssembly(new ObservableSubscribeOn<T>(this, scheduler));
>    }
>````
调用 `subscribeOn()` 时会产生一个新的`ObservableSubscribeOn`并把当前这个`Observable` 和传入的 `Scheduler`作为参数传入。前面分析过当最终调用 `subscribe()`时会引起整个观察链的 `Observable` 自下而上调用 `subscribe()`，而这个`subscribe()`方法中实际为调用抽象类 `Observable`的各个实现子类的 `subscribeActual()`方法 。
>```java
>    @Override
>    public void subscribeActual(final Observer<? super T> observer) {
>        final SubscribeOnObserver<T> parent = new SubscribeOnObserver<T>(observer);
>
>        observer.onSubscribe(parent);
>
>        parent.setDisposable(scheduler.scheduleDirect(new SubscribeTask(parent)));
>    }
>````
主要看这句 `scheduler.scheduleDirect(new SubscribeTask(parent));`,`SubscribeTask` 前面内容已经分析过，就是调用上级 `Observable` 来订阅生成的这个 `SubscribeOnObserver`。
>```java
>    @NonNull
>    public Disposable scheduleDirect(@NonNull Runnable run, long delay, @NonNull TimeUnit unit) {
>        final Worker w = createWorker();
>
>        final Runnable decoratedRun = RxJavaPlugins.onSchedule(run);
>
>        DisposeTask task = new DisposeTask(decoratedRun, w);
>
>        w.schedule(task, delay, unit);
>
>        return task;
>    }
>```
`scheduleDirect` 方法，会使用传入的 `scheduler` 在指定的线程创建一个 `Worker` 对象来执行`SubscribeTask`,从而达到了切换订阅线程的目的。所以多个`subscribeOn()`叠加时，最终线程还是会回到最后执行的（代码第一次出现的）`subscribeOn()` 指定的线程。
### observeOn()过程
调用 `observeOn(Scheduler)` 方法，会调用内部的同名方法生成一个新的 `ObservableObserveOn`对象，并把当前这个`Observable` 和传入的 `Scheduler`作为参数传入。订阅过程与`ObservableSubscribeOn`不一样，会直接在当前线程调用上级`Observable`订阅自己，，我们主要看`ObservableObserveOn`的`ObserveOnObserver`是如何调度结果数据发送的线程的。
>```java
>        @Override
>        public void onNext(T t) {
>            if (done) {
>                return;
>            }
>
>            if (sourceMode != QueueDisposable.ASYNC) {
>                queue.offer(t);
>            }
>            schedule();
>        }
>
>        void schedule() {
>            if (getAndIncrement() == 0) {
>                worker.schedule(this);
>            }
>        }
>```
从源码中可以发现，最终会使用 `worker` 去向下游发送事件。这个 `worker`就是我们`observeOn()` 方法中指定的线程创建的 worker。从而达到切换线程的目的，由于事件又是自上而下的，所以每次切换都能在下游事件中感受到线程的变化。
 ## 日志分析
把`subscribeOn()`和 `observeOn()`放一起来说不太容易说明白其中的线程变换，我先看看单独使用其中的一个操作符的时候，导致的线程变化。
### 仅调用 subscribeOn() 调度线程

```kotlin
Observable.just("Data")
                .map {
                    Log.d("Map 1", Thread.currentThread().name)
                    return@map it
                }
                .subscribeOn(Schedulers.io()) 
                .doOnSubscribe {
                    Log.d("doOnSubscribe 1 ", Thread.currentThread().name)
                }
                .map {
                    Log.d("Map 2 ", Thread.currentThread().name)
                    return@map it
                }
                .subscribeOn(Schedulers.newThread())
                .doOnSubscribe {
                    Log.d("doOnSubscribe 2 ", Thread.currentThread().name)
                }
                .map {
                    Log.d("Map 3 ", Thread.currentThread().name)
                    return@map it
                }
                .subscribe(object : Observer<String> {
                    override fun onComplete() {

                    }

                    override fun onSubscribe(d: Disposable) {
                        Log.d("onSubscribe", Thread.currentThread().name)
                    }

                    override fun onNext(t: String) {
                        Log.d("onNext", Thread.currentThread().name)
                    }

                    override fun onError(e: Throwable) {
                        e.printStackTrace()
                    }

                })
```
执行结果：

![日志打印](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1585811094493.png)
从日志可以看出：
- 1、订阅是自下向上的（onSubscribe -->doOnSubscribe 2 -->doOnsubscribe 1）
- 2、自下向上看，每次调用 `subscribeOn` 订阅线程将会发生改变，直到下次调用 `subscribeOn`
- 3、事件是自上向下传递的（Map 1 --> Map 2 --> Map 3 --> onNext）,且所在线程为最后一次线程切换后所在的线程 `RxCachedThreadScheduler-1`
### 仅调用 subscribeOn() 调度线程

```java
        Observable.just("Data")
                .map {
                    Log.d("Map 1", Thread.currentThread().name)
                    return@map it
                }
//                .doOnSubscribe {
//                    Log.d("doOnSubscribe 1 ", Thread.currentThread().name)
//                }
//                .subscribeOn(Schedulers.io())
                .observeOn(Schedulers.newThread())
                .map {
                    Log.d("Map 2 ", Thread.currentThread().name)
                    return@map it
                }
//                .doOnSubscribe {
//                    Log.d("doOnSubscribe 2 ", Thread.currentThread().name)
//                }
//                .subscribeOn(Schedulers.newThread())
                .observeOn(Schedulers.newThread())
                .map {
                    Log.d("Map 3 ", Thread.currentThread().name)
                    return@map it
                }
                .subscribe(object : Observer<String> {
                    override fun onComplete() {

                    }

                    override fun onSubscribe(d: Disposable) {
                        Log.d("onSubscribe", Thread.currentThread().name)
                    }

                    override fun onNext(t: String) {
                        Log.d("onNext", Thread.currentThread().name)
                    }

                    override fun onError(e: Throwable) {
                        e.printStackTrace()
                    }

                })
```
执行结果：

![日志打印](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1585812936176.png)
从日志可以看出：
- 1、事件发送是正常的自上向下（Map 1 --> Map 2 --> Map 3 --> onNex）
- 2、自上向下，每次调用 `observeOn` 观察结果回调线程都将切换一次（main -->RxNewThreadScheduler-1 -->RxNewThreadScheduler-2）

### 混合使用调度线程
我们把上述代码中注释部分都打开，得到的日志如下：

![日志打印](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1585813401310.png)
**通过上面的三次日志打印我们可以看出：**

订阅链的日志自下而上打印完毕后，再自上而下打印观察结果。`subscribeOn` 会切换线程，并不是像有的文章所说只有第一次指定线程(即自下而上的最后一次)有效。第一次有效只是我们的错觉，因为订阅是自下而上的，不管前面的线程怎样切换追踪都会切换到 `subscribeOn`第一次指定线程(即自下而上的最后一次)。我们在回调结果中未进行线程切换操作时，只能感知到这一次线程切换 (Map1 与 doOnSubscribe 1 所在线程一致)。`observeOn`的每次指定线程都会让事件流切换到对应的线程中去。完整的事件订阅和发送流程如下图所示，从我们调用 `subscribe()`将观察者和观察对象关联起来开始，`subscribe()` 中传入的 Observer 的 `onNext` 或 `onError`结束，形成了一个逆时针的 `n` 形的链条。右边部分的观察链中，每次 `subscribeOn` 都会切换观察线程。左边部分的事件发送链，会从观察链的最后一次指定的线程开始发送事件，每次调用 `observeOn`都会指定新的事件发送线程。
## 图解
参照上面的源码和日志分析，再结合本图相信大家会对 RxJava 的现场调度有一个更立体的认识

![RxJava2 线程切换流程](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1585645839462.png)