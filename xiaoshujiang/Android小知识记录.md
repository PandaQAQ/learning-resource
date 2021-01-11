---
title: Android小知识记录
---
**Android5.0及以上的水波纹效果，在资源文件中创建ripple 的xml文件**

token ：835b2248fdbad2bb543df3134b001079bbe3737b

``` xml
<?xml version="1.0" encoding="utf-8"?>
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
  android:color="#FF21272B">
  <item>
    <shape android:shape="rectangle">
      <solid android:color="#FFFFFF" />
      <corners android:radius="4dp" />
    </shape>
  </item>
  <item android:drawable="@drawable/rounded_corners" />
</ripple>
```
在需要用水波纹的控件中设置background为此文件。ripple中可以通过item为背景设置其他属性如颜色圆角等。
这需要API 21及以上才支持，因为ripple是API21才添加的属性，更低版本需要水波纹效果则需要通过canvas绘制。

**Transition**
Android5.1之后的activity和Fragment的变换都是建立在Transition上的。transition框架为在不同的UI状态之间产生动画效果提供了非常方便的API。该框架主要基于两个概念：场景（scenes）和变换（transitions）。场景（scenes）定义了当前的UI状态，变换（transitions）则定义了在不同场景之间动画变化的过程。

当一个场景改变的时候，transition主要负责：

（1）捕捉每个View在开始场景和结束场景时的状态。

（2）根据两个场景（开始和结束）之间的区别创建一个Animator。

**获取大型网站的时间戳（获取网络时间）**
``` java
    public static long getNetWorkTime() {
        try {
            URL url = new URL("http://www.baidu.com");
            URLConnection uc = url.openConnection();// 生成连接对象
            uc.connect(); // 发出连接
            return uc.getDate();
        } catch (MalformedURLException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
            return 0;
        }// 取得资源对象
        catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
            return 0;
        }
    }
```
**SP DP 跟PX的转换**
``` java
/** 
     * dp、sp 转换为 px 的工具类 
     *  
     * @author fxsky 2012.11.12 
     * 
     */  
    public class DisplayUtil {  
        /** 
         * 将px值转换为dip或dp值，保证尺寸大小不变 
         *  
         * @param pxValue 
         * @param scale 
         *            （DisplayMetrics类中属性density） 
         * @return 
         */  
        public static int px2dip(Context context, float pxValue) {  
            final float scale = context.getResources().getDisplayMetrics().density;  
            return (int) (pxValue / scale + 0.5f);  
        }  
      
        /** 
         * 将dip或dp值转换为px值，保证尺寸大小不变 
         *  
         * @param dipValue 
         * @param scale 
         *            （DisplayMetrics类中属性density） 
         * @return 
         */  
        public static int dip2px(Context context, float dipValue) {  
            final float scale = context.getResources().getDisplayMetrics().density;  
            return (int) (dipValue * scale + 0.5f);  
        }  
      
        /** 
         * 将px值转换为sp值，保证文字大小不变 
         *  
         * @param pxValue 
         * @param fontScale 
         *            （DisplayMetrics类中属性scaledDensity） 
         * @return 
         */  
        public static int px2sp(Context context, float pxValue) {  
            final float fontScale = context.getResources().getDisplayMetrics().scaledDensity;  
            return (int) (pxValue / fontScale + 0.5f);  
        }  
      
        /** 
         * 将sp值转换为px值，保证文字大小不变 
         *  
         * @param spValue 
         * @param fontScale 
         *            （DisplayMetrics类中属性scaledDensity） 
         * @return 
         */  
        public static int sp2px(Context context, float spValue) {  
            final float fontScale = context.getResources().getDisplayMetrics().scaledDensity;  
            return (int) (spValue * fontScale + 0.5f);  
        }  
    }  
```

**打包使用图片时 出现ide common。。。。debugresouce的错误时注意检查图片格式**

**textview 设置 singleLine = true   ellipsize = marquee 可以实现跑马灯显示较长的文字。 android:marqueeRepeatLimit  为循环次数，可无限循环。**

