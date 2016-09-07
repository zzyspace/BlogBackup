title: 【iOS Tip】提高Xcode编译速度
date: 2016-03-20 15:46:56
categories: iOS Tips
tags: [Xcode]
---

![](/img/reduce-xcode-build-times/title.png)
平常在调试工程的时候, 特别是大工程, 浪费在编译上的时间加起来也是个不小的数目. 我戏成它为**带薪编译**. 带薪编译一来呢浪费公司资源, 二来使得开发效率大大降低. 所以提高Xcode的编译速度还是非常有必要的. 关于通过更改一些Xcode的配置来提升编译速度[这篇文章](http://blog.csdn.net/zhaoxy_thu/article/details/30073485?utm_source=tuicool&utm_medium=referral)已经列举除了几个不错的方法, 这里就不再多说.

这篇文章主要讲的是从**硬件方面**来提升Xcode的编译速度. 因为对比了现在日常使用的Mac mini和MacBook Pro, 在CPU和内存基本没有差别的情况下, 相比MacBook Pro的SSD, Mac mini的机械硬盘的表现实在是惨不忍睹. 这也是拖慢了编译速度的主要元凶. ~~所以, 去换一个SSD吧.(Excuse me?)~~

本文所讲的方法理论上编译速度是可以超过SSD的. 在说具体方法之前, 先来了解一下拖慢编译速度的元凶Derived Data:

<!--more-->

# Derived Data
---
Derived Data是Xcode自动生成的一些派生数据的文件夹. 里面包含有一些索引文件、Log、**编译时产生的文件**. 编译时候产生的文件包括:
- 第三方库的`.a`文件
- 应用程序`.app`文件
- 保存函数地址映射信息的中转文件`.dSYM`文件

所以, 写这些数据的时候, 写得越快, 编译的时间也就能越少. 如何让数据写入得更快呢? 可以把Derived Data的路径换到内存中. 换句话说, 将Derived Data的读写从硬盘移动到内存中.

# 在内存中创建虚拟磁盘
---
What? 内存当硬盘用? 是的你没听错, 其实OS X系统是允许在内存中创建一个可以高速存取的虚拟磁盘的. 通过以下两种方法可以实现在内存中创建虚拟磁盘:

### 1. iRamDisk
在AppStore上找到了这么一个神器, 叫[iRamDisk](https://itunes.apple.com/us/app/iramdisk/id492615400?mt=12), 14.99刀. 使用它可以非常简单地创建一个Xcode的DerivedData的虚拟磁盘, 如下. 都是图形化界面, 很好操作, 就不多赘述了.
![](/img/reduce-xcode-build-times/iramdisk.png)
这里分享一个[破解版](http://pan.baidu.com/s/1hqLLc3a)(提取码`9u38 `)仅供学习交流使用, 建议购买正版支持开发者.

### 2. 命令行创建
**① 创建RAM disk并分配空间**
size的计算公式 `size = 需要分配的空间(M) * 1024 * 1024 / 512`. 例如下面分配的空间为2GB: `2048 * 1024 * 1024 / 512 = 4194304`.
```vim
$ hdid -nomount ram://4194304
```
执行完这个命令之后, 终端会输出`/dev/diskN`, `N`是一个数字, 记下它, 后面两步有用.

**② 在RAM disk上创建HFS**
给虚拟磁盘创建名字, 如下为`DerivedData`, 并将`rdiskN`的`N`改为第一步中的`N`的数字. (*注意: `rdiskN`不要错写成`diskN`*)
```vim
$ newfs_hfs -v DerivedData /dev/rdiskN
```

**③ 把虚拟磁盘安装到Xcode的DerivedData目录下**
如下命令, 同样把`diskN`中的`N`替换成第一步中`N`的数字. (*注意: 此时是`diskN`*)
```vim
$ diskutil mount -mountPoint ~/Library/Developer/Xcode/DerivedData /dev/diskN
```
这样就大功告成了! 不出意外就可以在桌面(或者Finder)上看到`DerivedData`这个盘. 如果不要虚拟磁盘了, 可以在Finder中推出这个磁盘, 或者执行以下命令**卸载**这个磁盘:
```vim
$ diskutil umount /dev/diskN
```

# 总结
---
### 1. 优点: 
- 编译速度加快.
- 创建虚拟磁盘后, 并不是直接占用掉所有分配的空间, 而是根据虚拟磁盘中的文件总大小来逐渐占用内存.

### 2. 缺点: 
- 每次关闭系统之后, 虚拟磁盘将会被卸载, 下次开机后需要重新创建虚拟磁盘. 同时因为Derived Data已经被删除.
- 内存有限, 不宜创建太大的虚拟磁盘, 需要在系统性能和需要的虚拟磁盘空间中找一个平衡点.

### 3. 可能遇到的坑
- 如果创建的虚拟磁盘已满, 会导致编译的失败. 此时清除掉Derived Data后重新编译, 就算有足够的空间也还是有可能会导致编译失败. 重启Xcode可以解决此问题.

