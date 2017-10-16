---
title: "Android 开发技术交流-2017-10-14"
date:   2017-10-14 21:35:00 +0800
categories: 编程世界 
tags: 
  - develop
  - android
published: false
---
<!--more-->
[](){: more }
## kotlin 使用的一细节
* lateinit
* var & val
* 标准委托、委托属性
* 扩展函数

学习资料：[Kotlin for android developers](https://wangjiegulu.gitbooks.io/kotlin-for-android-developers-zh/)

## 利用 styles.xml、dimens.xml、colors.xml 等提高代码复用性，使整体风格统一
```xml
<style name="ContentText">
	<item name="android:textSize">@dimen/font_normal</item>
	<item name="android:textColor">@color/basic_black</item>
</style>
```
```xml
<resources>
	<!-- grayscale -->
	<color name="white"     >#FFFFFF</color>
	<color name="gray_light">#DBDBDB</color>
	<color name="gray"      >#939393</color>
	<color name="gray_dark" >#5F5F5F</color>
	<color name="black"     >#323232</color>

	<!-- basic colors -->
	<color name="green">#27D34D</color>
	<color name="blue">#2A91BD</color>
	<color name="orange">#FF9D2F</color>
	<color name="red">#FF432F</color>
	...
</resources>
```

```xml
<resources>
	<!-- font sizes -->
	<dimen name="font_larger">22sp</dimen>
	<dimen name="font_large">18sp</dimen>
	<dimen name="font_normal">15sp</dimen>
	<dimen name="font_small">12sp</dimen>

	<!-- typical spacing between two views -->
	<dimen name="spacing_huge">40dp</dimen>
	<dimen name="spacing_large">24dp</dimen>
	<dimen name="spacing_normal">14dp</dimen>
	<dimen name="spacing_small">10dp</dimen>
	<dimen name="spacing_tiny">4dp</dimen>

	<!-- typical sizes of views -->
	<dimen name="button_height_tall">60dp</dimen>
	<dimen name="button_height_normal">40dp</dimen>
	<dimen name="button_height_short">32dp</dimen>
</resources>
```

## 利用 Lint 检查排查可能写得不合理的代码

## 使用 MVP 模式
[mosby](https://github.com/sockeqwe/mosby)

## 如何写出更好性能的程序
* layout 减少视图层次，建议使用谷歌新推的 Constraitlayout

    [了解使用ConstraintLayout 的性能优势](http://developers.googleblog.cn/2017/09/constraintlayout.html)

* 避免在 onDraw() for循环中大量创建新对象，减少内存抖动
* 避免内存泄漏

## 其他细节
* LoginHelper.java
* JavaBean