**RecyclerView 删除Item后只使用notifyItemRemoved() 删除了，但是剩下的item position还是没改变。需要刷新ViewHolder 或调用notifyItemRangeChanged(position, mDevices.size() - position);**

static 变量只会加载一次，程序结束静态内存区才会消失。
static 变量共享静态内存，当一个地方改变该变量的值的时候，其他引用该变量的值也会被改变。

元素共享动画 共享 ImageView 如果 两个界面的 ImageView 显示的图片不是同一张缓存下来需要重新再网络加载则会出现显示异常 图片有边距

如果为界面设置了转场动画则调用 finish() 不会执行动画，需要调用 finishAfterTranslation()

RxAndroid 解绑后再次调用会出现 结果无法获取，例如在 fragment 的 onpause() 中解绑观察，进到其他页面再回 fragment 去用 RxAndroid + Retrofit 刷新数据 无法触发 OnNext 中的结果

出现上面问题的原因：
统一使用的 CompositeSubscription 在解绑之后需要创建新的实例，如果 CSb自己都处于未绑定状态自然也就没法让 subcriber绑定观察者

**ScrollView 中元素设置么 marginTop 会导致不能滚动到底部 mScrollView.smoothScrollTo(0,0)；**

**oncreate 中获取控件宽高：**
        ViewTreeObserver vto = ssidtext.getViewTreeObserver();
        vto.addOnPreDrawListener(new ViewTreeObserver.OnPreDrawListener() {
            public boolean onPreDraw() {
             if (hasMeasured == false)
                {
                    int height = zoomImageView.getMeasuredHeight();
                    int width = zoomImageView.getMeasuredWidth();
                    Log.i("height",height+"");
                    Log.i("width",width+"");
                    hasMeasured = true;
                }
                return true;
            }
        });
		
**bitmap 图片转换方法**
``` java
	public Bitmap getCircleBitmap(Bitmap bitmap, int bitmapSize) {
        //前面同上，绘制图像分别需要bitmap，canvas，paint对象
        bitmap = Bitmap.createScaledBitmap(bitmap, bitmapSize, bitmapSize, true);
        Bitmap bm = Bitmap.createBitmap(bitmapSize, bitmapSize, Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(bm);
        Paint paint = new Paint(Paint.ANTI_ALIAS_FLAG);
        //这里需要先画出一个圆
        canvas.drawCircle(bitmapSize / 2, bitmapSize / 2, bitmapSize / 2, paint);
        //圆画好之后将画笔重置一下
        paint.reset();
        //设置图像合成模式，该模式为只在源图像和目标图像相交的地方绘制源图像
        paint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));
        canvas.drawBitmap(bitmap, 0, 0, paint);
        paint.setColor(Color.RED);
        canvas.drawRect(0, 3 * bitmapSize / 5, bitmapSize, 5 * bitmapSize / 6, paint);
        return bm;
    }
```
	
![enter description here][1]

**BottomSheetBehavior 需要在初始化的时候设置高度，否则不会有任何效果。再通过dispatchOntouchEvent 中判断滑动状态决定是否隐藏**
  
**BroadCastReceiver 静态注册后要在应用停止后还监听广播需要在发送时设置 Flag FLAG_INCLUDE_STOPPED_PACKAGES;**

