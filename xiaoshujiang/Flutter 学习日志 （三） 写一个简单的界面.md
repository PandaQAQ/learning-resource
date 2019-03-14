---
title: Flutter 底部导航栏页面的状态保持

grammar_cjkRuby: true
---
## 一、Stack + OffStage + TickerMode 堆叠
简单粗暴，跟 Android 开发中在 FrameLayout 中堆叠 View，再根据需要控制部分 View 的显示于隐藏
## 二、IndexedStack 控制页面的显示
类似 Android 中 FragementTransManager 控制 fragement 的显示隐藏
## 三、PageView 
PageView 类似于 Android 中的 ViewPager 支持，页面之间的滑动切换

# 总结
不需要支持左右滑动切换页面直接使用第二种方式实现导航栏效果，需要滑动切换页面效果则使用第三种方式。不推荐使用第一种方式实现此功能