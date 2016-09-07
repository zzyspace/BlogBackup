title: 【Linux】常用命令行
date: 2015-10-12 00:04:56
categories: iOS Tips
tags: [Linux]
---

![](/img/terminal-command/header.png)

<!--more-->

### **热键**
---
- 自动补全: `tab`
- 终止命令: `ctrl`+`c`
- 光标移动至行首: `ctrl`+`a`
- 光标移动至行尾: `ctrl`+`e`

### **路径操作**
---
- 当前文件夹的内容: `ls`
- 当前文件夹的内容(包括隐藏文件): `ls -a`
- 当前文件夹的内容(包含其属性权限): `ls -l`
- 当前文件夹下的所有内容(递归): `ls -R`
- 当前路径: `pwd`
- 当前路径的完整路径: `pwd -P`
- 进入目录: `cd`
- 进入上层目录: `cd ..`
- 进入同级的其它目录: `cd ../xx` (比如当前`/var/mail`要跳转到`/var/msgs`, 即`cd ../msgs`)
- 回到前一个目录: `cd -`
- 回到Home目录: `cd ~`

### **文件\文件夹操作**
---
- 创建文件夹: `mkdir`
- 创建文件夹(若上级不存在则自动创建): `mkdir -p`
- 创建文件: `touch`
- 删除空文件夹: `rmdir`
- 删除空文件夹(若上级为空文件夹则同时删除): `rmdir -p`
- 删除文件夹(递归): `rm -r [dir path]`
- 删除文件: `rm`
- 删除文件(强制, 不会出现警告): `rm -f`
- 复制文件: `cp [source] [destination]`
- 复制目录(递归): `cp -r`
- 复制文件(目标文件比源文件旧才更新): `cp -u`
- 复制为快捷方式: `cp -s`
- 移动文件(重命名): `mv [source] [destination]`
- 移动文件(强制, 如果目标已经存在则会被覆盖): `mv -f`
- 移动文件(目标文件比源文件旧才更新): `mv -u`
- 查看文件: `cat`
- 查看文件(包含特殊字符): `cat -A`
- 查看文件(列出行号): `cat -n`
- 打开文件: `open`
- 隐藏文件: `chflags hidden`
- 显示文件: `chflags nohidden`

### **功能操作**
---
- 查看日期: `date`
- 查看日历: `cal`

### **一些好用的命令**
---

- 显示Mac隐藏文件: 

```zsh
$ defaults write com.apple.finder AppleShowAllFiles -bool true
$ killall Finder #生效
```

- 隐藏Mac隐藏文件: 

```zsh
$ defaults write com.apple.finder AppleShowAllFiles -bool false
$ killall Finder #生效
```

- 使某个应用不出现在Dock上:

```zsh
$ /usr/libexec/PlistBuddy  -c "Add :LSUIElement bool true" /Applications/XXX.app/Contents/Info.plist
```

- 使某个应用重新出现在Dock上:

```zsh
$ /usr/libexec/PlistBuddy  -c "Delete :LSUIElement" /Applications/XXX.app/Contents/Info.plist
```

- 查看某个Objective-C工程有多少行代码:

```zsh
$ cd [project path]
$ find . -name "*.m" -or -name "*.h" -or -name "*.xib" -or -name "*.c" |xargs grep -v "^$"|wc -l
```

- Git超漂亮日志(设置为全局别名, `git lg`使用即可): 

```zsh
$ git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

### 参考
---
1. [linux常用基本命令详解](http://blog.csdn.net/xiaoguaihai/article/details/8705992)
2. [Git教程-廖雪峰](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000/)


