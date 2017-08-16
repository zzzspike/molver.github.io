---
title: "使用 ItemTouchHelper 实现 RecyclerView 滑动删除和长按拖放"
date:   2017-8-16 16:06:00 +0800
categories: 编程
tags: android
---

## 前言

有很多教程或第三方库可以实现 RecyclerView 的滑动删除和长按拖放功能，可能利用了 `GestureDetector` 或 `View.OnDragListener` 去实现。
但其实要实现这个功能只需要一个类，且已经在 Android Support Library 中了-- 那就是 [`ItemTouchHelper`](https://developer.android.com/reference/android/support/v7/widget/helper/ItemTouchHelper.html)
<!--more-->
[](){: #more }

## 设置
在 build.gradle 中添加:
{% highlight groovy %}
compile 'com.android.support:recyclerview-v7:26.0.0-alpha1'
{% endhighlight %}

## ItemTouchHelper 和 ItemTouchHelper.Callback 的使用
要使用 `ItemTouchHelp` 需要 `ItemTouchHelper.Callback` 依赖，Callback 是一个抽象类，需要通过继承实现。它可以监听 `move`、 `swip`，或对 ItemTouchHelper 的一些行为进行设置。

### 实现 ItemTouchHelper.Callback
要实现拖放和滑动删除功能，主要需实现以下几个回调
{% highlight java %}
    public int getMovementFlags(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder) //控制滑动删除和拖放的方向
    public boolean onMove(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder, RecyclerView.ViewHolder target) //拖放后回调
    public void onSwiped(RecyclerView.ViewHolder viewHolder, int direction) //滑动完成后回调
    public boolean isLongPressDragEnabled() //控制是否启用长按拖放
    public boolean isItemViewSwipeEnabled() //控制是否启用滑动删除
{% endhighlight java %}
下面我们来一一实现<br>
`getMovementFlags(RecyclerView, ViewHolder)`方法主要用来设置上下左右各方向是用来拖动还是滑动，用 `ItemTouchHelper.makeMovementFlag(int, int)`要构建返回的设置。下面设置了在线性布局时，上下方向为拖放，左右方向为滑动，在网络布局时，则上下左右方向均为拖放。
{% highlight java %}
public int getMovementFlags(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder) {
    RecyclerView.LayoutManager layoutManager = recyclerView.getLayoutManager();
    int dragFlag = 0;
    int swipeFlag = 0;
    if (layoutManager instanceof GridLayoutManager) {
        dragFlag = ItemTouchHelper.DOWN | ItemTouchHelper.UP
                | ItemTouchHelper.START | ItemTouchHelper.END; //网格布局的，则上下左右均为拖放
    } else if (layoutManager instanceof LinearLayoutManager) {
        dragFlag = ItemTouchHelper.DOWN | ItemTouchHelper.UP; //设置上下方向为拖放
        swipeFlag = ItemTouchHelper.START | ItemTouchHelper.END; //设置左右方向为滑动删除
    }
    return makeMovementFlags(dragFlag, swipeFlag);
}
{% endhighlight %}
`isLongPressDragEnabled()` 返回 true 启用长按拖放，或直接调用 `ItemTouchHelper.startDrag(RecyclerView.ViewHolder viewHolder)` 手动开始拖放<br>
`onMove` 和 `onSwiped` 需要用来通知 Adapter 拖放或滑动结束了，进行对数据更新。我们先创建一个接口 ItemTouchHelperAdapter 作为监听器用来传递通知给 Adapter
{% highlight java %}
public interface ItemTouchHelperAdapter {
    void onItemDismiss(int position);
    void onItemMove(int fromPosition, int toPosition);
}
{% endhighlight %}
将监听器在构造函数中初始化
{% highlight java %}
public class MyItemTouchHelperCallback {

    private final ItemTouchHelperAdapter adapter;

    public MyItemTouchHelperCallback(ItemTouchHelperAdapter adapter) {
        this.adapter = adapter;
    }

    @Override
    public boolean onMove(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder, RecyclerView.ViewHolder target) {
        adapter.onItemMove(viewHolder.getAdapterPosition(), target.getAdapterPosition());
        return true;
    }

    @Override
    public void onSwiped(RecyclerView.ViewHolder viewHolder, int direction) {
        adapter.onItemDismiss(viewHolder.getAdapterPosition());
    }
    ...
}
{% endhighlight %}
`MyRecyclerViewAdapter` 实现这个接口，接收到条目位置变化或被移除的回调后更新数据
{% highlight java %}
@Override
public class MyRecyclerViewAdapter {

    ...

    public void onItemDismiss(int position) {
        items.remove(position);
        notifyItemRemoved(position);
    }

    @Override
    public void onItemMove(int fromPosition, int toPosition) {
        if (fromPosition < toPosition) {
            for (int i = fromPosition; i < toPosition; i++) {
                Collections.swap(items, i, i + 1);
            }
        } else {
            for (int i = fromPosition; i > toPosition; i--) {
                Collections.swap(items, i, i - 1);
            }
        }
        notifyItemMoved(fromPosition, toPosition);
    }

    ...
}
{% endhighlight %}
### 实例化 ItemTouchHelper
`Callback` 搞定，我们就可以实例化 `ItemTouchHelper` 并调用 `attachToRecyclerView(RecyclerView rv) 关联相应的 `RecyclerView`
{% highlight java %}
MyItemTouchHelperCallback callback = new MyItemTouchHelperCallback(adapter);
ItemTouchHelper touchHelper = new ItemTouchHelper(callback);
touchHelper.attachToRecyclerView(recyclerView);
{% endhighlight %}

### 其他一些细节
* ItemTouchHelper 的 startDrag(RecyclerView.ViewHolder viewHolder) 方法，用来手动启动拖放，可以实现触摸拖放手柄View直接开始拖放
* Callback 里的其他控制选项：OnSelectedChanged(ViewHolder, int) 在条目进入拖放或滑动状态时会回调，onClearView(RecyclerView, ViewHolder)在条目被放下，滑动取消或成功恢复为正常状态时会回调。
这里可以处理一些细节，例如进入拖放或滑动模式时，修改View背景图为绿色。

## 效果图及Demo地址
![](/assets/images/itemtouchhelper1.gif)![](/assets/images/itemtouchhelper2.gif)
[Demo地址](https://github.com/molver/ItemTouchHelperDemo)

