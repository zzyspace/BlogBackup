title: 【iOS开发】DNS解析失败的处理 (支持IPv6)
date: 2016-09-07 13:23:03
categories: iOS开发
tags: [dns]
---

![](/img/dns/header.png)

在之前做的一个项目中, 网络请求常常会出现以下错误:
`Error Domain=NSURLErrorDomain Code=-1003 "A server with the specified hostname could not be found."`
错误提示也很明显地指出了原因: 指定域名的服务器无法被找到. 其实这个问题跟自己项目的代码无关, 跟服务端也没有关系, 真正的原因是来自于DNS (Domain Name System). 

<!--more-->

# DNS
---

我们知道, 网络中都是对应一个IP地址(比如208.80.154.225), 我们唯有知道了IP地址才能进行访问. 因为IP地址不利于记忆, 所以出现了"域名"(比如google.com)这个东西. 我们可以想象要给一个人打电话, 只要在电话本中找到输入相应的名字, 就能拨打出去了. DNS就是起到类似电话本的作用, 可以将你访问的域名解析成IP, 进而进行访问. 而这个解析的过程中, 就有很多坑了:

* **域名劫持**: 大部分情况下, 默认的DNS都是运营商提供的. 如果DNS上的某条记录被修改了, 就会导致域名和IP对应不上的问题. (至于为啥会被劫持, 那得问问运营商去了, 利益的诱惑那么多)

- **域名污染**: GFW(Great Fire Wall)就是利用的域名污染的手段, 你如果不科学上网的话, 就访问不了某些网站了.

就是因为这些小动作, 有时候会导致DNS解析的一些问题. 然后我们就会遇到文章开头的问题了.

# 解决方法
---

[windgo](http://www.jianshu.com/users/94b6bbf8765a/latest_articles) 的[《iOS应用中的DNS解析错误的处理》](http://www.jianshu.com/p/a8a8ff984f2e/comments/4050366#comment-4050366)中提供了很好的解决方案(因为作者没有更新对IPv6的支持, 所以我贴上了自己更改后的代码, 再次感谢原作者提供的思路):

- 我们可以在应用启动或者网络连通性变化时, 对域名进行解析:

```objc
#import <arpa/inet.h>

- (BOOL)resolveHost:(NSString *)hostname
{
    Boolean result;
    CFHostRef hostRef;
    CFArrayRef addresses;
    NSString *ipAddress = nil;
    hostRef = CFHostCreateWithName(kCFAllocatorDefault, (__bridge CFStringRef)hostname);
    if (hostRef) {
        result = CFHostStartInfoResolution(hostRef, kCFHostAddresses, NULL); // pass an error instead of NULL here to find out why it failed
        if (result) {
            addresses = CFHostGetAddressing(hostRef, &result);
        }
    }
    if (result) {
        CFIndex index = 0;
        CFDataRef ref = (CFDataRef) CFArrayGetValueAtIndex(addresses, index);
        
        int port=0;
        struct sockaddr *addressGeneric;
        
        NSData *myData = (__bridge NSData *)ref;
        addressGeneric = (struct sockaddr *)[myData bytes];
        
        switch (addressGeneric->sa_family) {
            case AF_INET: {
                struct sockaddr_in *ip4;
                char dest[INET_ADDRSTRLEN];
                ip4 = (struct sockaddr_in *)[myData bytes];
                port = ntohs(ip4->sin_port);
                ipAddress = [NSString stringWithFormat:@"%s", inet_ntop(AF_INET, &ip4->sin_addr, dest, sizeof dest)];
            }
                break;
                
            case AF_INET6: {
                struct sockaddr_in6 *ip6;
                char dest[INET6_ADDRSTRLEN];
                ip6 = (struct sockaddr_in6 *)[myData bytes];
                port = ntohs(ip6->sin6_port);
                ipAddress = [NSString stringWithFormat:@"%s", inet_ntop(AF_INET6, &ip6->sin6_addr, dest, sizeof dest)];
            }
                break;
            default:
                ipAddress = nil;
                break;
        }
    }
    
    if (ipAddress) {
        return YES;
    } else {
        return NO;
    }
}
```

- 如果`-resolveHost:`的返回值为`NO`的话, 说明解析失败了, 需要将项目中的api的域名替换为IP进行访问.

# 参考
---

1.[iOS应用中的DNS解析错误的处理](http://www.jianshu.com/p/a8a8ff984f2e/comments/4050366#comment-4050366)
2.[扫盲 DNS 原理，兼谈“域名劫持”和“域名欺骗/域名污染”](https://program-think.blogspot.com/2014/01/dns.html)



