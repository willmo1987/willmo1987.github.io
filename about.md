---
layout: page
title: 关于我
---

{% highlight java %}
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

#联系我

* Github: [willmo1987](https://github.com/willmo1987)

* Weibo: [那些年这些天](http://weibo.com/cleverwillmo)
 
* Blog: [Will的博客](http://willclub.me)
