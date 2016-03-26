title: 【iOS Tip】将图片、声音、nib(xib)等资源打包为Bundle
date: 2015-05-21 23:55:58
categories: iOS Tips
tags: [bundle]
---

在编写第三方库时, 如果需要用到一些图片、声音资源, 甚至是nib(xib), 就需要把这些资源打包成一个bundle. 一开始在其他第三方库中看到bundle的时候, 觉得它好像是一个很高级的东西. 但是事实上, bundle就是一个普通得不能再普通的文件夹, 只是加上了`.bundle`后缀, 一下子就高大上了起来. 
加上`.bundle`后缀的文件夹, 会被Mac识别为一个包. 将文件夹以包的形式存在, 可以当作一个整体方便地移动, 也可以让别人不至于不小心改动到库所依赖的资源. 

# **步骤**
1.将资源放到文件夹中, 重命名文件夹为`xxx.bundle`

2.若bundle中有使用到xib文件, 需要使用命令把xib文件转换为nib文件:
```vim
$ ibtool --errors --warnings --output-format human-readable-text --compile file.nib file.xib
```
*如果不转换, 读取的时候会导致如下错误:
```
Terminating app due to uncaught exception 
'NSInternalInconsistencyException', reason: 'Could not load NIB in bundle:
 'NSBundle </var/mobile/Applications/C6718DB8-0C0F-4D38-84E6-55C145279957
/Documents/asset-4.bundle> (not yet loaded)' with name 'file''
```

3.把bundle拖进工程, 此时通过以下方法即可取到ZYBannerView.bundle
```
NSBundle *bundle = [NSBundle bundleWithPath:[[NSBundle mainBundle] pathForResource:@"xxx" ofType:@"bundle"]];
```