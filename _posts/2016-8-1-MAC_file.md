---
layout: post
title: "MAC文件目录"
---
>在OS X的系统中，不再有Windows用户熟悉的C盘、D盘，这是因为OS X底层是Unix系统，其目录机构符合Unix系统的规范。MAC机器主板使用了Intel主导的EFI标准，硬盘分区格式采用GPT。这种EFI+GPT的方式相比传统的BIOS＋MBR的方式有很多好处，具体可以参考我之前写的博客。

### 1 硬盘分区

>默认情况下，MAC OS X把硬盘分成了3个GPT分区。第一个就是GPT标准要求的ESP分区，这个分区很小，200MB，FAT文件系统格式。按照EFI惯例，应该用来存放操作系统的引导程序。但是苹果没有遵守这个惯例，它的引导程序boot.efi并没有存放在ESP中，这个分区只是被苹果用来存放升级固件的文件。第二个分区就是OS X的系统分区了，它占用了大部分磁盘空间，用来存放整个OS X系统和用户数据，分区文件系统格式为HFS+。第三个分区是系统恢复分区，里面存放了一个精简的OS X系统，用来完成系统恢复、安装等任务，类似于WindowsPE。默认情况下，OS X自带的磁盘工具并不能显示ESP分区和恢复分区，需要开启DEBUG菜单才可以。开启方法为：
defaults write com.apple.DiskUtility DUDebugMenuEnabled 1
然后重启“磁盘工具”，菜单栏里会多出一项“调试”菜单，选中此菜单中的“显示所有分区”菜单项，就会在左侧显示出磁盘的隐藏分区。如下图所示：
![image](http://img.blog.csdn.net/20131112112502640?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc21zdG9uZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
>此时， ESP分区和恢复分区都显示为灰色，因为此分区虽然存在，但是没有被挂载到系统目录树中，右键点击分区，选择挂载就可以正常显示了，而且可以直接在Finder中查看这个分区。
其中ESP分区的目录结构如下：
![image](http://img.blog.csdn.net/20131112112739578?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc21zdG9uZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

### 2 OS X系统分区的目录结构
>Mac OS X已经是被认证的Unix系统，所以其目录结构基本符合Unix系统目录结构。但是有很多目录在Finder中并看不到，这是因为这些目录的被设置了隐藏属性，我们可以在终端窗口中利用unix命令查看。
可以看出，根目录下存在着传统的unix系统目录，也存在着一些os x特有的目录。
2.1 符合unix传统的目录
- /bin 传统unix命令的存放目录，如ls，rm，mv等。
- /sbin 传统unix管理类命令存放目录，如fdisk，ifconfig等等。
- /usr 第三方程序安装目录。
- /usr/bin, /usr/sbin, /usr/lib，其中/usr/lib目录中存放了共享库（动态链接库）.
- /etc. 标准unix系统配置文件存放目录，如用户密码文件/etc/passwd。此目录实际为指向/private/etc的链接。
- /dev 设备文件存放目录，如何代表硬盘的/dev/disk0。
- /tmp 临时文件存放目录，其权限为所有人任意读写。此目录实际为指向/private/tmp的链接。
- /var 存放经常变化的文件，如日志文件。此目录实际为指向/private/var的链接。
这些标准的Unix目录在Finder中并不可见，如下图所示：
![image](http://img.blog.csdn.net/20131112133128250?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc21zdG9uZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

2.2 os x特有的目录
>OS X系统中，除了标准的unix目录外，还增加了特有的目录。
- /Applications 应用程序目录，默认所有的GUI应用程序都安装在这里；
- /Library 系统的数据文件、帮助文件、文档等等；
- /Network 网络节点存放目录；
- /System 他只包含一个名为Library的目录，这个子目录中存放了系统的绝大部分组件，如各种framework，以及内核模块，字体文件等等。
- /Users 存放用户的个人资料和配置。每个用户有自己的单独目录。
- /Volumes 文件系统挂载点存放目录。
- /cores 内核转储文件存放目录。当一个进程崩溃时，如果系统允许则会产生转储文件。
- /private 里面的子目录存放了/tmp, /var, /etc等链接目录的目标目录。

### 3 用户的资料应该存放到什么目录？
>对于普通OS X用户来说，对系统目录树结构的理解与否并不影响正常使用系统，以至于OS X把很多目录都故意隐藏，让普通用户通过Finder不能看到。用户真正关心的是把自己的资料存放到哪里更加方便和安全。Windows用户通常会把个人资料存放在非系统盘（C）的其他分区中，因为Windows系统一旦死掉，C盘的内容很可能就找不回来了。Mac OS X的用户则不用担心这个问题，OS X发生崩溃和不能启动的概率实在是太低了，就算是系统出现问题，由于用户目录和系统目录是彼此独立的，所以也容易找回。所以通常情况下，用户直接把资料存放在自己的用户目录中，OS X也建议用户这么做，并且已经为用户准备好了常用的子目录，如下图所示：
![image](http://img.blog.csdn.net/20131112135804890?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc21zdG9uZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
应用程序，文档，下载，音乐，电影，图片，公共，对于普通用户这些子目录也就够用了，当然如果你觉得不够，可以自己随便添加，例如上图中就增加了Work目录来存放一些工作的项目文件，家庭照片视频则用来存放来自手机、Dv等等的照片视频资料。
从Windows过来的用户，如果还想保持原来的习惯，把用户文件和系统文件存放在不同的分区中，那么就需要利于“磁盘工具”，重新分区，把系统分区调整小一些，留出空间建立一个新的HFS+分区，使用的时候把这个分区挂载到系统目录树上就可以使用了。


-----------
------------------------