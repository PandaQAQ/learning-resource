---
title: Android 事件总线实现——RxBus
---
# 简介
RxBus 是基于 RxJava 实现的事件总线方式，他的强大之处是代码非常的少但是功能却并不简单。最简单的 RxBus 实现仅仅需要三十几行代码，相较之下 EventBus 显得有些臃肿。正因为强大的 RxBus ，Android 开源界的良心企业 Square 公司停止了维护他们的事件总线库 Otto并且在 github 项目主页上为 RxBus 投了一张推荐票：
>Deprecated!
This project is deprecated in favor of RxJava and RxAndroid. These projects permit the same event-driven programming model as Otto, but they’re more capable and offer better control of threading.
If you’re looking for guidance on migrating from Otto to Rx, this post is a good start.
该项目已被RxJava和RxAndroid取代。Rx类项目允许与Otto类似的事件驱动编程模型，而且能力更强，操作线程更方便。
如果你正在寻找从Otto迁移到Rx的指导，这个帖子是个很好的开始。

在引入 RxJava 和 RxAndroid 库的前提下 RxBus 的实现仅仅需要几十行代码的一个类，在越来越多的开发者倾向于使用 RxAndroid + Retrofit 实现网络请求框架的现在，RxBus 成为对应的事件总线实现方式也顺理成章，没必要再为了时间总线单独再引入一个库从而增加应用体积，当然如果项目已经介入 EventBus 作为事件总线库，那就得自己衡量切换成本了。
# 实践
RxJava 与 RxJava2 在操作符和方法上有了一些修改，两个版本的实现方式下面都会讲到.
### 最简实现
  - RxJava 1
``` java
public class RxBus {
    private static volatile RxBus defaultInstance;

    private final Subject<Object, Object> bus;
    // PublishSubject只会把在订阅发生的时间点之后来自原始Observable的数据发射给观察者
    public RxBus() {
        bus = new SerializedSubject<>(PublishSubject.create());
    }
    public static RxBus getDefault() {
        if (defaultInstance == null) {
            synchronized (RxBus.class) {
                if (defaultInstance == null) {
                    defaultInstance = new RxBus();
                }
            }
        }
        return defaultInstance ;
    }
    // 发送一个新的事件
    public void post (Object o) {
        bus.onNext(o);
    }
    // 根据传递的 eventType 类型返回特定类型(eventType)的 被观察者
    public <T> Observable<T> toObservable (Class<T> eventType) {
        return bus.ofType(eventType);
    }
}
```
  - RxJava 2
``` java
public class RxBus {
    private static volatile RxBus defaultInstance;

    private final Subject<Object> bus;

    // PublishSubject只会把在订阅发生的时间点之后来自原始Observable的数据发射给观察者
    private RxBus() {
        bus = PublishSubject.create().toSerialized();
    }

    // 单例RxBus
    public static RxBus getDefault() {
        if (defaultInstance == null) {
            synchronized (RxBus.class) {
                if (defaultInstance == null) {
                    defaultInstance = new RxBus();
                }
            }
        }
        return defaultInstance;
    }

    // 发送一个新的事件，所有订阅此事件的订阅者都会收到
    public void post(Object action) {
        bus.onNext(action);
    }


    // 根据传递的 eventType 类型返回特定类型(eventType)的 被观察者
    public <T> Observable<T> toObservable(Class<T> eventType) {
        return bus.ofType(eventType);
    }
}
```
RxJava1 升级到 RxJava2

### 简单使用
- RxJava 1
``` java
// 数据发送端
// data 为任意数据类型，以 Data 类型代表
RxBus.getDefault().post(data);
```
``` java
	.
	.
	.
// 数据接收端
    mSubscription = RxBus
        .getDefault()
        .toObservable(Data.class)
        .subscribe(new Action1<Data>()){
            @Override
            public void call(Data data){
                    // do something with data ...
            }
        }
	.
	.
	.
// 监听使用离开之后（如关闭监听所在界面）时记得解绑监听，避免引起内存泄漏
if (mSubscription != null && !mSubscription.isUnsubscribed()) {
	mSubscription.unsubscribe();
 }
``` 
- RxJava 2
 ``` java
// 数据发送端
// data 为任意数据类型，以 Data 类型代表
RxBus.getDefault().post(data);
```
``` java
private Disposable mDisposable;
	.
	.
	.
RxBus.getDefault()
        .toObservable(Data.class)
        //使用 subscribeWith 和 subscribe 都可以
        .subscribeWith(new Observer<Data>){
            @Override
            public void onSubscribe(Disposable d) {
                mDisposable = d;
            }

            @Override
            public void onNext(Data data) {
                // do something with data ...
            }

            @Override
            public void onError(Throwable e) {
                // do something with e ...
            }

            @Override
            public void onComplete() {

            }
        };
	.
	.
	.
if (mDisposable != null && !mDisposable.isDisposed()) {
                mDisposable.dispose();
}
```
# 带 Code 封装
通过上面的简单实用会发现一个问题，每发送一个类都需要一个发送对象类，接收的时候也需要传入该对象类。假如存在观察者 A、B、C 都已经注册了对 D.class 消息接收。如果此时只想向 A 发送 D对象，那么就会造成多个观察者都收到这个消息。这也违背了只想传给 A 观察者的初衷。因此我的想法是为每个发送对象在添加一个发送 Code 当 Code 和 class 都满足时观察者才会接收处理消息，这样如果需要通知多个观察者那么观察者注册时使用相同的 Code 就行。
### 代码实现
实现带 Code 的消息发送需要先实现一个 Code 封装类（我命名为 Action）然后在 RxBus 内部将传入的 Code 和对象转换成 Action 封装对象：
``` java
/**
* 封装的消息对象
*/
public class Action 
 // 消息 Code
    public int code;
 // 消息对象
    public Object data;

    public Action(int code, Object data) {
        this.code = code;
        this.data = data;
    }
}
```
  - RxJava 1
