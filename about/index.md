---
title: 关于
layout: page
comments: no
---

{{ site.about }}

{% highlight java linenos %}
public class Will {

  String fullName = "Will Mo";

  String company = "南京小鸡快跑网络科技有限公司";

  String[] tags = {"Android developer", "Introverted", "Love gym"};
    
  String weibo = "@那些年这些天";
    
  String github = "https://github.com/willmo1987";

  String blog = "http://willclub.me";

  private static Will will = null;
  
  private Will() {
  }

  public static Will getInstance() {
    if (will == null) {
      will = new Will();
    }
    return will;
  }

  public static void main(String[] args) {
    while(true) {
      Will.getInstance().life();
    }
  }

  public void life() {
    if (working time) {
      think
      discuss
      programme
    }
    else {
      write blog
      gym
      watch movie
      read book
    }
  }

}
{% endhighlight %}

----

###联系方式：

网站：[{{ site.name }}]({{ site.url }})

邮箱：[{{ site.email }}](mailto:{{ site.email }})

GitHub : [http://github.com/{{ site.github }}](http://github.com/{{ site.github }})

----


[![新浪微博](http://service.t.sina.com.cn/widget/qmd/{{ site.weibo }}/4f9db57b/7.png)](http://weibo.com/u/{{ site.weibo }})
