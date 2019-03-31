---
title: Android 组件化的个人理解 
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

# 为什么要做组件化
# 组件化过程中的几个问题
## 多个组件 module 怎样共用 Application
## Activity 及 Fragment 生命周期
## 组件间的通信
组件中的通信这里采用了 ARouter
### 注意事项
1、ARouter 分组不能重复
2、使用 AutoWired 绑定服务时一定要在对应位置调用 `ARouter.getInstance().inject(this)` 去发现服务，否则会找不到服务
## ORM 数据库，数据的共享

