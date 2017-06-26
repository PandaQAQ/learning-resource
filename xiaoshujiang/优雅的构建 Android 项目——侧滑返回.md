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
## 滑动处理及 ViewPager 处理
在 SwipeBackLayout 中通过重写 onInterceptTouchEvent(MotionEvent ev) 方法和 onTouchEvent(MotionEvent ev) 方法来实现侧滑返回事件的处理及对 ViewPager 滑动的兼容的。
``` java
@Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        //处理ViewPager冲突问题
        ViewPager mViewPager = getTouchViewPager(mViewPagers, ev);
        //当无触摸ViewPager或者该ViewPager未滑动到最左则不对滑动时间进行拦截
        if (mViewPager != null && mViewPager.getCurrentItem() != 0) {
            return super.onInterceptTouchEvent(ev);
        }
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                downX = tempX = (int) ev.getRawX();
                downY = (int) ev.getRawY();
                canSwipe = downX <= viewWidth / 2;
                if (!canSwipe) {
                    return super.onInterceptTouchEvent(ev);
                }
                break;
            case MotionEvent.ACTION_MOVE:
                if (!canSwipe) {
                    return super.onInterceptTouchEvent(ev);
                }
                int moveX = (int) ev.getRawX();
                // 满足此条件屏蔽SildingFinishLayout里面子类的touch事件
                if (moveX - downX > mTouchSlop
                        && Math.abs((int) ev.getRawY() - downY) < mTouchSlop) {
                    return true;
                }
                break;
        }
        return super.onInterceptTouchEvent(ev);
    }
```
在手指按下的时候相较于 onTouchEvent() 方法 onInterceptTouchEvent() 方法会先执行，在此方法中先判断当前触摸是否为 ViewPager，是 ViewPager 则判断是否滑动到了 ViewPager 的最左侧。如果触摸的 ViewPager 且未滑动到最左侧则不对事件进行拦截交给 ViewPager 处理触摸事件，否则触摸位置进行判断，在有效区域内则记录触摸开始点，否则按系统默认方式处理。在移动事件中会根据按下事件的判断结果决定是否按默认方式处理，当需要处理侧滑时会再次判断如果 X 方向的滑动大于最小有效滑动距离 Y方向滑动距离小于最小有效滑动距离则此次事件将会被 SwipeBackLayout 所消费，将进入 SwipeBackLayout 的 onTouchEvent() 方法中的处理逻辑。
``` java
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_MOVE:
                if (!canSwipe) {
                    return super.onInterceptTouchEvent(event);
                }
                int moveX = (int) event.getRawX();
                int deltaX = tempX - moveX;
                tempX = moveX;
                if (moveX - downX > mTouchSlop
                        && Math.abs((int) event.getRawY() - downY) < mTouchSlop) {
                    isSilding = true;
                }
                if (moveX - downX >= 0 && isSilding) {
                    //deltaX 为单次移动的距离向右滑为负数
                    // TODO: 2017/6/22 实现 y 方向的移动，即向右任意方向滑出界面
                    mContentView.scrollBy(deltaX, 0);
                }
                break;
            case MotionEvent.ACTION_UP:
                if (!canSwipe) {
                    return super.onInterceptTouchEvent(event);
                }
                isSilding = false;
                if (mContentView.getScrollX() <= -viewWidth / 4) {
                    isFinish = true;
                    scrollRight();
                } else {
                    scrollOrigin();
                    isFinish = false;
                }
                break;
        }
        return true;
    }
```
同样此方法中也会根据 onInterceptTouchEvent() 中的 DOWN 事件的判定结果 canSwipe 来决定是否按默认方式消费事件，MOVE 事件中如果满足侧滑条件则会调用 scrollBy() 将 mContentView 按滑动方向进行移动，而此处的 mContentView 即是 SwipeBackLayout 自身，因此整个显示的界面会被按照滑动方向移动。当手指抬起时如果滑动距离超过 1/4 界面宽度（可以按自己需求调整），则视为侧滑返回完成，让 Scroller 自动完成剩余距离的滑动，否则让 Scroller 恢复到滑动起始位置
``` java
    /**
     * 滚动出界面
     */
    private void scrollRight() {
        final int delta = (viewWidth + mContentView.getScrollX());
        // 调用startScroll方法来设置一些滚动的参数，我们在computeScroll()方法中调用scrollTo来滚动item
        mScroller.startScroll(mContentView.getScrollX(), 0, -delta + 1, 0,
                Math.abs(delta));
        postInvalidate();
    }

    /**
     * 滚动到起始位置
     */
    private void scrollOrigin() {
        int delta = mContentView.getScrollX();
		        // 调用startScroll方法来设置一些滚动的参数，我们在computeScroll()方法中调用scrollTo来滚动item
        mScroller.startScroll(mContentView.getScrollX(), 0, -delta, 0,
                Math.abs(delta));
        postInvalidate();
    }
	
	/**
     * 具体执行 Scroller 中的滚动及将滑动距离传递给外部接口
     */
	 @Override
    public void computeScroll() {
        Log.i("computeScroll","computeScroll");
        if (mSwipeListener != null) {
            double scrollx = Math.abs(mContentView.getScrollX());
            double offset = scrollx / viewWidth;
            if (offset > 0.9) {
                offset = 1d;
            }
            mSwipeListener.swipeValue(offset);
        }
        if (mScroller.computeScrollOffset()) {
            Log.i("computeScroll","mScroller");
            mContentView.scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
            postInvalidate();
            if (mScroller.isFinished() && isFinish) {
                mActivity.finish();
            }
        }
    }
```
# 结语
以上就是简化后的侧滑返回的基本使用和原理的简单分析，完整代码可以参考 [PandaEye][5]欢迎 Star。文章一遍过为反复检查如有不妥之处欢迎大家踊跃交流。


  [1]: https://github.com/ikew0ng/SwipeBackLayout
  [2]: http://oddbiem8l.bkt.clouddn.com/ViewTree.png
  [3]: http://oddbiem8l.bkt.clouddn.com/%E6%99%AE%E9%80%9ADecorView.jpg
  [4]: http://oddbiem8l.bkt.clouddn.com/%E4%BE%A7%E6%BB%91%E8%BF%94%E5%9B%9EDecorView.jpg
  [5]: https://github.com/PandaQAQ/PandaEye