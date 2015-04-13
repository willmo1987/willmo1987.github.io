---
layout: post
title: 怎样用Android Studio导入Fresco
categories: Android
tags: fresco
---

&emsp;&emsp;**本博客为个人原创，转载需在明显位置注明出处**

&emsp;&emsp;大概一周之前，Facebook开源了专为Android系统定制的图片下载缓存工具，当天该消息就上了各大技术论坛网站的头条，也成为了各个技术群里讨论的最主要的话题。也就在当天[stay4it](http://stay4it.com/)的QQ群里面就有人尝试着用Android Studio导入Fresco，折腾了半天还是失败了，发生这种情况的不止一个人，很多人都碰到这样的问题，导入不成功，编译不了，更别说运行了。前两天正好清明节放假，我怀着好奇的心情也尝试了一次，发现确实很麻烦，但是最终还是成功的编译运行了。先分享一下sample运行的效果图，勾引一下大家：

![fresco_sample](/images/fresco_sample.png)

&emsp;&emsp;现在就把我的导入过程分享给大家，希望大家不要走弯路。本来有想过在我的[Android Studio系列视频课程](http://stay4it.com/course/15)里面再额外增加一节课来专门讲解Fresco的导入，但是想了想还是决定写成博客分享给大家，原因大家往下看就知道了。

先介绍一下我电脑的系统以及环境：

* Macbook Pro
* Yousemite
* Android Studio 1.2 beta （刚刚升级的，大部分人应该还是1.1的版本）
* Gradle 2.2.1-all

另外你还需要预先下载[Android NDK](https://developer.android.com/tools/sdk/ndk/index.html)，并且将Android NDK的路径配置到环境变量里面，具体的下载与配置我这里就不再描述了，作为一个优秀的程序员这些都不是问题，记得翻墙哦。

&emsp;&emsp;相信大部分朋友用的是Windows系统，但是由于我本人已抛弃Windows太久，家里的Windows机器已过花甲之年，也已病入膏肓所以就没有在Windows系统上尝试，但是从[Github上的Fresco issues](https://github.com/facebook/fresco/issues?q=is%3Aopen+is%3Aissue)来看，Windows的导入还是存在问题的，至少我写这篇文章的时候还没有解决。由于Facebook的团队都是用Mac或者Linux来开发的，所以Windows上并没有测试过，所以会出现各种各样的bug，这里有一个[Github上关于Windows的open issue](https://github.com/facebook/fresco/issues/24)大家可以参考一下或者实时跟进。

&emsp;&emsp;好了，让我们继续来看Mac上怎么导入的，首先，将[Fresco](https://github.com/facebook/fresco)从Github上clone下来。**这里跟大家提醒一下，整个导入过程最好翻墙，因为在导入以及后面build的时候，有部分资源是必须翻墙才能访问的。而且导入以及build的过程比较耗时，需要下载很多相关资源，所以最好确保自己在网络状况良好的情况下尝试导入。这也是我为什么没有录制成视频课程的主要原因。**打开Android Studio，选择Import Project，如图：

![fresco_as_import](/images/fresco_as_import.png)

&emsp;&emsp;选择你clone下来的Fresco的路径，点击确定，接下来就是一个长时间的下载以及导入的过程，如图：

![fresco_importing](/images/fresco_importing.png)

&emsp;&emsp;这是一个漫长的过程，这段时间大家可以干点别的事情，玩个游戏休息一下或者看看[Fresco的中文文档](http://fresco-cn.org/)之类的，导入的过程当中主要下载的资源有：

1. 项目中依赖的jar包 （每个module中的build.gradle文件中依赖的jar包，jcentral或者maven）
2. 各式各样的插件 （包括1.0.1的gradle插件，基于JVM的自动化测试工具robolectric插件等等，很多）

&emsp;&emsp;当你各种资源下载完成进入到Android Studio的主界面之后，你已经成功了一半了。下一步就是build，**这里大家要特别注意，只能用命令行进行build，不能使用菜单栏上面的build->make project或者rebuild project，因为利用后者build会失败。**我碰到的报错如下图：

![fresco_gui_build](/images/fresco_gui_build.png)

&emsp;&emsp;当然，在碰到上述错误的情况下我也尝试过去搜索解决方案，Github上有这样一条关于[Mac导入build的issue]，里面有人提到要将imagepipeline module的build.gradle中的ndk路径全部配成你本地的绝对路径，我尝试了一下，还是失败了，具体错误我就不再贴了，我觉得这里还存在bug，有兴趣的朋友可以关注一下刚提到的issue。

&emsp;&emsp;既然GUI的build不行，我们就用命令行来build，打开Android Studio左下角的terminal tab，检查一下terminal的路径是不是Fresco工程的根目录，因为gradlew和gradle wrapper均在工程的根目录下。接着运行./gradlew clean将工程clean一下，可能你会碰到这样的错误：

![fresco_terminal_clean_failed](/images/fresco_terminal_clean_failed.jpg)

&emsp;&emsp;没关系，不用管他，我们可以进行手动clean，将每个module中的build文件夹都删掉就行，其实clean的过程就是如此。删干净之后我们就可以运行./gradlew build了，整个build的过程可以分为三个阶段：

1. 又是一轮download，主要是imagepipeline中需要一些资源，如图：

    ![fresco_terminal_build_download](/images/fresco_terminal_build_download.png)
    
2. 所有资源download成功之后又会进入clonewebp的阶段，这个阶段将会持续一段时间，并且没有任何log出现在命令行，如图：

    ![fresco_terminal_build_clonewebp](/images/fresco_terminal_build_clonewebp.png)
    
    注意，在clonewebp的过程当中，如果你没有翻墙的话可能会出现这样一个错误，如图：
    
    ![fresco_terminal_build_clonewebp_failed](/images/fresco_terminal_build_clonewebp_failed.png)
    
    大家一看便知为什么会失败，chromium相关的资源，需要从googlesource去download的
    
3. 还有一个常见问题也会发生，就是取消lint error的问题，如图：

    ![fresco_terminal_build_linterror](/images/fresco_terminal_build_linterror.png)
    
    解决方案很简单，就是在imagepipeline的build.gradle文件中的android的代码块中添加如下图所示的代码：
    
    ![fresco_linterror_solution](/images/fresco_linterror_solution.png)
    
&emsp;&emsp;经历过上述三个阶段之后，你的build也应该会成功了，整个build的过程应该在10分钟左右，说实话我也头一次见过build这么长时间的工程。build成功之后应该如下图：

![fresco_terminal_build_succeed](/images/fresco_terminal_build_succeed.png)

&emsp;&emsp;好了，等你到了这一步，Fresco的导入也完成了，祝贺你成功了，下面我们来运行一下吧：

![fresco_sample_fresco](/images/fresco_sample_fresco.png)

![fresco_sample_spinner](/images/fresco_sample_spinner.png)

&emsp;&emsp;大家可以通过Spinner随便选一个加载工具将所有图片和gif下载下来，然后切换加载工具进行比较，**你会发现Fresco的缓存加载效率比我们常用的picasso和imageloader高太多，效果太明显了。**至于下载速度我个人觉得区别不大，这个跟网络环境有关，就算相同网络环境下的下载速度应该差距不大。

&emsp;&emsp;这就是我导入Fresco的全部过程，时间有限，能力一般，如果哪里有讲解错误，或者你有更好的解决方案请给我留言，咱们一块讨论讨论。