**YUV图像数组转Bitmap**
``` java
public Bitmap rawByteArray2RGBABitmap2(byte[] data, int width, int height) {  
        int frameSize = width * height;  
        int[] rgba = new int[frameSize];  
  
            for (int i = 0; i < height; i++)  
                for (int j = 0; j < width; j++) {  
                    int y = (0xff & ((int) data[i * width + j]));  
                    int u = (0xff & ((int) data[frameSize + (i >> 1) * width + (j & ~1) + 0]));  
                    int v = (0xff & ((int) data[frameSize + (i >> 1) * width + (j & ~1) + 1]));  
                    y = y < 16 ? 16 : y;  
  
                    int r = Math.round(1.164f * (y - 16) + 1.596f * (v - 128));  
                    int g = Math.round(1.164f * (y - 16) - 0.813f * (v - 128) - 0.391f * (u - 128));  
                    int b = Math.round(1.164f * (y - 16) + 2.018f * (u - 128));  
  
                    r = r < 0 ? 0 : (r > 255 ? 255 : r);  
                    g = g < 0 ? 0 : (g > 255 ? 255 : g);  
                    b = b < 0 ? 0 : (b > 255 ? 255 : b);  
  
                    rgba[i * width + j] = 0xff000000 + (b << 16) + (g << 8) + r;  
                }  
  
        Bitmap bmp = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888);  
        bmp.setPixels(rgba, 0 , width, 0, 0, width, height);  
        return bmp;  
    }  

```
**Android 禁用系统截屏（敏感信息页面禁用截屏）**
``` java
getWindow().addFlags(WindowManager.LayoutParams.FLAG_SECURE);
```
**判断软键盘是否弹出**
实现思路：判断当前界面可见区域高度是否跟 DecorView 相等（带软键盘的要减去虚拟按键高度）
``` java
private boolean isSoftShowing() {  
       //获取当前屏幕内容的高度  
       int screenHeight = getWindow().getDecorView().getHeight();  
       //获取View可见区域的bottom  
       Rect rect = new Rect();  
       getWindow().getDecorView().getWindowVisibleDisplayFrame(rect);  
       //return screenHeight - rect.bottom != 0;  //无虚拟按键
	   return screenHeight-rect.bottom-getSoftButtonsBarHeight() !=0;
   } 
   
/** 
   * 底部虚拟按键栏的高度 
   * @return 
   */  
  @TargetApi(Build.VERSION_CODES.JELLY_BEAN_MR1)  
  private int getSoftButtonsBarHeight() {  
      DisplayMetrics metrics = new DisplayMetrics();  
      //这个方法获取可能不是真实屏幕的高度  
      mActivity.getWindowManager().getDefaultDisplay().getMetrics(metrics);  
      int usableHeight = metrics.heightPixels;  
      //获取当前屏幕的真实高度  
      mActivity.getWindowManager().getDefaultDisplay().getRealMetrics(metrics);  
      int realHeight = metrics.heightPixels;  
      if (realHeight > usableHeight) {  
          return realHeight - usableHeight;  
      } else {  
          return 0;  
      }  
  } 
  
  private void hideKeyBord(){
  	  InputMethodManager imm = (InputMethodManager)  
         getSystemService(Context.INPUT_METHOD_SERVICE);  
         imm.hideSoftInputFromWindow(v.getWindowToken(), 0); 
  }
```
**获取一个 View 或者 ViewGroup 的 Bitmap 信息**
``` java
    /**
     * 将view生成bitmap
     *
     * @param view
     * @param width
     * @param height
     * @return
     */
    public static Bitmap getBitmap(View view, int width, int height) {
        view.setDrawingCacheEnabled(true);
        view.measure(
                View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED),
                View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED));
        view.layout(0, 0, width, height);
        view.buildDrawingCache();
        return view.getDrawingCache();
    }
```
**旋转 Bitmap 图片**
``` java
    /**
     * bitmap 旋转
     *
     * @param bm
     * @param orientationDegree
     * @return
     */
    private static Bitmap adjustPhotoRotation(Bitmap bm, final int orientationDegree) {
        Matrix m = new Matrix();
        m.setRotate(orientationDegree, (float) bm.getWidth() / 2, (float) bm.getHeight() / 2);
        try {
            return Bitmap.createBitmap(bm, 0, 0, bm.getWidth(), bm.getHeight(), m, true);
        } catch (OutOfMemoryError ex) {
            return null;
        }
    }
```
**Android 8.0 透明 Activity 闪退问题 **
``` java
Only fullscreen opaque activities can request orientation
```

将原来的windowIsTranslucent改为windowDisablePreview

