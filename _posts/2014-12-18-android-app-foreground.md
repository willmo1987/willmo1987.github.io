---
layout: post
title: Android判断App是否运行在前台
categories: Android
tags: AppStatus
---

> **本博客为个人原创，转载需在明显位置注明出处**

&emsp;&emsp;前几天，公司项目中集成了推送功能，相信大家应该很熟悉，一旦集成了推送功能，那么Android客户端就得按需求弹出相应的Notification，而且当用户点击Notification的时候，也要做出相应的处理。紧接着就出现了一个比较棘手的问题，当用户点击Notification的时候，我们的App有可能正在前台运行，也有可能被用户关掉或者Home键隐藏掉了，这时，我们在处理点击事件的时候不得不做出判断，App到底运行在前台还是在后台呢？看样子貌似很简单，觉得Android SDK应该会提供相应的API供开发者调用，结果是并没有，我们不得不通过其他手段来获取App当前的状态。

&emsp;&emsp;网上对于这个问题讨论最多的有两个版本，一个是获取runningProcess，另一个是获取runningTasks，具体代码如下：

版本1

{% highlight java linenos %}
public static boolean isAppOnForeground(Context context) {
  ActivityManager activityManager = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
  List<ActivityManager.RunningAppProcessInfo> appProcesses = activityManager.getRunningAppProcesses();
  if (appProcesses == null) {
    return false;
  }
  final String packageName = context.getPackageName();
  for (ActivityManager.RunningAppProcessInfo appProcess : appProcesses) {
    if (appProcess.importance == ActivityManager.RunningAppProcessInfo.IMPORTANCE_FOREGROUND && appProcess.processName.equals(packageName)) {
      return true;
    }
  }
  return false;
}
{% endhighlight %}

版本2

{% highlight java linenos %}
public static boolean isAppOnForeground(Context context) {
  String packageName = context.getPackageName();
  ActivityManager activityManager = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
  List<ActivityManager.RunningTaskInfo> tasksInfo = activityManager.getRunningTasks(1);
  if (tasksInfo.size() > 0) {
    if (packageName.equals(tasksInfo.get(0).topActivity.getPackageName())) {
      return true;
    }
  }
  return false;
}
{% endhighlight %}

亲测之后发现版本1是没有用的，版本2可用，但是发现一个问题，版本2中activityManager.getRunningTasks(1)这一句代码是@deprecated，下面是deprecated的原文说明：

	* @deprecated As of {@link android.os.Build.VERSION_CODES#LOLLIPOP}, this method
	* is no longer available to third party
	* applications: the introduction of document-centric recents means
	* it can leak person information to the caller.  For backwards compatibility,
	* it will still return a small subset of its data: at least the caller's
	* own tasks, and possibly some other tasks
	* such as home that are known to not be sensitive.
	
大致意思就是，这个方法在5.0 Lollipop之后就不在对第三方App可用了，因为这样会泄漏用户的信息，为了向后兼容，这个方法只返回一小部分调用者本身的task和其他一些不太敏感的task。

&emsp;&emsp;针对这个问题我没有再用Lollipop系统继续测试，可能这个方法在Lollipop上还是有效的，但是个人觉得这个方法迟早会被抛弃的，所以开始Google其他更好的解决方案。果然，发现了一篇老外的博客[foreground or background](http://steveliles.github.io/is_my_android_app_currently_foreground_or_background.html)，很好的解决了这个问题。仔细阅读之后发现，Android 在4.0之后，Application类中新增了一个名叫[ActivityLifecycleCallbacks](http://developer.android.com/reference/android/app/Application.ActivityLifecycleCallbacks.html)的接口，顾名思义，该接口是Activity生命周期的回调，下面是接口定义：

{% highlight java linenos %}
public interface ActivityLifecycleCallbacks {
  void onActivityCreated(Activity activity, Bundle savedInstanceState);
  void onActivityStarted(Activity activity);
  void onActivityResumed(Activity activity);
  void onActivityPaused(Activity activity);
  void onActivityStopped(Activity activity);
  void onActivitySaveInstanceState(Activity activity, Bundle outState);
  void onActivityDestroyed(Activity activity);
}
{% endhighlight %}

简单说一下原理，当你在Application中注册了你自定义的ActivityLifecycleCallbacks类之后，每访问一个Activity，该Activity的生命周期方法回调的同时，ActivityLifecycleCallbacks中对应的方法也会回调。我们以onActivityResumed和onActivityPaused为例（我觉得使用onActivityStarted和onActivityStopped也同样有效）：

* 当onActivityResumed回调的时候，表示App当前肯定有一个Activity是用户可见的，那么此时App肯定是运行在前台

* 当onActivityPaused回调的时候要分为几种情况了：

    1. Activity的切换，前一个Activity会调用onPause
    2. 用户按Home键隐藏了App
    3. 用户一直back键退出了应用

    后两种情况肯定是App进入了后台，第一种情况则不是，此时我们需要在onActivityPaused的实现中调用用handler.postDelayed一个runnable去检查onActivityResumed是否再次回调即可，若回调了，表示App依然运行在前台，若没有，表示App已经进入后台。handler的delay设置为500ms或者1s都可以，这里要着重说明一下，这个延时值的设置不一定对每个Android手机都有效，对于卡顿比较严重的手机，时间可能要更长一些，但是对于大部分手机，500ms到1s还是比较合适的。
    
&emsp;&emsp;讲解了这么多，相信大家应该理解了大致的思路了，下面是我参考上述老外的博客[foreground or background](http://steveliles.github.io/is_my_android_app_currently_foreground_or_background.html)修改以及精简了之后的部分实现，供大家参考。希望大家还是读一读英文原文的博客，老外给的实现功能还不仅仅是判断App在前台还是在后台，当App进入前台或者后台的时候，还有相应的回调方法，有类似需求的同学更要仔细研读一下。

{% highlight java linenos %}
public boolean isForeground(){
  return foreground;
}
 
public boolean isBackground(){
  return !foreground;
}
 
@Override
public void onActivityResumed(Activity activity) {
  paused = false;
  foreground = true;
  if (check != null) {
    handler.removeCallbacks(check);
  }
}
 
@Override
public void onActivityPaused(Activity activity) {
  paused = true;
  if (check != null) {
    handler.removeCallbacks(check);
  }
  handler.postDelayed(check = new Runnable(){
    @Override
    public void run() {
      if (foreground && paused) {
        foreground = false;
        Trace.d("App went background");
      }
      else {
        Trace.d("App still foreground");
      }
    }
  }, CHECK_DELAY);
}
{% endhighlight %}
