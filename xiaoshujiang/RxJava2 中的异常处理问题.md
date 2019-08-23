---
title: 聊一聊 RxJava2 中的异常及处理方式
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---
众所周知，RxJava2 中当链式调用中抛出异常时，如果没有对应的 Consumer 去处理异常，则这个异常会被抛出到虚拟机中去，Android 上的直接表现就是 crash，程序崩溃。

# 订阅方式
说异常处理前咱们先来看一下 RxJava2 中 `Observable` 订阅方法 `subscribe()` 我们常用的几种订阅方式：

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

- 1、Observer onNext 中抛出异常（切换线程）
``` kotlin
                apiService.newJsonKeyData()
                    .doOnSubscribe { t -> compositeDisposable.add(t) }
                    .compose(RxScheduler.sync()) // 封装的线程切换
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
`结果：不会触发 onError，App 崩溃`

- 2、Observer onNext 中抛出异常（未切换线程）
``` kotlin
               Observable.create<String> {
                        it.onNext("ssss")
                    }
                            .subscribe(object : Observer<String> {
                                override fun onComplete() {

                                }

                                override fun onSubscribe(d: Disposable) {

                                }

                                override fun onNext(t: String) {
                                    Log.d("result::", t)
                                    throw RuntimeException("run llllll")
                                }

                                override fun onError(e: Throwable) {
                                    Log.e("sss", "sss", e)
                                }

                            })
```
`结果：会触发 onError，App 未崩溃`

- 3、Observer map 操作符中抛出异常
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
`结果：会触发 Observer 的 onError，App 未崩溃`

- 4、Consumer onNext 中抛出异常
```kotlin
             apiService.newJsonKeyData()
                    .doOnSubscribe { t -> compositeDisposable.add(t) }
                    .compose(RxScheduler.sync())
                    .subscribe({
                        throw RuntimeException("messsasassssssssssssssssssssssssssssssssssssss")
                    }, {
                        Log.d("Error", it.message)
                    })
```
`结果 A：有 errorConsumer 触发 errorConsumer，App 未崩溃`
```kotlin
    apiService.newJsonKeyData()
                    .doOnSubscribe { t -> compositeDisposable.add(t) }
                    .compose(RxScheduler.sync())
                    .subscribe {
                        throw RuntimeException("messsasassssssssssssssssssssssssssssssssssssss")
                    }
```
`结果 B：无 errorConsumer，App 崩溃`

那么为什么会出现这些不同情况呢？我们从源码中去一探究竟。

## Consumer 订阅方式的崩溃与不崩溃
`subscribe()` 传入 consumer 类型参数最终在 `Observable` 中会将传入的参数转换为 `LambdaObserver` 再调用 `subscribe(lambdaObserver)`进行订阅。展开  `LambdaObserver`：(主要看 onNext 和 onError 方法中的处理)
```java
		.
		.
		.
		    @Override
    public void onNext(T t) {
        if (!isDisposed()) {
            try {
                onNext.accept(t);
            } catch (Throwable e) {
                Exceptions.throwIfFatal(e);
                get().dispose();
                onError(e);
            }
        }
    }

    @Override
    public void onError(Throwable t) {
        if (!isDisposed()) {
            lazySet(DisposableHelper.DISPOSED);
            try {
                onError.accept(t);
            } catch (Throwable e) {
                Exceptions.throwIfFatal(e);
                RxJavaPlugins.onError(new CompositeException(t, e));
            }
        } else {
            RxJavaPlugins.onError(t);
        }
    }
		.
		.
		.

