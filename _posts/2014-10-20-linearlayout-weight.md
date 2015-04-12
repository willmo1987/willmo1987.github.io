---
layout: post
title: LinearLayout中layout_weight的计算
categories: Android
tags: layout
---

&emsp;&emsp;**本博客为个人原创，转载需在明显位置注明出处：http://willclub.me/linearlayout-weight/**

&emsp;&emsp;在我们项目开发中，经常会用到LinearLayout + layout_weight这样的组合，印象比较深的应该是用Eclipse开发的时候，当你将某个View控件layout_weight设为1之后，lint总是警告你需要讲layout_width设为0dp，但是好像设为match_parent也没有问题，结果是一样的，另外子View设置layout_weight之后需要LinearLayout设置weightSum嘛，如果设置了，weightSum != layout_weight之和会怎么样，今天就跟大家探讨一下这个问题。

&emsp;&emsp;下面是一个很简单的layout.xml，RelativeLayout中垂直居中显示一个LinearLayout，LinearLayout中水平显示三个不同颜色的View，当前设置是每个View的layout_width = 0dp，layout_weight = 1：

{% highlight xml linenos %}
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_centerVertical="true"
        android:orientation="horizontal">
        <View
            android:layout_width="0dp"
            android:layout_height="50dp"
            android:layout_weight="1"
            android:background="@color/orange"/>
        <View
            android:layout_width="0dp"
            android:layout_height="50dp"
            android:layout_weight="1"
            android:background="@color/blue"/>
        <View
            android:layout_width="0dp"
            android:layout_height="50dp"
            android:layout_weight="1"
            android:background="@color/dark"/>
    </LinearLayout>
</RelativeLayout>
{% endhighlight %}

然后我分别给出五种不同设置，来看看这几种设置结果如何：

1. 如上述代码，每个View的layout_width = 0dp，layout_weight = 1

    ![NO.1](/images/layout_weight-s01.png)

2. 每个View的layout_width = match_parent，layout_weight = 1

    ![NO.2](/images/layout_weight-s02.png)

3. layout_weight = 1，前两个View的layout_width = match_parent，最后一个View的layout_width = 0dp

    ![NO.3](/images/layout_weight-s03.png)

4. 外层LinearLayout的weightSum = 3，每个View的layout_width = 0dp，layout_weight = 1

    ![NO.4](/images/layout_weight-s04.png) 

5. 外层LinearLayout的weightSum = 5，每个View的layout_width = 0dp，layout_weight = 1

    ![NO.5](/images/layout_weight-s05.png)
    
&emsp;&emsp;有点费解吧，有的一样，有的不一样，有的还很怪，为什么呢？我们一起来看看LinearLayout中onMeasure()的源码，由于源码太多，我摘了最重要的一部分，其他用省略号代替：

{% highlight java linenos %}
...
int delta = widthSize - mTotalLength;
if (skippedMeasure || delta != 0 && totalWeight > 0.0f) {
  float weightSum = mWeightSum > 0.0f ? mWeightSum : totalWeight;
  ...
  for (int i = 0; i < count; ++i) {
    final View child = getVirtualChildAt(i);
    if (child == null || child.getVisibility() == View.GONE) {
      continue;
    }
    final LinearLayout.LayoutParams lp = (LinearLayout.LayoutParams) child.getLayoutParams();
    float childExtra = lp.weight;
    if (childExtra > 0) {
      // Child said it could absorb extra space -- give him his share
      int share = (int) (childExtra * delta / weightSum);
      weightSum -= childExtra;
      delta -= share;
      ...
    }   
  }
}
...
{% endhighlight %}

仔细分析代码，我们可以看出，LinearLayout在计算weight的时候是用剩余空间来计算的，用数学公式表达应该是：

![weight计算](/images/linearlayout_weight_cal.png)

按这个思路，我们将上述每个情况各计算一次，为了方便理解，LinearLayout总宽度我们用MATCH_PARENT表示，计算出的childView的宽度的结果我们也将是MATCH_PARENT的分数表示：

1. 每个View的layout_width = 0dp，layout_weight = 1

    ![childWidth-f01](/images/linearlayout_weight_cal_f01.png)
    
    计算过后，每个子View的宽度均为三分之一的MATCH_PARENT，结果得证
    
2. 每个View的layout_width = match_parent，layout_weight = 1

    ![childWidth-f02](/images/linearlayout_weight_cal_f02.png)
 
    计算过后，每个子View的宽度均为三分之一的MATCH_PARENT，结果得证
    
3. layout_weight = 1，前两个View的layout_width = match_parent，最后一个View的layout_width = 0dp

    ![childWidth-f03](/images/linearlayout_weight_cal_f03.png)
    
    结果，childView1和childView2的宽度均为三分之二的MATCH_PARENT，childView3的宽度为负的三分之一的MATCH_PARENT，由于总宽度只有MATCH_PARENT，所以childView2只显示了一半，childView3不显示，结果得证。
    
4. 外层LinearLayout的weightSum = 3，每个View的layout_width = 0dp，layout_weight = 1

    ![childWidth-f04](/images/linearlayout_weight_cal_f04.png)
    
    根据上述LinearLayout的源码分析，如果LinearLayout给定了weightSum，按weightSum计算，否则，按totalWeight计算，这里LinearLayout指定了weightSum，所以计算的时候按weightSum计算。结果显示与1、2相同，均为三分之一的MATCH_PARENT，结果得证。
    
5. 外层LinearLayout的weightSum = 5，每个View的layout_width = 0dp，layout_weight = 1

    ![childWidth-f05](/images/linearlayout_weight_cal_f05.png)
    
    结果为每个childView的宽度均为五分之一的MATCH_PARENT，结果得证。
    
&emsp;&emsp;分析完了，相信这五种情况应该足够解决大家的疑惑了吧。细心的同学可能会发现，上述源码中的算法跟我下面给出的公式有些不一样，源码中每计算一次childWidth，无论是totalWeight还是剩余空间delta都将当前计算好的childView的weight和width减掉，然后再进行下一轮计算。其实源码跟我的计算方法是一样的，数学公式演算一下就是：

![childWidth-f06](/images/linearlayout_weight_cal_f06.png)

依次类推。

&emsp;&emsp;好了，关于LinearLayout的layout_weight计算就介绍这么多，时间有限，难免出错，还请大家斧正。
 





