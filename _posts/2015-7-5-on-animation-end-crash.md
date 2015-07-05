---
layout: post
title: "你不知道的onAnimationEnd的坑"
tags: [animation]
categories: [Android]
---

> **本博客为个人原创，转载需在明显位置注明出处**

&emsp;&emsp;最近公司项目里面需要做一些动画效果，本以为会非常顺利并且愉快的完成，但还是发现了一个之前没有碰到过的问题，这里分享给大家，大家以后在做动画的时候多多注意。

&emsp;&emsp;写过动画的应该都用过AnimationListener，其中有一个回调方法是onAnimationEnd()，我项目中实现的动画流程是这样的：首先add一个view到ViewGroup中，然后执行动画，当动画结束时再remove一个view，所以我需要在onAnimationEnd中执行removeView操作。

&emsp;&emsp;在动画实现的过程中，我一直都是用4.3+的系统测试的，没有发现任何问题。但是在提测之前，为了保险起见，我用4.0模拟器和2.x的模拟器又测试了一遍，这时问题来了，动画执行完成的时候，出现了crash，log也非常诡异：

![crash_log](/images/on_animation_end_crash.png)

只知道是ViewGroup的dispatchDraw方法中出现了空指针，没有定位到项目里面的任何代码，于是我仔细检查我动画的实现过程，并翻看源码，希望定位问题，然而并没有什么发现，最后直接Google帮忙，找到了一个[解决方案](http://stackoverflow.com/questions/5569267/nullpointerexception-that-doesnt-point-to-any-line-in-my-code)。意思是说**在onAnimationEnd方法中改变View的层级结构的话，系统会抛出这样的异常**。亲测结果是2.x和4.0的系统会出现这样的问题，貌似4.1+开始，Google解决了这个bug。解决方案非常简单，**只需要用handler post一个runnable去执行改变View层级结构的操作**，例如：

{% highlight java linenos %}
@Override
public void onAnimationEnd(Animation animation) {
    new Handler().post(new Runnable() {
        public void run() {
            myLayout.removeView(simulateMovingImg);
        }
    });
}
{% endhighlight %}

我也是第一次碰到这样的问题，大家引以为戒！