```
`onNext` 中调用了对应 consumer 的 `apply()` 方法，并且进行了 try catch。因此我们在 consumer 中进行的工作抛出异常会被捕获触发 LambdaObserver 的 `onError`。再看 `onError` 中，如果订阅未取消且 errorConsumer 的 `apply()` 执行无异常则能正常走完事件流，否则会调用 `RxJavaPlugins.onError(t)`。看到这里应该就能明白了，当订阅时未传入 errorConsumer时 `Observable` 会指定 `OnErrorMissingConsumer` 为默认的 errorConsumer，发生异常时抛出 `OnErrorNotImplementedException`。

### RxJavaPlugins.onError(t)
上面分析，发现异常最终会流向 RxJavaPlugins.onError(t)。这个方法为 RxJava2 提供的一个全局的静态方法。
```java
    public static void onError(@NonNull Throwable error) {
        Consumer<? super Throwable> f = errorHandler;

        if (error == null) {
            error = new NullPointerException("onError called with null. Null values are generally not allowed in 2.x operators and sources.");
        } else {
            if (!isBug(error)) {
                error = new UndeliverableException(error);
            }
        }

        if (f != null) {
            try {
                f.accept(error);
                return;
            } catch (Throwable e) {
                // Exceptions.throwIfFatal(e); TODO decide
                e.printStackTrace(); // NOPMD
                uncaught(e);
            }
        }

        error.printStackTrace(); // NOPMD
        uncaught(error);
    }
```
查看其源码发现，当 `errorHandler` 不为空时异常将由其消耗掉，为空或者消耗过程产生新的异常则 RxJava 会将异常抛给虚拟机（可能导致程序崩溃）。 `errorHandler`本身是一个 Consumer 对象，我们可以通过如下方式配置他：
```java
    RxJavaPlugins.setErrorHandler(object : Consumer1<Throwable> {
        override fun accept(t: Throwable?) {
            TODO("not implemented") //To change body of created functions use File | Settings | File Templates.
        }

    })
```
## 数据操作符中抛出异常
以 map 操作符为例，map 操作符实际上 RxJava 是将事件流 hook 了另一个新的 Observable `ObservableMap`
```java
    @CheckReturnValue
    @SchedulerSupport(SchedulerSupport.NONE)
    public final <R> Observable<R> map(Function<? super T, ? extends R> mapper) {
        ObjectHelper.requireNonNull(mapper, "mapper is null");
        return RxJavaPlugins.onAssembly(new ObservableMap<T, R>(this, mapper));
    }
```
进入 ObservableMap 类，发现内部订阅了一个内部静态类 `MapObserver`，重点看 `MapObserver`  的 `onNext` 方法
``` java
        public void onNext(T t) {
            if (done) {
                return;
            }

            if (sourceMode != NONE) {
                downstream.onNext(null);
                return;
            }

            U v;

            try {
                v = ObjectHelper.requireNonNull(mapper.apply(t), "The mapper function returned a null value.");
            } catch (Throwable ex) {
                fail(ex);
                return;
            }
            downstream.onNext(v);
        }
```
`onNext` 中 try catch 了 mapper.apply()，这个 apply 执行的就是我们在操作符中实现的 `function` 方法。因此在 map 之类数据变换操作符中产生异常能够自身捕获并发送给最终的 Observer。如果此时的订阅对象中能消耗掉异常则事件流正常走 `onError()` 结束,如果订阅方式为上以节中的 consumer，则崩溃情况为上一节中的分析结果。

## Observer 的 onNext 中抛出异常
上述的方式 `1` 为一次网络请求，里面涉及到线程的切换。方式 `2` 为直接 create 一个 `Observable` 对象，不涉及线程切换，其结果为线程切换后,观察者 Observer 的 onNext() 方法中抛出异常无法触发 onError()，程序崩溃。
### 未切换线程的 Observable.create
查看 `create()` 方法源码，发现内部创建了一个 `ObservableCreate` 对象，在调用订阅时会触发 `subscribeActual()`  方法。在  `subscribeActual()` 中再调用我们 create 时传入的 `ObservableOnSubscribe` 对象的 `subscribe()` 方法来触发事件流。

```java
    @Override
    protected void subscribeActual(Observer<? super T> observer) {
	
		// 对我们的观察者使用 CreateEmitter 进行包装,内部的触发方法是相对应的
        CreateEmitter<T> parent = new CreateEmitter<T>(observer);
        observer.onSubscribe(parent);

        try {
			// source 为 create 时创建的 ObservableOnSubscribe 匿名内部接口实现类
            source.subscribe(parent);
        } catch (Throwable ex) {
            Exceptions.throwIfFatal(ex);
            parent.onError(ex);
        }
    }
