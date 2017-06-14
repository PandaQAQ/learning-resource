通过上面的简单实用会发现一个问题，每发送一个类都需要一个发送对象类，接收的时候也需要传入该对象类。假如存在观察者 A、B、C 都已经注册了对 D.class 消息接收。如果此时只想向 A 发送 D对象，那么就会造成多个观察者都收到这个消息。这也违背了只想传给 A 观察者的初衷。因此我的想法是为每个发送对象在添加一个发送 Code 当 Code 和 class 都满足时观察者才会接收处理消息，这样如果需要通知多个观察者那么观察者注册时使用相同的 Code 就行。
### 代码实现
实现带 Code 的消息发送需要先实现一个 Code 封装类（我命名为 Action）然后在 RxBus 内部将传入的 Code 和对象转换成 Action 封装对象：
``` java
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
  ● RxJava 1
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
  ● RxJava 2