```java
public class RxBus {

    private static volatile RxBus defaultInstance;

    private final Subject<Object,Object> bus;

    // PublishSubject只会把在订阅发生的时间点之后来自原始Observable的数据发射给观察者
    private RxBus() {
        bus = new SerializedSubject<>(PublishSubject.create());
    }

    // 单例RxBus
    public static RxBus getDefault() {
        if (defaultInstance == null) {
            synchronized (RxBus.class) {
                if (defaultInstance == null) {
                    defaultInstance = new RxBus();
                }
            }
        }
        return defaultInstance;
    }

    // 发送一个新的事件
    public void postWithCode(int code, Object action) {
        bus.onNext(new Action(code,action));
    }

    // 根据传递的 eventType 类型返回特定类型(eventType)的 被观察者
    public <T> Observable<T> toObservableWithCode(final int code, Class<T> eventType) {
        return bus.ofType(Action.class)
                // 过滤掉非自己 code 对应的接收项
                .filter(new Func1<Action,Boolean>(){
                    @Override
                    public boolean call(Action action){
                        return action.code == code;
                    }
                })
                // 返回传递的数据对象
                .map(new Func1<Action,Object>){
                    @Override
                    public Object call(Action action){
                        return action.data;
                    }
                }
                .cast(eventType);
    }
```
  - RxJava 2
``` java
public class RxBus {

    private static volatile RxBus defaultInstance;

    private final Subject<Object> bus;

    // PublishSubject只会把在订阅发生的时间点之后来自原始Observable的数据发射给观察者
    private RxBus() {
        bus = PublishSubject.create().toSerialized();
    }

    // 单例RxBus
    public static RxBus getDefault() {
        if (defaultInstance == null) {
            synchronized (RxBus.class) {
                if (defaultInstance == null) {
                    defaultInstance = new RxBus();
                }
            }
        }
        return defaultInstance;
    }

    // 发送一个新的事件，所有订阅此事件的订阅者都会收到
    public void post(Object action) {
        bus.onNext(action);
    }

    // 用 code 指定订阅此事件的对应 code 的订阅者
    public void postWithCode(int code, Object action) {
        bus.onNext(new Action(code, action));
    }

    // 根据传递的 eventType 类型返回特定类型(eventType)的 被观察者
    public <T> Observable<T> toObservable(Class<T> eventType) {
        return bus.ofType(eventType);
    }

    // 根据传递的 eventType 类型返回特定类型(eventType)的 被观察者,
    public <T> Observable<T> toObservableWithCode(final int code, Class<T> eventType) {
        return bus.ofType(Action.class)
                .filter(new Predicate<Action>() {
                    @Override
                    public boolean test(Action action) throws Exception {
                        return action.code == code;
                    }
                })
                .map(new Function<Action, Object>() {
                    @Override
                    public Object apply(Action action) throws Exception {
                        return action.data;
                    }
                })
                .cast(eventType);
    }
}
```
与简单使用一样，封装上 RxJava1 与 RxJava2 仅用操作符上的一些差别，实现思路未改变都是在发送时将行为 code 和数据对象 object 封装成 Action 对象，在转换成 Observable 时返回 Action 对象的处理结果，通过 Action 的 code 进行数据过滤。
  ### 使用
除了发送和接收数据对象时会同时接收 code 参数外，带 Code 的 Rxbus 封装使用与不带 Code 完全一致
``` java
RxBus.getDefault().postWithCode(code, data);
```
``` java
  	.
	.
	.
  RxBus.getDefault()
            .toObservableWithCode(RxConstants.BACK_PRESSED_CODE, String.class)
			.subscribeWith(...);
	.
	.
	.
``` 