**多 module 混淆规则配置**
<span style="border-bottom:2px solid red;">子模块混淆文件的指定是通过consumerProguardFiles这个属性来指定的，并不是proguardFiles 属性，而且我们无需配置其他的选项，只需要配置consumerProguardFiles属性就可以。该属性表示在打包的时候会自动寻找该module下我们指定的混淆文件对代码进行混淆。</span>
**布局性能优化相关**
1、当布局只会作为 include 引入父布局时在不影响显示的情况下可使用` <merge/>`标签代替根布局节点，在合并时 ` <merge/>` 标签将会自动被忽略
2、使用 ViewStub ***(要代替的布局中不能包含 `<merge/>`)***

**编译遇到问题**
AS 终端执行如下命令可编译查看具体错误
``` cmd
gradlew compileDebugJavaWithJavac --stacktrace 
```
**postDelay()**
使用 postDelay 延迟调用时一定要注意执行方法中的空指针问题，如果延迟加载时间内页面关闭并回收则会报空指针错误

  [1]: http://oddbiem8l.bkt.clouddn.com/custom_bitmap.jpg
  
**注解依赖 annotationProcessor**
使用 annotationProcessor 注解依赖时必须配置在使用注解的 module 内。否则生成的代码不在对应module内会报空指针

**ScrollView 与 WebView 嵌套导致 WebView 显示不完整**

 给 webview 的父布局添加属性 
```xml
android:descendantFocusability="blocksDescendants"
```
属性的值有三种：

		 beforeDescendants：viewgroup会优先其子类控件而获取到焦点

        afterDescendants：viewgroup只有当其子类控件不需要获取焦点时才获取焦点

        blocksDescendants：viewgroup会覆盖子类控件而直接获得焦点
		
** WebView 加载 html 片段。文本中的换行符 "\n" 需要替换成 "\<br>"

**自动化构建过程**
- gitlab 推送代码
- 配置编译时替换参数
- 配置编译时进行 SnorCube Scanner 扫描代码规范
- 生成扫描结果报告
- 生成 apk 安装包

** Android8.0 华为手机BUG**
可隐藏导航栏模式，在未展开导航栏时进入其他应用，自己应用进入后台。展开导航栏再回到自己应用会触发页面重建。处理办法为：Manifest 中对应 Activity 设置 configchanges 加上 screenlayout 属性，对这种情况的页面重构进行屏蔽

**popupwindo 要沉浸式，设置 matchparent 并且 设置 isClippingEnable = false**

**RxJava2 异常处理**
- 慎用 Consumer 回调，确保事件流永远不会出错时才使用
``` kotlin
.subscribe(object : Consumer<String> {
                                override fun accept(t: String?) {
                                    Log.d("result: ", t)
                                }
                            })
```
否则必须添加再添加个异常处理，或者使用带异常处理的 Observer 回调。
**Seekbar 去掉烦人的点击效果**
```java
    android:thumb="@null"
    android:background="@null"
```
** SurfaceView 与 TextureView**
surfaceview 必须在顶层才可显示，textureview 不需要在顶层，可在上面覆盖布局

**AndroidStudio build 日志乱码问题**
在Android Studio/bin目录下的studio64.exe.vmoptions文件添加一行
```xml
-Dfile.encoding=UTF-8
```
** RecyclerView 刷新 item 会渐显出现**
```kotlin
(rv_coupons.itemAnimator as SimpleItemAnimator).supportsChangeAnimations = false
```
**拦截器中获取 Retrofit 接口中的注解内容**
```java
     Invocation invocation = request.tag(Invocation.class);
        if (invocation != null) {
            ZipBody zipBody = invocation.method().getAnnotation(ZipBody.class);
            if (zipBody != null) {
                zipType = zipBody.type().type;
            }
        }
```
**Java 压缩算法对比**
https://blog.csdn.net/zero__007/article/details/79782846
**Kotlin RecyclerView使用问题** 
创建 View 时使用 item.value? 设置值时注意 null 时情空 View，避免 View 复用显示之前的数据。与 View 相关的监听修改 item 数据值因为 View 复用可能导致数据不一致 bug