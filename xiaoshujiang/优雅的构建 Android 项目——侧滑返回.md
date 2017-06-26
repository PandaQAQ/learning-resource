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
## 工作原理
要明白侧滑返回的原理我们得先明白 Android Activity 界面的视图层级关系：


## ViewPager 滑动冲突的处理

  [1]: https://github.com/ikew0ng/SwipeBackLayout