---
layout: post
title: Material Design系列之RecyclerView（二）
categories: Android
tags: RecyclerView
---

> **本博客为个人原创，转载需在明显位置注明出处**

&emsp;&emsp;[上一篇博客](http://willclub.me/recyclerview1/)给大家介绍了RecyclerView的基本使用，相信大家应该对RecyclerView有了一个初步的认识，今天继续给大家分享一下LayoutManager以及ItemAnimator的相关知识。

###LayoutManager

&emsp;&emsp;首先来看看LayoutManager，[上一篇博客](http://willclub.me/recyclerview1/)中跟大家简单提到了LinearLayoutManager，不知道大家对于LayoutManager的作用是否完全理解了，这里再着重讲解一下，先看一张图：

![md_rv_structure](/images/md_rv_structure.png)

1. RecyclerView是一个ViewGroup，是承载所有itemView的载体

2. LayoutManager可以看作是一个控制器，它可以控制所有itemView的排列和展现（线性、格子等）

3. ListView ＝ RecyclerView + Vertical的LinearLayoutManager

这么解释大家应该都能明白了吧，为什么说RecyclerView是升级版的ListView，**因为RecyclerView其实是将ListView的职能细分开了，提取出LayoutManager来控制itemView的排列展现，变得更灵活，可扩展性更好**。SDK中默认提供了几种LayoutManager，如果有特殊情况，你还可以轻松定制CustomLayoutManager，满足相应的产品需求。下面介绍一下SDK中的LayoutManager：

1. **LinearLayoutManager**

    线性布局，LayoutManager的实现类，类型包括Vertical和Horizontal

2. **GridLayoutManager**

    格子布局，继承自LinearLayoutManager，实现效果类似GridView

3. **StaggeredGridLayoutManager**

    交错的格子布局，同样也是LayoutManager的实现类，类型包括Vertical和Horizontal，与GridLayoutManager很相似，不过是交错的格子，也就是宽高不等的格子视图，类似瀑布流的效果

概念介绍完了，接着按顺序给几张LayoutManager的效果图吧：

![md_multilayout_linearlayout](/images/md_multilayout_linearlayout.png)

![md_multilayout_gridlayout](/images/md_multilayout_gridlayout.png)

![md_multilayout_staggeredgrid](/images/md_multilayout_staggeredgrid.png)

###ItemAnimator

&emsp;&emsp;除了LayoutManager之外，RecyclerView较ListView的改进还包括ItemAnimator，顾名思义，就是itemView的动画，先来看一下动画效果：

![md_rv_animated](/images/md_rv_animated.gif)

非常smooth吧，用法也非常简单，看代码：

{% highlight java linenos %}
//add
contentList.add(position, newContent);
adapter.notifyItemInserted(position);

//delete
contentList.remove(position);
adapter.notifyItemRemoved(position);
{% endhighlight %}

&emsp;&emsp;上述代码可以理解为notifyDataSetChanged + animation，是不是很方便呢？这里我只是以单个item的add和delete为例，RecyclerView.Adapter中还有其他几种方法，大家最好一个一个试，看看每种方法的区别，下面列举一下notify系列的所有方法为：

{% highlight java linenos %}
notifyDataSetChanged()

notifyItemChanged(int position)

notifyItemInserted(int position)

notifyItemMoved(int fromPosition, int toPosition)

notifyItemRemoved(int position)

//range系列，多个item的操作
notifyItemRangeChanged(int positionStart, int itemCount)

notifyItemRangeInserted(int positionStart, int itemCount)

notifyItemRangeRemoved(int positionStart, int itemCount)
{% endhighlight %}

&emsp;&emsp;所有这些itemView的动画都是通过ItemAnimator实现的，可以通过recyclerView.setItemAnimator()方法设置，默认情况下RecyclerView中运用了DefaultItemAnimator，所以我们可以轻松实现。如果你想自定义ItemView的动画，你可以继承ItemAnimator来自定义CustomItemAnimator，达到你想要的效果。除此之外，一位日本朋友在Github上的开源了[recyclerview-animators](https://github.com/wasabeef/recyclerview-animators)，stars是1000+的，大家可以学习学习，看看别人是怎么实现这些动画的。

&emsp;&emsp;**本博客所有的源码都在[LearnMaterialDesign](https://github.com/willmo1987/LearnMaterialDesign)上，水平有限，能力一般，还请大家斧正，下一篇博客再见**。




