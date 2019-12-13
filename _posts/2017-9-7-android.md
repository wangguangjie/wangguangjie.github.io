---
layout: post
title: Android:信息爬取
---


>这个第一个在APP Store正式发布有完善功能的android程序，也算是小小的练手，用博客记录下来，在此做一个记录与总结...

#### 一.程序功能
  本款APP主要是通过android客户端随时随地的获取并且查看哈工大发布的新闻、通知、公告等信息，主界面简洁，可以很快速的查看消息概要，也可点击进入具体消息，支持本地查看与手机自带浏览器查看。

#### 二.程序界面
   - 主界面
    <img src="http://ovy9gem9a.bkt.clouddn.com/project1-hit/1.png" class="project1-1-picture" width="176" height="368" align="center">
   - 详细信息
    <img src="http://ovy9gem9a.bkt.clouddn.com/project1-hit/2.png" class="project1-1-picture" width="176" height="368" align="center">


#### 三.程序知识点与创新点
  1.采用Jsoup库获取网页html文档，再从文档中解析出需要的内容。  
  2.本程序由于是解析html文档得到有用信息，避免主线程产生ANR和提高用户体验，所以不可避免使用多线程，并且使用线程间通信，使程序正确有效的运行。  
  3.由于需要获取多种信息类型，所以采用Spinner来切换内容。  
  4.为了提高用户体验，每次用户刷新或者自动刷新都添加了动画。  
  5.本程序采用了github开源项目的PullList，并且进行改造(由于没时间就没重写了),实现了下拉更新等。  
  6.为了提高用户体验，本程序采用了缓存的机制，当前的最新的内容存储在本地，以便下次用户打开软件的时候可以看到上次的内容，而不用等待太长时间。

### 四. 总结

&nbsp;&nbsp;&nbsp; 通过本程序锻炼自己编程的思考能力，同时也对一些知识点进行了实践，并且学习优秀的开源项目。虽然程序功能有限，但是也对哈工大的学生有一定的实际意义，本程序只作为第一个项目，以后还会更多，也会持续更新，希望越来越好...本款APP在腾讯应用宝上线，有需要的小伙伴们，可以去下载哦需要复制， 然后在手机QQ或者微信打开，请多多支持！非常感谢！！！ 

>下载地址:  
<a>http://fusion.qq.com/cgi-bin/qzapps/unified_jump?appid=52486820&from=mqq&actionFlag=0&
params=pname%3Dhit.wangguangjie.com.hit%26versioncode%3D1%26channelid%3D%26actionflag%3D0</a>

下面是我的个人连接，有兴趣的的可以关注我：
> - [CSDN](http://blog.csdn.net/wgj13718925364)
 - [博客园](http://www.cnblogs.com/wangguangjie/)
 - [office365:xm649@silently.party](https://www.office.com/1/?auth=2&home=1&from=ShellLogo)
 - [github](https://github.com/wangguangjie)
 - [QQ]()
 <img src="http://ovy9gem9a.bkt.clouddn.com/HIT/QQ.png" class="qq-picture" width="138" align="center">
 - [微信]()
  <img src="http://ovy9gem9a.bkt.clouddn.com/HIT/weixin.png" class="weixin-picture" width="138" align="center">

  -----------------
  --------------------