```
上述代码中的订阅过程是使用 try catch 今夕包裹的。订阅及订阅触发后发送的事件流都在一个线程，所以能够捕获整个事件流中的异常。（PS : 大家可以尝试下使用  observeOn() 切换事件发送线程。会发现异常不能再捕获，程序崩溃）

### 涉及线程变换时的异常处理
Retrofit 进行网络请求返回的 Observable 对象实质上是 `RxJava2CallAdapter` 中生成的 `BodyObservable`,期内部的 `onNext` 是没有进行异常捕获的。其实这里是否捕获并不是程序崩溃的根本原因，因为进行网络请求，必然是涉及到线程切换的。就算此处 try catch 处理了，也并不能捕获到事件流下游的异常。
```java
    @Override public void onNext(Response<R> response) {
      if (response.isSuccessful()) {
        observer.onNext(response.body());
      } else {
        terminated = true;
        Throwable t = new HttpException(response);
        try {
          observer.onError(t);
        } catch (Throwable inner) {
          Exceptions.throwIfFatal(inner);
          RxJavaPlugins.onError(new CompositeException(t, inner));
        }
      }
    }
```
以我们在最终的 Observer 的 onNext 抛出异常为例，要捕获这次异常那么必须在最终的调用线程中去进行捕获。即 `.observeOn(AndroidSchedulers.mainThread())` 切换过来的 Android 主线程。与其他操作符一样，线程切换时产生了一组新的订阅关系，RxJava 内部会创建一个新的观察对象 `ObservableObserveOn`。
```java
        @Override
        public void onNext(T t) {
            if (done) {
                return;
            }

            if (sourceMode != QueueDisposable.ASYNC) {
                queue.offer(t);
            }
            schedule();
        }
		.
		.
		.
		void schedule() {
            if (getAndIncrement() == 0) {
                worker.schedule(this); // 执行 ObservableObserveOn 的 run 方法
            }
        }
		.
		.
		.
	      @Override
        public void run() {
            if (outputFused) {
                drainFused();
            } else {
                drainNormal();
            }
        }
	
```
而执行任务的 worker 即为对应线程 Scheduler 的对应实现子类所创建的 Worker，以 `AndroidSchedulers.mainThread()` 为例，Scheduler 实现类为 `HandlerScheduler`，其对应 Worker 为 `HandlerWorker`，最终任务交给 `ScheduledRunnable` 来执行。
```java
    private static final class ScheduledRunnable implements Runnable, Disposable {
        private final Handler handler;
        private final Runnable delegate;

        private volatile boolean disposed; // Tracked solely for isDisposed().

        ScheduledRunnable(Handler handler, Runnable delegate) {
            this.handler = handler;
            this.delegate = delegate;
        }

        @Override
        public void run() {
            try {
                delegate.run();
            } catch (Throwable t) {
                RxJavaPlugins.onError(t);
            }
        }

        @Override
        public void dispose() {
            handler.removeCallbacks(this);
            disposed = true;
        }

        @Override
        public boolean isDisposed() {
            return disposed;
        }
    }
```
会发现，run 中 进行了 try catch。但 catch 内消化异常使用的是全局异常处理 `RxJavaPlugins.onError(t);`，而不是某一个观察者的 `onError`。所以在经过切换线程操作符后，观察者 onNext 中抛出的异常，onError 无法捕获。

# 处理方案
既然知道了问题所在，那么处理问题的方案也就十分清晰了。
1、注册全局的异常处理
2、Consumer 作为观察者时，不完全确定没有异常一定要添加异常处理 Consumer
3、Observer 可以创建一个 BaseObaerver 将 onNext 内部进行 try catch 人为的流转到 onError 中，项目中的观察这都使用这个 BaseObserver 的子类。