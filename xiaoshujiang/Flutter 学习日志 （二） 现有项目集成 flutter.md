---
title: Flutter 学习日志 （二） 现有项目集成 flutter
---

本文主要介绍怎样在现有的 Android 项目中集成 flutter 开发界面，以及 flutter 与原生之间怎样进行方法互调、数据双向传递
# 依赖集成
官方目前没有正式的集成文档，只有一个 github 上的 [wiki](https://github.com/flutter/flutter/wiki/Add-Flutter-to-existing-apps#experiment-turn-the-flutter-project-into-a-module) 页面做了相关介绍。原因嘛，看了 wiki 就知道了

>This page documents ongoing experiments and will be updated as we find more and better ways to do this, and as we build out tooling to aid.

>This support is in preview, and is generally only available in the master channel.

目前现有工程集成 flutter 还处在开发探索阶段。

## 注意
在现有的 Android 项目中集成 flutter 是以 module 的方式依赖到项目中。但 flutter 项目是创建在` Android 工程的同级目录下`的，而不是像一般 Android lib module 一样跟 app module 在同级目录,譬如要为 `D:\AndroidSpace\xxx\AndroidProject`工程集成 flutter 则 flutter module 也需要创建在 `D:\AndroidSpace\xxx\` 目录下。
## 集成步骤
1、命令行/powershell 进入到 Android 工程所在的目录，即上文举例中的 `xxx` 目录，执行 flutter 命令创建 flutter module。
```cmd
flutter create -t module flutter_module
```
2、待创建完毕后，打开 Android Project 的 `setting.gradle` 文件，添加如家代码
``` groovy
setBinding(new Binding([gradle: this]))                       
evaluate(new File( 
        settingsDir.parentFile,
        'flutter_check/.android/include_flutter.groovy'
))
```
3、同步设置后在 app gradle 中添加依赖(`flutter 中的 support 包可能会有依赖冲突，加入 exclude 排除掉 support 包依赖即可解决`)
``` groovy
dependencies {
    ...
    implementation(project(':flutter'),{
        exclude group: 'com.android.support'
    })
	...
```
4、经过上面的配置 flutter 已经集成到现有 Android 工程中了，可以将 flutter 页面当做 View 添加到原生界面上，并在界面的生命周期函数中调用 FlutterView 的对应生命周期方法。也可以创建一个 FlutterFragment 显示在界面上，由 Flutter 自行处理生命周期相关问题。
- 添加 flutterview 方式
```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
		...
        FrameLayout.LayoutParams layout = new FrameLayout.LayoutParams(FrameLayout.LayoutParams.MATCH_PARENT,
                FrameLayout.LayoutParams.MATCH_PARENT);
        flutterView = Flutter.createView(this, getLifecycle(), "PhoneCheck");
        addContentView(flutterView, layout);
		...
    }
``` 
- FlutterFragment 方式
```java
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_flutter_test);
		...
        FragmentTransaction tx = getSupportFragmentManager().beginTransaction();
        tx.replace(R.id.fl_flutter_container, Flutter.createFragment("PhoneCheck"));
        tx.commit();
		...
    }
```
当然，上面两种方式不是必须在 oncreate 中调用，当做普通的 View / fragment 在合适的地方使用就行。使用 Flutterfragment 时如果需要 flutter 与原生间进行通信则需要继承 Flutterfragment 重新 `onCreateView()` 拿到自动创建的 FlutterView 对象。因为初始化通信信道时需要用到 FlutterView 对象。另外在使用时还遇到了 `getLifecycle()` 方法找不到的问题，原因是 Android 工程的 `buildToolsVersion` 太低，之前是 25 后面改成 27 的就解决了。

# Flutter 与原生的方法互调及数据传递
通过上面的步骤，flutter 页面是可以嵌入显示在原生应用上了。但现阶段还是各显示各的互不交流，这显然是不满足需求的。好在 Flutter 为我们提供了通信的桥梁——平台插件，Flutter 与原生的通信通过两个类型的 Channel 来实现。即通过反射进行方法调用的 `MethodChannel` 和将数据转换成二进制比特数组直接传递的 `BasicMessageChannel` ，且通信都是双向的。Channel 支持传输的数据类型有以下：

![enter description here](http://oddbiem8l.bkt.clouddn.com/channel_msg_support.png)，如果需要传递自定义的对象数据等则需要转成 json 字符串或 map 对象传递过去再解析。
## MethodChannel
MethodChannel 是 Flutter 与平台原生进行方法互调的插件，Android 中通过反射实现方法调用。
### example
flutter module 对应的 State 中
``` dart
	...
	MethodChannel platform;
  @override
  void initState() {
	...
    platform = const MethodChannel('my_flutter/plugin');
    platform.setMethodCallHandler(methodHandler);
    super.initState();
	...
  }
  
   // 接收原生平台的方法调用处理
  Future methodHandler(MethodCall call) async {
	...
  }
