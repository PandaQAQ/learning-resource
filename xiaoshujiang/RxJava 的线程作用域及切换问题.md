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

1、数据流向是自上向下的流向，订阅是自下向上的订阅
2、数据流产生必定是在所有的订阅之后，这也就是为什么 subscribeOn 不管怎样设置订阅线程，只要一遇到 observeOn 数据流的线程就会被切换到 observeOn 定义的线程上的原因。
3、综上所述，subscribeOn 每次订阅都会切换上级的订阅线程，但是事件回来后只要遇到 observeOn 就会把数据流换到 observeOn 的线程
4、subscribeOn 线程作用区间为 自下向上直到遇到下一个 subscribeOn 或者 observeOn 时。这也就是为什么有人说只有自上向下的第一个 subscribeOn 起作用的原因。订阅阶段的线程最终会切换到最后一次指定的订阅线程。

![enter description here](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1585807148894.png)
# 线程调度
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

![RxJava2 线程切换流程](https://raw.githubusercontent.com/PandaQAQ/learning-resource/master/image/1585645839462.png)