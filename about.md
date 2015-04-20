---
layout: page
title: About
---

> **作为一名优秀的Android程序员，就应该以我最擅长的方式介绍自己**

{% highlight java %}
public class Will {

  String fullName = "Will Mo";

  String company = "京东金融";

  String[] tags = {"Android developer", "Introverted", "Love gym"};
    
  String QQ = "3204461186";
    
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