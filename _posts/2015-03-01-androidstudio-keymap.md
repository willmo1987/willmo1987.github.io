---
layout: post
title: Android Studio设置Eclipse风格快捷键
categories: Android
tags: tools
---

<br>
&emsp;&emsp;Android Studio的1.1.0版本都发布了，ADT也不会再更新了，童鞋们还有理由不换嘛，不要死守着Eclipse了，Android Studio是你唯一的也是最好的选择。什么？用Eclipse用惯了，快捷键神马的很难改过来的。放心，用Android Studio开发照样可以保持Eclipse的快捷键，下面跟我一步一步配置吧！

&emsp;&emsp;下载安装神马的我就不说了，对于程序员的你应该不是问题，直接[Android Developer](developer.android.com)下去。运行Android Studio，新建一个HelloWorld工程，如果你是第一次运行Studio的话，它会自动帮你下载gradle，如果不翻墙，这个过程会非常缓慢，为了提高大家的工作效率，我建议大家购买VPN，不光是为了开发资源的下载，在使用Android Studio的时候我们也时常需要访问“墙外”的资源。

&emsp;&emsp;新建工程成功之后，你将会看到

![studio-default-page](/images/as-default-page.png)

&emsp;&emsp;这里我给出的黑色主题的界面，默认情况下应该是白色主题的界面，我个人是比较喜欢黑色主题的，这样看起来眼睛会比较舒服。如果你想更改主题的话，可以按如下操作：

* 打开Android Studio->Preferences，选择Appearance，更改红方框里的东西

![studio-theme](/images/as-theme.png)

**注意**，这里的字体和大小更改的是界面左边侧边栏中Project目录结构的字体和大小，更改之后会提示你重启Android Studio，**重启之后更改生效**。

&emsp;&emsp;刚刚只是更改了界面主题和侧边栏的字体和大小，代码编辑界面的主题和字体大小的更改，如下：

* 依然是Android Studio->Preferences，选择Editor->colors&fonts->font，更改右边红方框中的schema和font

![studio-code-theme](/images/as-schema-font.png)

**注意**，font默认是不能更改的，你必须点击schema右边的save as按钮，**将当前schema另存为个人自定义的schema，才能更改**。

&emsp;&emsp;Studio只提供给我们两个主题，一个是Default，另一个Darcula，如果这两个主题你都不喜欢怎么办？没关系，这里有一个[Android Studio主题](http://www.ideacolorthemes.org/themes/)的网站，里面有各式各样的主题，挑选一个你喜欢的下载下来，然后导入到Studio当中即可。导入方法为：

* File -> Import Settings，然后选择刚刚下载的主题jar包，重启Studio即可

![studio-import-theme](/images/as-import-setting.png)

&emsp;&emsp;主题更改完毕，下面是快捷键，使用Android Studio一样可以保持Eclipse中的快捷键习惯，具体步骤是：

* Android Studio->Preferences，选择Keymap，根据系统，选择右边红方框中相应的Eclipse

![studio-keymap](/images/as-keymap.png)

这是Studio提供的将大部分快捷键替换成Eclipse风格，之所以说大部分，是因为还有个别快捷键需要特殊设置

1. Studio默认是自动提示的，也就是说你每敲一个字母，都会弹出提示框
    
    ![studio-popup](/images/as-popup.png)
    
    可能有些人不喜欢这样的风格，他们的习惯更倾向于手动提示，也就是手动按住option + /（windows下是alt + /）弹出提示具体步骤如下

    * Android Studio->Preferences，选择Editor -> code completion，右边红方框内，将auto popup的勾选去掉
    
    ![studio-autopopup](/images/as-autopopup.png)
    
    * 然后选择Keymap，搜索框内输入class name completion（熟悉的人可能知道对应Eclipse中的名称是content assist）
    
    ![studio-contentassist](/images/as-contentassist.png)
    
    * 然后右键点击点击class name completion，选择更改keyboard shortcut，在弹出的编辑框中输入你个人习惯的快捷键，按Eclipse的开发习惯来说应该是option + /（windows下是alt + /）
    
    ![studio-contentassist-shortcut](/images/as-contentassist-keyboard-shortcut.png)
    
2. Studio默认是首字母大小写敏感的，什么意思，就拿Intent为例，你输入Inte，提示框会给出Intent的提示，但是你输入inte，Intent的提示就不会有，我个人比较讨厌这样，所以可以这么改

    * Android Studio->Preferences，选择Editor -> code completion，红方框内case sensitive completion的下拉框默认是First letter，改选成None即可

    ![studio-firstletter](/images/as-firstletter-sensitive.png)
    
3. 当我们点击一个class或者一个super方法，想要查看源代码的时候，Eclipse的习惯是按住Command + 鼠标click（windows下是Ctrl + click），点过去查看，这里默认不是，更改步骤

    * Android Studio->Preferences，选择Keymap，搜索框内输入Declaration
    
    ![studio-declaration](/images/as-declaration.png)
    
    * 右键点击Declaration，选择Add Mouse Shortcut，在有鼠标icon的区域内设置Command + click（windows下是Ctrl + click）
    
    ![studio-declaration-mouse-shortcut](/images/as-declaration-mouse-shortcut.png)
    
4. 最后一个是Auto Import，当我们拷贝代码到一个java文件时，需要Studio为我们自动引入相关的包名，更改步骤是

    * Android Studio->Preferences，选择Editor -> Auto Import，红色方框中java标题下面的方框全部勾选即可
    
    ![studio-auto-import](/images/as-auto-import.png)    
    

 &emsp;&emsp;搞定收工，现在大家可以愉快的使用Android Studio啦！
   




