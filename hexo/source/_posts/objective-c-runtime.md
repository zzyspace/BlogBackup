title: 【iOS开发】Objective-c Runtime小记
date: 2016-03-16 17:16:08
categories: iOS开发
tags: [runtime]
---

# **一. Runtime**
Runtime是一套底层的C语言API. 实际上, 平时我们编写的Objective-C代码, 底层都是基于runtime实现的, 也就是说, 平时我们编写的Objective-C代码, 最终都是转成了底层的runtime代码(C语言代码). Runtime使得Objective-C这门语言的灵活性大大地提升. 有了runtime, 我们可以在应用运行的时候动态操作对象、类、方法, 也因为这个原因, 使得编程有了更多的可能性, 对于开发中遇到的一些比较棘手的问题, 往往用runtime可以优雅地解决, 接下来让我们看看runtime是为何可以如此牛x.

# **二. Objective-C中类和对象的本质**
### **1. 对象(Instance)**
**概念:**
对象的本质是一个结构体, 在`<objc/objc.h>`中可以找到它的声明:
```objc
struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
};
```
- `isa`: 指针指向对象所属的类(Class), Class结构体中包含了成员变量、对象方法等等.  

<!--more-->

**补充:**
平常我们常常使用`id`来作为对象的指针, 原因就是`<objc/objc.h>`中定义`id `类型来代替`struct objc_object *`:
```objc
typedef struct objc_object *id;
```

<br/>

### **2. 类(Class)**
**概念:**
类的本质是一个Class类型的对象. 在`<objc/runtime.h>`中, 对类的声明如下:
```objc
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;
#if !__OBJC2__
    Class super_class                                        OBJC2_UNAVAILABLE;
    const char *name                                         OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;
#endif
} OBJC2_UNAVAILABLE;
```
- `isa`: 指针指向Class对象的元类(MetaClass), 元类中记录了类方法列表.
- `super_class`: 指针指向父类. 根类(NSObject)中的super_class指针为空.
- `name`: 类名
- `version`: 类的版本, 默认为0
- `info`: 类的信息, 因为是long型的, 推测是作为标识使用
- `instance_size`: 类的对象的大小
- `ivars`: 存放该类所有成员变量的链表
- `methodLists`: 存放该类所有对象方法的链表
- `cache`: 缓存常用的对象方法, 提高消息分发的效率
- `protocols`: 存放该类的协议的链表

**补充:**
就像`id`类型代表对象指针一样, `<objc/objc.h>`中定义`Class`类型来代替`struct objc_class *`:
```objc
typedef struct objc_class *Class;
```

<br/>

### **3. 元类(MetaClass)**
**概念:**
元类也是一个类, 每个类都有对应的一个元类. 可以通过类中的isa指针找到其对应的元类. 虽然在runtime相关头文件中没有找到MetaClass的声明, 但是在[这个博客](http://www.sealiesoftware.com/blog/archive/2009/04/14/objc_explain_Classes_and_metaclasses.html)中对元类的与类的关系解释中, 我们可以推测出元类结构体和类是相似的, 包含(但不仅有)如下成员:
- `isa`: 指针都是指向根元类(NSObject的元类), 即使是根元类本身的isa也是指向自己
- `super_class`: 指针指向父元类, 根元类指向根类(NSObject)
- `methodLists`: 存放该类的所有类方法的链表

**对象、类、元类的关系图：**
<div style="text-align: center">
<img src="/img/runtime/runtime_0.pdf"/>
</div>

# **三. 消息机制**
我们平常所说的"方法调用", 其实是不准确的, 因为在Objective-C中, 所谓的"方法调用"本质是消息分发. 比如下面这个"方法调用":
```objc
[receiver message]
```
最终会被编译器转化为:
```objc
objc_msgSend(receiver, @selector(message))
```
所以说, 消息分发是通过定义在`<objc/message.h>`中的`objc_msgSend()`方法以及相关的方法来实现的. 

需要注意的是消息分发是运行时特性, 说白了就是运行的时候, 一条消息才会知道它所对应的方法的实现是什么. 所以运行的时候, 一条消息的传递过程是这样的:
>1. 向对象`receiver`对象发送`message`消息.
2. 通过`receiver`对象的`isa`指针找到它的Class.
3. 在Class结构体中的`cache`中查找是否有的`message`的selector, 没有的话到`methodLists`里面查找. 若有找到`message`的selector, 则跳转至对应的方法实现完成此次消息分发.
4. 如果没有找到`message`的实现, `objc_msgSend`会通过当前的Class结构体中的`super_class`指针找到它的父Class, 并重复第3点的动作. 
5. 如果一直没有找到`message`的实现, 第3点与第4点会一直重复直到根类(NSObject).
6. 如果在根类中依然没有找到`message`的实现, 默认(未实现[消息转发](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtForwarding.html#//apple_ref/doc/uid/TP40008048-CH105-SW1)方法的情况下)就会抛出`unrecognized selector send to instance xxxx`的异常.

这里引用[Apple官方文档](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtHowMessagingWorks.html#//apple_ref/doc/uid/TP40008048-CH104-SW1)的消息分发原理插图便于理解:

<div style="text-align: center">
<img src="/img/runtime/runtime_1.gif"/>
</div>

看到这里或许会担心消息分发的过程太过于繁琐, 以至于影响性能? 其实不会的, 一个类中常用的方法会缓存在`cache`中, 如上面消息分发流程的第2点所说, 一个对象接收到一条消息之后, 并不是直接去`methodLists`查找, 而是先在`cache`中查找, 查找不到了再到`methodLists`中查找. 这样就能使得消息转发的效率得到保障.

# 扩展阅读
---
1.[Objective-C Runtime Programming Guide](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html) by Apple
2.[Objective-C Runtime 运行时系列文章](http://southpeak.github.io/blog/2014/10/25/objective-c-runtime-yun-xing-shi-zhi-lei-yu-dui-xiang/) by 南峰子
3.[Objective-C Runtime](http://yulingtianxia.com/blog/2014/11/05/objective-c-runtime/) by 玉令天下
4.[(译)Objective-C的动态特性](http://limboy.me/ios/2013/08/03/dynamic-tips-and-tricks-with-objective-c.html) by Limboy’s HQ
5.[Objective-C 中的消息与消息转发](http://blog.ibireme.com/2013/11/26/objective-c-messaging/) by ibireme
6.[Objective-C Runtime](http://tech.glowing.com/cn/objective-c-runtime/) by Glow技术团队博客
