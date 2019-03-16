---
title: Flutter 底部导航栏页面的状态保持

grammar_cjkRuby: true
---
使用 Flutter 实现带底部导航栏的 APP 主页面，根节点使用 Scaffold，再配置 bottomNavigationBar 设置点击事件通过调用 `setState()` 方法在 build 返回不同的页面即可。但运行后会发现，点击切换页面后之前的页面状态无法保持，切换页面后再切回之前的页面，页面显示的数据会重绘为初始状态。这是因为 flutter 页面默认情况下每次都是返回的一个新的对象，重新绘制再界面的，不做任何处理的情况下自然不能保留数据。要解决这个问题，有下面的三种实现方式：

## 一、Stack + OffStage + TickerMode 堆叠
简单粗暴，跟 Android 开发中在 FrameLayout 中堆叠 View，再根据需要控制部分 View 的显示于隐藏，在构建页面时 body 直接返回一个 Stack 对象，子 widget 即为要显示的所有页面：
```dart
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      body: Stack(children: <Widget>[
        _makePage(0),
        _makePage(1),
        _makePage(2),
        _makePage(3),
      ]),
      bottomNavigationBar: _getNavigationBar(),
    );
	
 /*
   * Stack 方式创建页面
   */
  Widget _makePage(int index) {
    return Offstage(
        offstage: this._currentIndex != index,
        child: TickerMode(enabled: this._currentIndex == index, child: _pageList[index]));
  }
```
通过 TickerMode 的 enabled  来控制页面是否显示，当页面的 index 为当期选中的 index 时显示否则隐藏。

## 二、IndexedStack 控制页面的显示
类似 Android 中 FragementTransManager 控制 fragement 的显示隐藏，这与第一种方式没有太大本质上的区别，只不过是显示判断逻辑交给 IndexedStack 内部自动完成，与第一种方式相比代码量会少很多
``` dart
    // IndexStack 方式
    return new Scaffold(
      body: IndexedStack(index: _currentIndex, children: _pageList),
      bottomNavigationBar: _getNavigationBar(),
    );
```
## 三、PageView 
PageView 类似于 Android 中的 ViewPager 支持，页面之间的滑动切换。这种方式需要每一个要保持状态的 Widget 状态管理类实现 `AutomaticKeepAliveClientMixin<Widget>`,并且 `wangKeepAlive` 返回 true。伪代码如下：
``` dart
class WidgetState extends State<Widget>
    with AutomaticKeepAliveClientMixin<Widget> {
  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return Widget;
  }

  @override
  // TODO: implement wantKeepAlive
  bool get wantKeepAlive => true;
```
这个样 PageView 在切换时 flutter 内部就会知道你想要保持对应页面的状态，在页面切换时会将数据保留下来。这种方式是支持滑动切换页面的
# 总结
不需要支持左右滑动切换页面直接使用第二种方式实现导航栏效果，需要滑动切换页面效果则使用第三种方式。不推荐使用第一种方式实现此功能