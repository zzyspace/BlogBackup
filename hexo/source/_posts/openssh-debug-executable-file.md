title: 【iOS越狱】使用OpenSSH调试可执行文件
date: 2015-10-12 00:04:56
categories: iOS Tips
tags: [jailbreak]
---

# **一. 连接设备**
---
### **1. 连接**
通过USB连接:

```vim
$ ssh -p 2222 root@localhost
```

通过局域网连接:

```vim
$ ssh root@192.168.9.126
```

### **2. 输入密码**
连接后, 终端会提示:

```vim
The authenticity of host '192.168.9.126 (192.168.9.126)' can't be established.
RSA key fingerprint is xxxx.
Are you sure you want to continue connecting (yes/no)? 
```

此时输入yes同意连接, 然后根据提示输入OpenSSH的默认密码:`alpine`. 到此已经成功连接iOS设备, 并进入了iOS设备的`/var/root`文件夹下.

<!--more-->

# **二. 传输可执行文件**
---
 从服务器上下载文件

```vim
$ scp username@servername:/path/filename
```

上传本地文件到服务器

```vim
$ scp /path/filename username@servername:/path
```

**注意**: 执行这两个命令所在的目录为Mac而不是iPhone

# **三. 执行可执行文件**
---
传完可执行文件后, 此时可执行文件并没有权限, 所以要执行以下命令给权限:

```vim
chmod a+x ExecutableFile
```

执行当前目录下的可执行文件:

```vim
./ExecutableFile
```

# **白苹果的解决方案**
---
一般情况下, 只要没有动到系统文件, 白苹果之后还是有救的, 可以进入安全模式. 安全模式下会禁用所有插件, 如果能成功进入安全模式, 手机就能再次连接电脑了, 同时就有可能恢复之前错误的操作.
### **进入安全模式的方法:**
1. 同时按住home键和电源键
2. 重新出现苹果标志的时候, 放开home键和电源键, 然后按住音量+键不放
3. 正常情况下就可以进入安全模式了