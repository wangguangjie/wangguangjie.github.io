---
layout: post
title: "Synchronized"
---

### Java Synchronized 关键字
- 当两个并发线程访问同一个对象object中的这个synchronized(this)同步代码块时，一个时间内只能有一个线程得到执行。另一个线程必须等待当前线程执行完这个代码块以后才能执行该代码块。
- 然而，当一个线程访问object的一个synchronized(this)同步代码块时，另一个线程仍然可以访问该object中的非synchronized(this)同步代码块。
- 尤其关键的是，当一个线程访问object的一个synchronized(this)同步代码块时，其他线程对object中所有其它synchronized(this)同步代码块的访问将被阻塞。

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