```
初始化 WidgetState 的时候初始化 MethodChannel 并为之设置回调处理方法,回调方法中有参数 MethodCall，可通过 call.method 来判断需要执行的具体操作。
Android 原生代码
与 flutter 端一样，也需要先初始化 channel
```java
MethodChannel mChannel；
private void initChannel() {
        mChannel = new MethodChannel(flutterView, "my_flutter/plugin");
        
        mChannel.setMethodCallHandler((MethodCall methodCall, MethodChannel.Result result) -> {
            switch (methodCall.method) {
                case "action1":
                    ...
                    break;
                case "action2":
					...
                    result.success("jump");
                    break;
            }
        });
    }
```
### 调用方式
- flutter 端调用原生中的方法：
//  flutter 中有一个 getheaders 方法，从原生去获取 headers map 对象 
```dart
Map<String, String> headers;
  getHeaders() async {
    try {
      Map<dynamic, dynamic> result = await platform.invokeMethod('getHeaders');
	  // 将 dynamic 类型转换成需要的类型（与平台端返回值给定的类型要一致）
      headers = result.cast();
    } on PlatformException catch (e) {
      print(e.message);
    }
  }
```
在 flutter 中使用 MethodChannel 对象执行 `invoke（‘methodName’）`即可调用，如方法需要传递参数则调用 `invoke（‘methodName’[dynamic]）`，传递一个 dynamic 类型的参数数组。在原生平台中使用 `methodCall.arguments()`获取参数。 需要返回值则需要添加 await 修饰。原生平台中的 MethodCallHandler 回调就会接收到调用信息,`methodCall.method`值为 flutter 端 invoke 的方法名，result 对象设置的返回值则为 flutter 平台获取到的返回值
```java
mChannel.setMethodCallHandler((MethodCall methodCall, MethodChannel.Result result) -> {
            switch (methodCall.method) {
                case "action1":
                    ...
                    break;
                case "action2":
					...
                    result.success("jump");
                    break;
            }
        });
```
- 原生平台调用 flutter 方法
```java
//与 flutter 调用原生一样，使用 MethodChannel 的 invoke 方法
mChannel.invokeMethod("setAddress");
// 调用并传递参数
mChannel.invokeMethod("setAddress"，Object);
```
```dart
  Future methodHandler(MethodCall call,) async {
    print(call.method);
    if (call.method == 'setAddress') {
		...
    }
  }
```
flutter 中的 methodHandler 接收到方法调用，再做相关处理。与 flutter 调用原生不一样，原生调用 fultter 方法是没有返回值的。

##注意事项
- MethodChannel 初始化时，指定的 `channelName` 参数，如上面代码片段中的`my_flutter/plugin`。需要 flutter 端与原生平台端一致，才能保证 Channel 信道一致，互相接收到对放发送的方法调用请求。
- 参数传递时，传递的参数为基本数据类型或者 Map List 之类的双方都能识别的类型，自定义数据类转换成 map 或者 json 字符串再传递。当 flutter 传递的参数为 json对象（内部也是转成 map 传递的） 或者 map 时可以在原生中可以直接使用 `methodCall.argument("key");`获取参数值，获取前可以使用 `methodCall.hasArgument("key");`判断参数是否存在。

## BasicMessageChannel
BasicMessageChannel 是 Flutter 与平台原生直接发送和接收数据的插件，数据以二进制的形式在其中传输。
基本步骤和 MethodChannel 一致，在 flutter 端和平台端都实例化对象并设置回调，与 MethodChannel 初始化不同之处在于初始化时多了一个类型参数，因为 BasicMessageChannel 本身是可以传入泛型的，因此在初始化时需要指定泛型类型。其实打开 `MethodChannel` 的初始化方法会发现，内部也是传入了类型的，只不过使用的默认值 `StandardMethodCodec.INSTANCE`。具体有哪些类型实现，打开 project 依赖目录找到 flutter.jar 其中的 `io.flutter.common` 包中可以看到

| Class          | Note                         |
| -------------------- | ------------------------------------------------------------ |
| BinaryCodec          | 二进制数据类型，泛型对应 ByteBuffer,传递 byte 数组时使用                          |
| JSONMessageCodec     | Json消息类型，泛型对应 Object。传递 Json时可使用                                  |
| StringCodec          | 字符串消息类型，泛型对应 String。传递字符串                  |
| StandardMessageCodec | 标准消息类型，泛型对应 Object。传递 Map 等的时候可使用此类型 |
Flutter 中对应 class 在 `flutter\src\services\message_codecs.dart` 文件中。
# 总结
本文只是记录对照官方文档上手在 demo 中实践的心得，可能会理解有所偏差。待深入研究再进行纠正。flutter 与原生平台通信主要通过三个默认的 Channel ，除了上面说到的 `MethodChannel`、`BasicMessageChannel`还有一个 `EventChannel`具体可以看一下[这篇文章](https://blog.csdn.net/johnwcheung/article/details/79180732)。
 