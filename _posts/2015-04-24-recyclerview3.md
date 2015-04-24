---
layout: post
title: "Material Design系列之RecyclerView（三）"
tags: [RecyclerView]
categories: [Android]
---

> **本博客为个人原创，转载需在明显位置注明出处**

&emsp;&emsp;今天给大家介绍RecyclerView的最后一部分内容divider和listSelector。之前和大家提过，RecyclerView不像ListView，没有诸如listSelector和divider的属性，那么我们怎么在RecyclerView中实现呢？

###divider
    
&emsp;&emsp;想在RecyclerView上实现divider的效果相对ListView上要麻烦很多，需要我们继承抽象类[ItemDecoration](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.ItemDecoration.html)，在相应的抽象方法中实现。ItemDecoration，顾名思义，就是item上面的装饰，咱们先从使用上开始说，RecyclerView中有两个方法：
    
{% highlight java linenos %}
public void addItemDecoration(ItemDecoration decor)

public void addItemDecoration(ItemDecoration decor, int index)
{% endhighlight %}

再看一下RecyclerView的源码：

{% highlight java linenos %}
@Override
public void draw(Canvas c) {
  super.draw(c);
  
  final int count = mItemDecorations.size();
  for (int i = 0; i < count; i++) {
    mItemDecorations.get(i).onDrawOver(c, this, mState);
  }
  
  ...
}

@Override
public void onDraw(Canvas c) {
  super.onDraw(c);

  final int count = mItemDecorations.size();
  for (int i = 0; i < count; i++) {
    mItemDecorations.get(i).onDraw(c, this, mState);
  }
}
{% endhighlight %}

应该不难明白吧，RecyclerView中有一个ArrayList的itemDecorations，也就是说可以给每个item添加若干个Decoration，今天就以divider为例来给大家讲讲ItemDecoration的使用。细心的朋友应该已经发现了，上述RecyclerView源码在draw和onDraw方法中分别调用了ItemDecoration的onDrawOver和onDraw方法，下面就来解释一下这两个方法的区别：

 {% highlight java linenos %}
 /*
 * 在itemView绘制完成之前调用，也就是说此方法draw出来的效果将会在itemView的下面
 */
public void onDraw(Canvas c, RecyclerView parent, RecyclerView.State state)
{% endhighlight %}

 {% highlight java linenos %}
 /*
 * 与onDraw相反，draw出来的效果将叠加在itemView的上面
 */
public void onDrawOver(Canvas c, RecyclerView parent, RecyclerView.State state)
{% endhighlight %}
   
 为了实现divider的效果，我们需要在onDrawOver中draw出来，但是在draw之前，我们还需要了解另外一个ItemDecoration的方法：
 
 {% highlight java linenos %}
  /*
 * 计算通过配置outRect来设置itemView的inset边界，相当于设置itemView的margin
 */
public void getItemOffsets(Rect outRect, View view, RecyclerView parent, RecyclerView.State state)
{% endhighlight %}

好了，该介绍的概念介绍完了，下面就来讲讲具体的实现吧。首先我们自定义一个DividerItemDecoration类继承自ItemDecoration，并且重写getItemOffsets和onDRawOver方法。在构造方法中传入一个dividerDrawable用来进行绘制，这个大家可以用.9图片或者用shape去画一个，这里就不再多说了。

 {% highlight java linenos %}
 /**
 * RecyclerView中实现divider功能
 * 只支持LinearLayoutManager
 */
public class DividerItemDecoration extends RecyclerView.ItemDecoration {

  private Drawable dividerDrawable;

  public DividerItemDecoration(Drawable divider) {
    dividerDrawable = divider;
  }
  
  ...
}
{% endhighlight %}

大家应该知道divider是每个itemView之间的分割线，但是第一个item应该除外，所以getItemOffsets中的实现应该是这样：

 {% highlight java linenos %}
