---
title:  Android 原生 WebView 与 JavaScript 交互
---
现在的纯原生的 APP 很少见了，几乎都是原生中嵌者网页的 hybird APP，或者直接使用 React Native 开发。原生 App 中嵌者网页原生与网页中 JavaScript 的交互几乎无法避免，毕竟在自己的应用中嵌入网页不只是想让应用当个浏览器用。
Android 原生中加载网页的控件是 WebView，因此要想原生与 JavaScript 交互也一样必须通过 WebView 来实现。
# 相关类和主要方法
## WebView
低版本和高版本的 Android 系统中 WebView 采用不同的内核， Android 4.4 之后直接使用了 Chrome 的内核。
**常用方法：**
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
**常用方法：**
``` java
// 设置 webView 是否支持 JavaScript 的调用（应用中涉及原生与 JS 交互的必须设置为 true）
webSettings.setJavaScriptEnabled(true);

// 设置是否允许 JS 开启新窗口(function window.open())
webSettings.setJavaScriptCanOpenWindowsAutomatically(true);

// 设置 webView 是否支持插件
webSettings.setPluginsEnable(true);

// 设置编码方式

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
## WebViewClient
WebViewClient 主要用来帮助 WebView 处理请求的各种状态事件。
**常用方法：**
``` java
// 页面开始加载的时候回调
onPageStarted(WebView view, String url, Bitmap favicon);
// 页面加载结束时回调
onPageFinished(WebView view, String url)；
// 将要加载资源时回调，每次资源的加载都会调用
onLoadResource(WebView view, String url);
// 加载的页面 404 时将会回调
onReceivedError(WebView view, WebResourceRequest request, WebResourceError error);
```
webView 默认情况下是不会加载 https 请求的，页面将显示空白，如果加载 https 页面需要进行如下设置：
``` java 
webView.setWebViewClient(new WebViewClient() {    
        @Override    
        public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {    
            handler.proceed();    //表示等待证书响应
        // handler.cancel();      //表示挂起连接，为默认方式
        // handler.handleMessage(null);    //可做其他处理
        }    
    });
```
## WebChromeClient
辅助 WebView 处理 JavaScript 对话框，加载进度，网页图标等。
**常用方法：**
``` java
// 加载进度回调
onProgressChanged(WebView view, int newProgress);
// 获取到网页 Title 回调
onReceivedTitle(WebView view, String title);
// 获取到网页图标回调
onReceivedIcon(WebView view, Bitmap icon);
```
# WebView 的使用
通过前面的了解，WebView 及他的辅助类的一些基本方法我们是知道了，接下来就看一下一下怎么使用 WebView 以及怎样通过 WebView 让 Android 源生代码与 JavaScript 代码进行交互。
## JS 调用原生方法
JS 调用源生的方法时该方法必须要加上 @JavascriptInterface 注解，我们可以定义一个类或者接口来单独存放用于 JS 调用的方法，这里我以接口为例。接口中提供一个 getUrl(String url) 方法用于提供给 JS 调用，传递一个 String 类型的值给源生方法：
``` java
/**
 * Created by PandaQ on 2017/3/22.
 * JS 接口方法定义
 * 接口中定义方法时不用加注解，使用时才加注解
 */
public interface JavaScriptFunction {
    void getUrl(String string);
}
```
``` java
// 此代码片段为 PandaEye 中点击加载的 html 数据中的图片跳转到新的 Activity 显示图片的功能
webView.addJavascriptInterface(new JavaScriptFunction() {
            @Override
            @JavascriptInterface // 加上注解 getUrl() 方法才能被 JS 调用
            public void getUrl(String imageUrl) {
                LogWritter.LogStr(imageUrl);
                Intent intent = new Intent();
                intent.putExtra("imageUrls", mImageUrls);
                intent.putExtra("curImageUrl", imageUrl);
                intent.setClass(ZhihuStoryInfoActivity.this, PhotoViewActivity.class);
                startActivity(intent);
            }
        }, "JavaScriptFunction");
```
``` javascript
// 在 HTML 中图片的点击事件 JS 方法中就可以执行如下代码来调用源生的接口方法
function(){
	window.JavaScriptFunction.getUrl(this.src);
}
```
## 原生调用 JS
在熊猫眼 [PandaEye][1] 中因为使用的知乎日报和网易新闻的 API 所以下发的 HTML body 数据需要我们自己加个 HTML 的壳，然后再源生加载我们需要的 JavaScrpit 代码来实现我们的功能。
加载 JS 可以直接 webView.load() 加载完整的 JS 代码也可以在自己加的 head 节点引入 JS 文件然后 webView 直接 load 其中的方法， [PandaEye][2] 中使用的是第二种方法：
在拼装 HTML 时在 head 中引入放在 asset 文件夹中的 js 文件

![enter description here][3]

imageClick.js 中的内容如下：
``` javascript
function initClick()
{
    var objects = document.getElementsByTagName("img");
    for(var i=0;i<objects.length;i++)
    {
        objects[i].onclick= function (){
            window.JavaScriptFunction.getUrl(this.src);
        }
    }
}
```
然后在 WebChromeClient 当加载进度达到 100% 后去调用 JS 文件中的 initClick() 方法，为每一张图片设置点击事件：
``` java
    /**
     * 为所有的图片添加点击事件
     *
     * @param webView 对应的 WebView
     */
    private void addImageClickListener(WebView webView) {
        webView.loadUrl("javascript:(initClick())()");
    }
```
直接加载 JS 把 initClick() 用 JS 代码替换掉即可。
# 利用 Chrome 调试 WebView
WebView 中 JavaScript 代码的调试直接使用 AndroidStudio 是没办法的，那么我们怎样调试 HTML 页面呢？答案是用 Chrome 浏览器来调试：
- USB 选项是打开的（AS 能调试应用就行）
- Chrome 浏览器打开地址：

  [1]: https://github.com/PandaQAQ/PandaEye
  [2]: https://github.com/PandaQAQ/PandaEye
  [3]: http://oddbiem8l.bkt.clouddn.com/htmlcore.png