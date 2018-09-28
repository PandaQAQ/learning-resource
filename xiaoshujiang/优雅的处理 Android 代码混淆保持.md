---
title: 优雅的处理 Android 代码混淆保持
grammar_cjkRuby: true
---
为了源码安全以及缩小 APK 体积，Android 应用发布前是必须要进行混淆打包的。而混淆打包并不是全量打包，特定的类、方法、属性是需要排除在混淆之外的，比如数据模型类，自定义 View 等在混淆时如果不通过规则排除在外可能存在运行时找不到资源的问题。
# 常规操作
常规操作有以下两种方式：
- 要保持的代码规则逐条添加到 `proguard-rules.pro` 混淆规则文件中

	**存在问题**：随着代码量的不断增大，混淆规则会爆炸式增长。还会经常忘记将资源加入到混淆规则中

- 要保持的类方法统一的包中，再讲整个包的保持规则加入到 `proguard-rules.pro` 混淆规则文件中

	**存在问题**：
	1、keep 粒度是 `Class` ，如果只想 keep某个方法或者某个属性就不行了
	2、会干预代码的目录，不利于工程内部代码隔离
# 简化操作
针对常规操作的问题，做出对应的处理就得到了简约操作 —— Tag 标记法，要做到不干预代码目录，每个类都能标记，因此使用接口对 `Class`进行标记，Java 类多继承的特性不会对代码有其他的干扰。
- 定义空接口，将接口保持规则加入到 `proguard-rules.pro` 混淆规则文件中，需要保持的类实现接口，使用接口标记身份
- 
```java
// 与项目包名路径对应
-keep public interface com.company.project.KeepClass{public *;}
-keep class * implements com.company.project.KeepClass {
<methods>;
<fields>;
} 
```
**存在问题**：
1、每个 keep 类都要继承比较麻烦（当然比每个类写 keep 要省事多了...）
2、依然不能只 keep 方法或者属性，不 keep 整个类


# 优雅操作
同样针对`接口标记法`中的存在的问题，进一步优化得到了注解标记法。因为注解可以标记类、方法、属性，因此可以将 keep 粒度降到属性方法上。
- 1、写一个空注解类
``` java
@Retention(RetentionPolicy.CLASS)
@Target({ElementType.TYPE, ElementType.METHOD, ElementType.CONSTRUCTOR, ElementType.FIELD})
public @interface ProguardKeep {

}
```
- 2、混淆规则中添加如下规则
``` java
-keep @com.pandaq.appcore.framework.annotation.ProguardKeep class * {*;}
-keep class * {
    @com.pandaq.appcore.framework.annotation.ProguardKeep <fields>;
}
-keepclassmembers class * {
    @com.pandaq.appcore.framework.annotation.ProguardKeep <methods>;
}
```
- 3、在需要 keep 的地方使用注解
``` java
// 示例需要，使用时不需要类、方法、属性 全都给加上注解
@ProguardKeep
public class UserInfo {
    @ProguardKeep
    private String account;
    private String userName;
    private String token;

    @ProguardKeep
    public String getAccount() {
        return account;
    }
}
```
# 总结
使用注解标记,大大的减少了 keep 规则文件内容，开发时只需要记住在需要保持的地方加上 `@ProguardKeep` ,这也降低了忘记添加混淆规则的风险。当然这只是针对项目自写代码，三方库三方框架的混淆规则还是需要 copy 进规则文件的。

[我的简书地址](https://www.jianshu.com/p/3e7a90070a2b)