@Override
    public void getItemOffsets(Rect outRect, View view, RecyclerView parent, RecyclerView.State state) {
  if (dividerDrawable == null) {
    return;
  }

  //如果是第一个item，不需要divider，所以直接return
  if (parent.getChildLayoutPosition(view) < 1) {
    return;
  }

  //相当于给itemView设置margin，给divider预留空间
  int layoutOrientation = getOrientation(parent);
  if (layoutOrientation == LinearLayoutManager.VERTICAL) {
    outRect.top = dividerDrawable.getIntrinsicHeight();
  } else if(layoutOrientation == LinearLayoutManager.HORIZONTAL) {
    outRect.left = dividerDrawable.getIntrinsicWidth();
  }
}
{% endhighlight %}

大家看我方法内的注释应该能看懂个七七八八了，这边再针对最后一部分详细解释一下，divider的绘制是从下标为1开始的itemView上进行绘制，针对Vertical的layout，我们需要给每一个itemView的上面预留dividerDrawable.getIntrinsicHeight()的空间，以便下面的绘制不会覆盖itemView的任何部分，所以设置了outRect.top ＝ dividerDrawable.getIntrinsicHeight()。同理可分析出Horizontal的layout应该设置outRect.left。

接下来就是divider的绘制了：

 {% highlight java linenos %}
@Override
public void onDrawOver(Canvas c, RecyclerView parent, RecyclerView.State state) {
  LinearLayoutManager layoutManager = getLinearLayoutManger(parent);
  if (layoutManager == null) {
    return;
  }

  int firstVisiblePosition = layoutManager.findFirstVisibleItemPosition();
  int orientation = getOrientation(layoutManager);
  int childCount = parent.getChildCount();
  if (orientation == LinearLayoutManager.VERTICAL) {
    int right = parent.getWidth();
    for (int i=0; i<childCount; i++) {
      //判断第一个item的下标是不是0，是则return，不需要draw divider
      if (i == 0 && firstVisiblePosition == 0) {
        continue;
      }
      View childView = parent.getChildAt(i);
      RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) childView.getLayoutParams();
      int left = parent.getPaddingLeft() + childView.getPaddingLeft();
      int bottom = childView.getTop() - params.topMargin;
      int top = bottom - dividerDrawable.getIntrinsicHeight();
      dividerDrawable.setBounds(left, top, right, bottom);
      dividerDrawable.draw(c);
    }
  } else if(orientation == LinearLayoutManager.HORIZONTAL) {
    ...
  }
}
{% endhighlight %}

具体的计算思路我就不多介绍了，就是算出drawable的bounds(left, top, right, bottom)，不算复杂，大家只要草稿纸上画个草图，仔细看一看，算一算应该可以理解的。到了这里divider的绘制已经成功了，下面来看一下效果图吧：

![md_rv_divider_decoration](/images/md_rv_divider_decoration.png)

###listSelector

listSelector的实现较divider来说要简单一些，就是在itemView的rootLayout中自定义selector作为background，例如：

 {% highlight xml linenos %}
 android:background="@drawable/selector_rect_item"
 {% endhighlight %}
 
 至于selector怎么配置就不需要我再多介绍了吧。但是从上述divider的实现来看，我们也可以用ItemDecoration来实现listSelector的效果，不过我个人不推荐大家这么做，太麻烦，而且效率不一定比自定义的selector要好。

好了，RecyclerView的基本内容到这里已经介绍完毕，以后如果有一些使用上的心得或者某些进阶类的使用我再给大家分享。

**PS：关于上述DividerItemDecoration的实现，我参考了[fatfingers](https://gist.github.com/fatfingers)分享在Gist上的[DividerItemDecoration](https://gist.github.com/fatfingers/233abbae200b5e87297b)。另，所有Material Design的相关代码均可参考我Github上的[LearnMaterialDesign](https://github.com/willmo1987/LearnMaterialDesign)。**