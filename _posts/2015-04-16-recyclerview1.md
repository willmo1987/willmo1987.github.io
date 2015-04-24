---
layout: post
title: "Material Design系列之RecyclerView（一）"
tags: [RecyclerView]
categories: [Android]
---

> **本博客为个人原创，转载需在明显位置注明出处**

&emsp;&emsp;今天给大家介绍Material Design系列中的RecyclerView，由于内容比较多，我将会准备三篇文章分享给大家。我们日常开发当中碰到列表这样的需求大家肯定会想到用ListView来实现，用法也很简单，但是ListView功能比较单一，局限，只能实现Vertical的列表。为了方便开发者开发，Google在Lollipop中添加了RecyclerView，它可以看作是升级版的ListView，无论Vertical的还是Horizontal的，或是Grid的都可以轻松利用RecyclerView来实现。本篇文章所介绍的内容是**利用RecyvlerView实现简单列表和多type的列表**。

&emsp;&emsp;先看看你需要依赖哪些Library：

{% highlight groovy linenos %}
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:22.0.0'
    compile 'com.android.support:recyclerview-v7:22.0.0'
}
{% endhighlight %}

&emsp;&emsp;如果你的support包很久没有更新了，建议更新一下再使用。下面是layout文件：

{% highlight xml linenos %}
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:minHeight="?attr/actionBarSize"
        android:background="?attr/colorPrimary"/>
    <android.support.v7.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_below="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:overScrollMode="never"
        android:fadingEage="none"/>
</RelativeLayout>
{% endhighlight %}

&emsp;&emsp;比较简单，跟ListView的配置差不多，**但是这里要注意，RecyclerView没有divider和listSelector这两个属性**。下面直接看代码：

{% highlight java linenos %}
private void setupRecyclerView() {
  recyclerView = (RecyclerView) findViewById(R.id.recyclerView);
  recyclerView.setHasFixedSize(true);
  recyclerView.setLayoutManager(new LinearLayoutManager(this, LinearLayoutManager.VERTICAL, false));
  recyclerView.setAdapter(new BaseListAdapter());
}
{% endhighlight %}

&emsp;&emsp;如果这是个静态列表，那么可以调用setHasFixedSize(true)。接着跟ListView不一样的来了，setLayoutManager()方法，这个方法表示给RecyclerView指定一个布局类型，我当前设置的是LinearLayout并且是Vertical的类型，也就是说当前的设置跟ListView的效果是一样的。除此之外，你还可以调用这个方法将RecyclerView设置成其他类型的布局，比如Horizontal的LinearLayout或者GridLayout等，这就是我为什么说RecyclerView强大的地方。如果你不太明白LayoutManager是什么情况，没关系，下一篇博客我会详细讲解。最后咱们说说Adapter和ViewHolder，这里的Adapter不再继承自BaseAdapter了，ViewHolder也不再是自定义的独立的类了，它们都要继承子RecyclerView的Adapter和ViewHolder内部类。先看看代码吧：

{% highlight java linenos %}
private class ViewHolder extends RecyclerView.ViewHolder {

  private TextView textView;

  public ViewHolder(View itemView) {
    super(itemView);
    textView = (TextView) itemView;
    textView.setOnClickListener(new View.OnClickListener() {
      @Override
      public void onClick(View v) {
        int layoutPosition = getLayoutPosition();
        onItemClick(contentArray[layoutPosition], layoutPosition);
      }
    });
  }
}
{% endhighlight %}

&emsp;&emsp;关于ViewHolder可以这样解释：RecyclerView.ViewHolder = ViewHolder + convertView，也就是说这里的ViewHolder是将我们在ListView中使用的ViewHolder和convertView的合体，有点奇怪是吧，不着急，咱们继续往下看Adapter的实现：

{% highlight java linenos %}
private class BaseListAdapter extends RecyclerView.Adapter<ViewHolder> {

  private LayoutInflater layoutInflater;

  public BaseListAdapter() {
    layoutInflater = LayoutInflater.from(BaseListActivity.this);
  }

  @Override
  public int getItemCount() {
    if (ArrayUtil.isValid(contentArray)) {
      return contentArray.length;
    }
    return 0;
  }

  @Override
  public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
    return new ViewHolder(layoutInflater.inflate(R.layout.list_item_main, parent, false));
  }

  @Override
  public void onBindViewHolder(ViewHolder holder, int position) {
    holder.textView.setText(contentArray[position]);
  }

}
{% endhighlight %}

&emsp;&emsp;看完Adapter的实现还是有点费解是吧，肯定的，我刚开始学习的时候也觉得别扭，但是仔细研究一下就明白了，onCreateViewHolder() + onBindViewHolder() = getView()。咱们先回顾一下getView的实现吧：

{% highlight java linenos %}
@override
public View getView(View convertView, int position) {
  if(convertView == null) {
    //instaniate convertView and ViewHolder
  }
  else {
    //get ViewHolder from convertView
  }
  //set value
}
{% endhighlight %}

&emsp;&emsp;结合getView的实现以及翻看RecyclerView的部分源码发现，**其实RecyclerView.Adapter中已经帮助我们实现了convertView的优化复用，然后将实例化convertView和set value拆分成两个抽象方法(onCreateViewHolder()和onBindViewHolder())在子类来实现**。相信这么解释大家应该都能明白这些方法各自是什么含义了吧，下面给一张效果图：

![md_recyclerview_normal](/images/md_recyclerview_normal.png)

&emsp;&emsp;另外，如果这个列表有两种或多种布局的情况，在ListView中我们是通过itemType来区分实现的。在RecyclerView中也差不多，下面是部分Adapter的实现，大家可以参考一下，比较容易理解我就不多解释了：

{% highlight java linenos %}
@Override
public int getItemViewType(int position) {
  return position % 2 == 0 ? TYPE_LEFT : TYPE_RIGHT;
}

@Override
public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
  if (viewType == TYPE_LEFT) {
    return new LeftViewHolder(layoutInflater.inflate(R.layout.list_item_left, parent, false));
  }
  else {
    return new RightViewHolder(layoutInflater.inflate(R.layout.list_item_right, parent, false));
  }
}

@Override
public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
  if (holder.getItemViewType() == TYPE_LEFT) {
    LeftViewHolder leftHolder = (LeftViewHolder) holder;
    leftHolder.leftView.setText(contentArray[position]);
  }
  else {
    RightViewHolder rightHolder = (RightViewHolder) holder;
    rightHolder.rightView.setText(contentArray[position]);
  }
}
{% endhighlight %}

还有一张效果图：

![md_recyclerview_multitype](/images/md_recyclerview_multitype.png)

这就是RecyclerView的第一部分内容，水平有限，能力一般，欢迎大家斧正并恳请大家继续关注Material Design系列的博客，近期我会持续更新。

**UPDATE：**RecyclerView中没有现成的方法让我们添加HeaderView和FooterView，但是了解了MultiType的实现，我们可以轻松利用这种方式搞定，自己动手实现一下呗！！

**PS：关于效果图中的阴影效果，其实是两个View的新属性android:elevation和android:translationZ，只有在Lollipop以上的系统中才有效并且该View一定要设置background，不然阴影不会显示。具体代码大家可以参考我Github上的[LearnMaterialDesign](https://github.com/willmo1987/LearnMaterialDesign)。**
