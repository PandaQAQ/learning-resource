---
title:  Android 原生 WebView 与 JavaScript 交互
---
现在的纯原生的 APP 很少见了，几乎都是原生中嵌者网页的 hybird APP，或者直接使用 React Native 开发。原生 App 中嵌者网页原生与网页中 JavaScript 的交互几乎无法避免，毕竟在自己的应用中嵌入网页不只是想让应用当个浏览器用。
Android 原生中加载网页的控件是 WebView，因此要想原生与 JavaScript 交互也一样必须通过 WebView 来实现。
# 相关类和主要方法
## WebView
低版本和高版本的 Android 系统中 WebView 采用不同的内核， Android 4.4 之后直接使用了 Chrome 的内核。
### WebView 的常用方法
``` java
//根据 url 去加载一个网页
webView.loadUrl(String url);

//根据 HTML 数据显示出内容,其中 loadData() 方法的 data 中不能出现英文的 %，#，\，? 四个字符，否则可能会报错
webView.loadData(String data,String mimeType,String encoding);
webView.loadDataWithBaseUrl(String baseUrl,String mimeType,String encoding,String historyUrl);

// webView 处于激活状态，能正常加载和响应网页
webView.onResume();

// webView 处于暂停状态，当页面失去焦点切换到后台时调用
// 处于 pause 状态的 WebView 会停止动画和计算，但是不会停止 JavaScript 的执行
webView.onPause();

//暂停所有 WebView 的 layout、parsing、JavaScriptTimers 以降低 CPU 消耗（全局有效）
webView.pauseTimers();

// 恢复 pauseTimers 的暂停状态
webView.resumeTimers();

// 销毁一个 WebView 时要先将 WebView 和当前界面解绑再销毁
parentLayout.removeView(webView);
webView.destroy();

// 判断网页是否可以后退(可以实现监听，当网页可以后退事点击返回键和返回按钮不退出浏览界面)
webView.canGoBack();

//后退网页
webView.goBack();

// 判断网页是否可以前进
webView.canGoForward();

//前进网页
webView.goForward();

//以当前页为起点前进后退 steps 个页面(正数前进，负数后退)
webView.goBackOrForward(int steps);
```
## WebSetting
WebSetting 是为 WebView 提供配置和管理的一个抽象类，它通过 webView.getSettings() 方法获取实例。
###  WebSetting 的常用方法
``` java
// 设置 webView 是否支持 JavaScript 的调用（应用中涉及原生与 JS 交互的必须设置为 true）
webSettings.setJavaScriptEnabled(true);

// 设置是否允许 JS 开启新窗口(function window.open())
webSettings.setJavaScriptCanOpenWindowsAutomatically(true);

// 设置 webView 是否支持插件
webSettings.setPluginsEnable(true);

// 设置是否自适应屏幕，（一般图片和网页缩放同时使用）
webSettings.setUseWideViewPort(true); //将图片调整到适合 webView 的大小
webSettings.setLoadWithOverviewMode(true); //缩放至屏幕的大小（显示未做移动端兼容的 PC 网页）

// 设置是否支持缩放，默认为支持
webSettings.setSupportZoom(true);
// 设置内置的缩放控件
webSettings.setBuiltInZoomControls(true);
// 隐藏原生的缩放控件
webSettings.setDisplayZoomControls(true);


```
## WebClient
## WebChromeClient
# 原生调用 JS
# JS 调用原生方法