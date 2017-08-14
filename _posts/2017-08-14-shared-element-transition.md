---
layout: post
title: "Shared Element Activity Transition"
date:   2017-8-14 14:06:00 +0800
---
# 概述
传统的acitivity和fragment的进入和退出的过渡动画都是整个视图的，诸如淡入淡出、滑入滑出等动画效果。
但是很多情况下，Acitivities之间有共有的元素，让这些共有的元素分别有个过渡动画，使人眼无缝切换，带来更好的用户体验。
# 共享元素变化的实现步骤
注意：该特性仅支持 Android 5.0以上系统，所以使用某些方法时需运行时检查系统版本，或使用兼容库里的方法
## 1.启用窗口共享内容变化
styles.xml文件中启用：
{% highlight xml %}
<!-- Base application theme. -->
<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
    <!-- Customize your theme here. -->
    <item name="android:windowContentTransitions">true</item>
    ...
</style>
{% highlight %}
