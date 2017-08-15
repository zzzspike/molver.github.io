---
title: "Android共享元素变化实现总结"
date:   2017-8-14 14:06:00 +0800
categories: code
tags: android
---

## 概述
传统的 activity 和 fragment 的进入和退出的过渡变化都是整个视图的，有诸如淡入淡出、滑入滑出等动画效果。但是很多情况下， Activities 之间有共有的元素，让这些共有的元素分别有个过渡动画，使人眼无缝切换，可以带来更好的用户体验。
下面让我们看下如何实现共享元素变化，并分享下个人在实现过程中遇到的一些问题。以下为我实现效果图:   
![效果图](/assets/images/shared_element_transition.gif)

## 共享元素变化的实现
注意：该特性仅支持 Android 5.0以上系统，所以使用某些方法时需运行时检查系统版本，或使用兼容库里的方法

### 启用窗口共享内容变化
styles.xml文件中启用：
<!-- more -->
{% highlight xml %}
<!-- Base application theme. -->
<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
    <!-- Customize your theme here. -->
    <item name="android:windowContentTransitions">true</item>
    ...
</style>
{% endhighlight %}
在代码中运行时启用：    
{% highlight java %}
window.requestFeature(Window.FEATURE_CONTENT_TRANSITIONS)
{% endhighlight %}

### 指定一个相同的 Transition Name
在布局文件中使用 `android:transitionName` 标签来给共享元素指定变化名称    
或在运行时指定 `ViewCompat.setTransitionName(shareView, transitionName);`

