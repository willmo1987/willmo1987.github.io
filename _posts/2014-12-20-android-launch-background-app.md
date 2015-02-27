---
layout: post
title: Android启动运行在后台的App
categories: Android
tags: tip
---

&emsp;&emsp;接上一篇博客[Android判断App是否运行在前台]({{ page.previous.url }})，当用户点击Notification时，我们需要判断此时App运行在前台还是后台，如果App运行在前台，我们一般处理是直接跳转到相应的通知detail界面或者干脆clear掉Notification，App内未读bage的显示会引导用户查看未读消息；如果App运行在后台，大多数情况会直接跳转到Notification的detail界面，方便用户查看。

&emsp;&emsp;针对App在后台的情况，公司产品经理给出的需求是clear掉Noification，启动App即可。一开始以为这样实现起来简单太多，只需要new一个Intent然后startActivity就好了，分分钟搞定的事情，事实上并没有，这个问题耗费了我大概两个小时去搜解决方案和测试，最后终于搞定。事实上它确实很简单，只需要按AndroidManifest.xml中入口Activity的配置来实现就可以了。

一般情况下，我们的入口Activity定义为SplashActivity，如下：

{% highlight xml linenos %}
<activity
	android:name=".SplashActivity"
	android:label="@string/app_name" 
	android:screenOrientation="portrait">
	<intent-filter>
		<action android:name="android.intent.action.MAIN" />
		<category android:name="android.intent.category.LAUNCHER" />
	</intent-filter>
</activity>
{% endhighlight %}

所以当我们要启动App的时候，应该这样实现：

{% highlight java linenos %}
private void startApp(Context context) {
  Intent intent = new Intent(context, SplashActivity.class);
  intent.setAction(Intent.ACTION_MAIN);
  intent.addCategory(Intent.CATEGORY_LAUNCHER);
  intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);  //从Notification启动，必须添加此flag
  context.startActivity(intent);
}
{% endhighlight %}

确实很简单吧，这个问题让我明白，工作当中对待任何问题都不能小看或轻视，为人处事亦如此，与大家共勉！