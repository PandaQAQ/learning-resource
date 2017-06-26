---
title: 优雅的构建 Android 项目——侧滑返回使用及原理分析
---
大屏幕手机在返回前页操作时，点击左上角的 APP 内返回键或者手机自带的返回按键都不是很方便，这时候能通过屏幕侧滑退出当前页面体验就会好很多了。但是 Android 系统并没有想 IOS 一样自带侧滑返回，好在 Android 轮子比较多，本文记录一下个人开源项目 PandaEye 中使用的侧滑返回库 SwipBackLayout 。该库参考 github 上的开源库 [SwipeBackLayout][1] 做了一些简化；
# 使用方式
## 定义侧滑基础 Activity
侧滑返回的实现是基于 Activity 的，可以直接继承 Activity 或者继承自己应用实现的 BaseActivity 然后实现 SwipeBackLayout.SwipeListener 接口即可.
``` java
public class SwipeBackActivity extends BaseActivity implements SwipeBackLayout.SwipeListener {
    protected SwipeBackLayout layout;
    private ArgbEvaluator argbEvaluator;

    @SuppressLint("InflateParams")
    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        layout = (SwipeBackLayout) LayoutInflater.from(this).inflate(
                R.layout.swipeback_base, null);
        layout.attachToActivity(this);
        argbEvaluator = new ArgbEvaluator();
        layout.addSwipeListener(this);
        if (Build.VERSION.SDK_INT >= 23) {
            currentStatusColor = getResources().getColor(R.color.colorPrimaryDark, null);
        } else {
            currentStatusColor = getResources().getColor(R.color.colorPrimaryDark);
        }
    }

	// 提供给子类设置 ViewPager 的接口，用于 SwipeLayout 中处理滑动冲突
    public void addViewPager(ViewPager pager) {
        layout.addViewPager(pager);
    }

}
```
## 效果优化
需要侧滑返回的 Activity 继承 SwipeBackActivity 即可实现侧滑返回的功能了，但是侧滑过程中返回界面会被默认的窗口背景颜色覆盖，因此我们需要把实现侧滑返回的界面的 theme 做一些小小的优化，将背景设置为透明状态，并设置进入和退出的动画。
style 中的属性设置
``` xml
    <!--全屏加透明-->
    <style name="TranslucentFullScreenTheme" parent="AppTheme">
        <item name="android:windowBackground">@android:color/transparent</item>
        <item name="android:colorBackgroundCacheHint">@null</item>
        <item name="android:windowIsTranslucent">true</item>
        <!--<item name="android:windowAnimationStyle">@android:style/Animation</item>-->
        <item name="android:windowAnimationStyle">@style/AnimationActivity</item>
    </style>
	<!--动画设置-->
	    <style name="AnimationActivity" mce_bogus="1" parent="@android:style/Animation.Activity">
        <item name="android:activityOpenEnterAnimation">@anim/base_slide_right_in</item>
        <item name="android:activityOpenExitAnimation">@anim/base_slide_right_out</item>
        <item name="android:activityCloseEnterAnimation">@anim/base_slide_right_in</item>
        <item name="android:activityCloseExitAnimation">@anim/base_slide_right_out</item>
    </style>
```
界面进入动画
``` xml
<!--base_slide_right_in-->
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android" >
    <translate
        android:duration="300"
        android:fromXDelta="100.0%"
        android:interpolator="@android:anim/decelerate_interpolator"
        android:toXDelta="0.0%" />

</set>
``` 
界面退出动画
``` xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android" >
    <translate
        android:duration="300"
        android:fromXDelta="100.0%"
        android:interpolator="@android:anim/decelerate_interpolator"
        android:toXDelta="0.0%" />

</set>
``` 
然后在 manifest 文件中将继承 SwipeBackActivity 的 Activity 的 theme 设置为 TranslucentFullScreenTheme 即可解决滑动过程中背景覆盖问题。
# 原理浅析
## Activity 中 View 视图层级
要明白侧滑返回的原理我们得先明白 Android Activity 界面的视图层级关系：

![Activity 界面视图层级][2]
Activity 和 PhoneWindow 这里可以忽略，重点在 DecorView上。这个 DecorView 是 Activity 中 View 布局的祖宗级布局，是一个 FrameLayout，通过 getWindow().getDecorView() 可以获取到对象；如图中 DecorView 有且仅有一个 LinearLayout 子布局，即图中的黄色部分。这个 LinearLayout 一般情况下又包含有 ViewStub 和 FrameLayout 两部分(不同的主题 Theme 可能会多处一些对象)，ViewStub 即是应用的 ActionBar，他会根据 theme 来决定是否真正引入 ActionBar 到界面显示。而这个  FrameLayout 中的内容即是我们写的 layout 布局文件中想要展示的内容。需要注意如果 Activity 继承自 AppComcatActivity 则这个 FrameLayout 中还会有两个子布局，第一个子布局中的内容才是我们写的布局文件中的内容
## 实现原理
通过 SDK 自带的视图分析工具 Hierarchy View 我们可以看到视图的如下分布：

![DecorView 视图节点][3]
界面所有显示的内容其实都在这个 LinearLayout 中，如果我们给这个 LinearLayout 增加一个父布局然后对这个父布局进行滑动处理就可以实现界面的整体滑动，即把整个可视界面放入一个滑动抽屉。因此实现滑动的界面视图应该变成如下的样子：

![可侧滑的界面的 DecorView 试图节点][4]
如图 SwipeBackLayout 即是添加的滑动抽屉，接下来我们看一下 SwipeBackLayout 中是怎样实现在 LinearLayout 上层插入一个 SwipeBackLayout 布局的。
在 SwipeBackActivity 中只调用了 attachToActivity() 方法，方法中代码如下：
``` java
    public void attachToActivity(Activity activity) {
        mActivity = activity;
        TypedArray a = activity.getTheme().obtainStyledAttributes(
                new int[]{android.R.attr.windowBackground});
        int background = a.getResourceId(0, 0);
        a.recycle();
        //获取到 DecorView 对象
        ViewGroup decor = (ViewGroup) activity.getWindow().getDecorView();
        Log.i("decorChildCount", decor.getChildCount() + "");
        ViewGroup decorChild = (ViewGroup) decor.getChildAt(0);
        Log.i("decorChild", decorChild.toString());
		//重置背景色资源
        decorChild.setBackgroundResource(background);
        //decorView 中将子布局移除
        decor.removeView(decorChild);
        //SwipeBackLayout 添加从decorView中移除布局
        addView(decorChild);
        //将ContentView设置为decorChild的父布局即添加进来的SwipeBackLayout
        setContentView(decorChild);
        //将SwipeBackLayout添加进DecorView
        decor.addView(this);
    }
```
从中我添加的注释不难看出，实现替换的流程：
- 1、传入的 activity 对象获取到 DecorView
- 2、DecorView.getChildAt(0) 获取到 LinearLayout 对象
- 3、将 LinearLayout 背景资源重置，并从 DecorView 中移除
- 4、将 LinearLayout  添加到自定义的 SwipeBackLayout 中
- 5、将自定义的 SwipeBackLayout 添加到 DecorView 中
## 滑动处理

## ViewPager 滑动冲突的处理


  [1]: https://github.com/ikew0ng/SwipeBackLayout
  [2]: http://oddbiem8l.bkt.clouddn.com/ViewTree.png
  [3]: http://oddbiem8l.bkt.clouddn.com/%E6%99%AE%E9%80%9ADecorView.jpg
  [4]: http://oddbiem8l.bkt.clouddn.com/%E4%BE%A7%E6%BB%91%E8%BF%94%E5%9B%9EDecorView.jpg