### 启动一个 Activity
{% highlight java %}
Intent intent = new Intent(this, DetailsActivity.class);
ActivityOptionsCompat options = ActivityOptionsCompat.
    makeSceneTransitionAnimation(this, shareView, ViewCompat.getTransitionName(shareView);
startActivity(intent, options.toBundle());
{% endhighlight %}
在从第二个activity返回时，用 `supportFinishAfterTransition()` 代替 `finish()`   
#### 多个共享元素的情况：
{% highlight java %}
Intent intent = new Intent(context, SecondActivity.class);
Pair<View, String> p1 = Pair.create(shareView1, "transitionName1");
Pair<View, String> p2 = Pair.create(shareView2, "transitionName2");
Pair<View, String> p3 = Pair.create(shareView3, "transitionName3");
ActivityOptionsCompat options = ActivityOptionsCompat.
 makeSceneTransitionAnimation(this, p1, p2, p3);
startActivity(intent, options.toBundle());
{% endhighlight %}

### 自定义共享元素变化
 共享元素变化方式默认是 ChangeBounds, ChangeTransform, ChangeImageTransform 和 ChangeClipBounds 的组合，通常运作完美，但也可以通过以下方式进行自定义
{% highlight xml %}
<!-- Base application theme. -->
<style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
    <!-- enable window content transitions -->
    <item name="android:windowContentTransitions">true</item>

    <!-- specify enter and exit transitions -->
    <!-- options are: explode, slide, fade -->
    <item name="android:windowEnterTransition">@transition/change_image_transform</item>
    <item name="android:windowExitTransition">@transition/change_image_transform</item>

    <!-- specify shared element transitions -->
    <item name="android:windowSharedElementEnterTransition">
      @transition/change_image_transform</item>
    <item name="android:windowSharedElementExitTransition">
      @transition/change_image_transform</item>
</style>
{% endhighlight %}
{% highlight xml %}
<!-- res/transition/change_image_transform.xml -->
<transitionSet xmlns:android="http://schemas.android.com/apk/res/android">
  <changeImageTransform/>
</transitionSet>
{% endhighlight %}

### 共享元素异步加载情况
在 `onCreate` 中调用 `supportPostponeEnterTransition();` ，让transition暂停进行
{% highlight java %}
//使用Glide或Picasso加载图片后
imageView.getViewTreeObserver().addOnPreDrawListener(
    new ViewTreeObserver.OnPreDrawListener() {
        @Override
        public boolean onPreDraw() {
            ivBackdrop.getViewTreeObserver().removeOnPreDrawListener(this);
            //让transition开始运行
            supportStartPostponedEnterTransition();
            return true;
        }
    }
);
{% endhighlight %}

### RecyclerView  -> ViewPager 共享动画实现
#### transitionName 在整个控件树中应该是唯一的
在 RecyclerViewAdapter和ViewPagerAdapter中，可用position或item的id拼装成transitionName保证其唯一   
`ViewCompat.setTransitionName(shareView, "transitionName" + position)`
#### ViewPager中滑动到另外一个page时，共享元素如何更改为当前界面里的元素？
`RecyclerViewActivity` 应使用 `startActivityForResult()` 来启动 `ViewPagerActivity`    
当 `ViewPager` 页面发生变化时，通过 `setEnterSharedElementCallback()` 方法来修改进入的共享元素，并将退出时的位置传递给 `RecyclerViewActivity` 
{% highlight kotlin %}
    override fun onBackPressed() {
        val intent = Intent()
        intent.putExtra("exit_pos", currentPage)
        if (enterPosition != currentIndex) {
            val shareView = getShareView(currentPage) 
            setEnterSharedElementCallback(shareView)
        }
        setResult(Activity.RESULT_OK, intent)
        supportFinishAfterTransition()
    }

    fun setEnterSharedElementCallback(sharedView: View) {
        setEnterSharedElementCallback(object : SharedElementCallback() {
            override fun onMapSharedElements(names: MutableList<String>?, sharedElements: MutableMap<String, View>?) {
                names?.clear()
                sharedElements?.clear()
                names?.add(ViewCompat.getTransitionName(sharedView))
                sharedElements?.put(ViewCompat.getTransitionName(sharedView), sharedView)
            }
        })
    }

{% endhighlight %}
`RecyclerViewActivity` 在开始共享元素变化前会调用 `onActivityReenter();` 回调，在这里暂停transition，拿到传递传递过来的 `exitPosition` ，更新共享元素后再开始动画。
{% highlight kotlin %}
    override fun onActivityReenter(resultCode: Int, data: Intent?) {
        if (resultCode == Activity.RESULT_OK && data != null) {
            supportPostponeEnterTransition()
            this.exitPosition = data.getIntExtra("exit_pos", enterPosistion)
            if (enterPosistion != exitPosition) {
                recyclerView.scrollToPosition(exitPosition ?: 0)
                val newShareView = getShareView(exitPosition)
                if (newShareView != null)
                    setCallback(newShareView)
            }
            recyclerView.viewTreeObserver.addOnPreDrawListener(object : ViewTreeObserver.OnPreDrawListener {
                override fun onPreDraw(): Boolean {
                    albumRv.viewTreeObserver.removeOnPreDrawListener(this)
                    albumRv.requestLayout()
                    supportStartPostponedEnterTransition()
                    return false
                }
            })
        }
        super.onActivityReenter(resultCode, data)
    }
    
    fun setCallback(newShareView: View) {
        setExitSharedElementCallback(object : SharedElementCallback() {
            override fun onMapSharedElements(names: MutableList<String>?, sharedElements: MutableMap<String, View>?) {
                super.onMapSharedElements(names, sharedElements)
                val newShareName = ViewCompat.getTransitionName(newShareView)
                names?.clear()
                sharedElements?.clear()
                names?.add(newShareName)
                sharedElements?.put(newShareName, newShareView)
                //清空回调，避免下次进入时共享元素混乱
                setExitSharedElementCallback(object : SharedElementCallback() {})
            }
        })
    }
{% endhighlight %}

### 其他注意事项
* 当共享元素为图片，且采用 Glide 或 Picasso 加载图片时需禁用加载动画，如 Glide 需调用 `dontAnimate()`：
{% highlight java %}
Glide.with(context)
        .load(url)
        .dontAnimate()
        .into(imageView)
{% endhighlight %}
 
