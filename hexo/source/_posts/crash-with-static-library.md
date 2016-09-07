title: 【iOS开发】静态库导致的运行时崩溃问题
date: 2015-10-18 20:45:33
categories: iOS开发
tags: [Other Linker Flags]
---
使用其他一些第三方静态库时, 如果没有注意按照文档中的提示进行配置, 很容易在程序运行过程中因 "unrecorgnized selector send to instance xxx" 的异常而崩溃. 而且可以发现, 导致崩溃的方法都是Category中的方法. 

<!--more-->

在[苹果官方文档](https://developer.apple.com/library/mac/qa/qa1490/_index.html)中解释到: 
> The dynamic nature of Objective-C complicates things slightly. Because the code that implements a method is not determined until the method is actually called, Objective-C does not define linker symbols for methods. Linker symbols are only defined for classes.
Since categories are a collection of methods, using a category's method does not generate an undefined symbol. This means the linker does not know to load an object file defining the category, if the class itself is already defined. This causes the same "selector not recognized" runtime exception you would see for any unimplemented method.

翻译过来, 就是因为Objective-C的动态特性使得方法是在运行时才确定的, 所以链接的时候不会为方法(Method)定义连接符号, 而是为类(Class)定义连接符号. 这样当在一个静态库中使用类别来扩展已有类的时候，链接器不知道如何把类原有的方法和类别中的方法整合起来, 所以就导致运行的时候Category的方法无法被找到抛出异常.


# 解决方法: Other Linker Flags
---

- ### -ObjC

[苹果官方文档](https://developer.apple.com/library/mac/qa/qa1490/_index.html)对这个flag的解释是这样的:
> Passing the -ObjC option to the linker causes it to load all members of static libraries that implement any Objective-C class or category. This will pickup any category method implementations. But it can make the resulting executable larger, and may pickup unnecessary objects. For this reason it is not on by default.

  `-ObjC`这个标志选项会让链接器加载静态库所有的Objective-C的类和Category, 这样就能把Category中实现的方法整合起来. 但是由于这样做会使可执行文件变大, 也会整合一些用不到的对象, 所以才没有默认使用-ObjC标志, 而是需要我们手动添加.

- ### -all_load

加载所有静态库中的文件. 相比`-ObjC`, 不同点就是`-all_load`会将所有的(包括非Objective-C)文件都整合到静态库中. 
_***注意** : 假如你使用了不止一个静态库，然后又使用了这个参数，那么你很有可能会遇到`duplicate symbol`错误，因为不同的库文件里面可能会有相同的目标文件._

- ### -force_load (path_to_archive)

加载指定路径的静态库. 相比`-all_load`, 不同点就是`-force_load`只是完全加载了一个库文件，不影响其余库文件的按需加载.

# 补充:
---
使用`-all_load`或者`-force_load`大部分原因是因为**Xcode4.2之前**的版本的链接器的bug, 在64位iOS应用环境下当静态库中只有分类而没有类的时候, `-ObjC`参数就会失效了. 所以为了兼容**Xcode4.2之前**的版本, 有两种解决方法:

- ### 静态库使用者:

使用`-all_load`或者`-force_load`代替`-ObjC`.

- ### 静态库开发者:

可以通过在分类中添加一个类的声明和实现, 使得Category源文件中不仅仅只有分类, 同时还有类存在来避免链接器的bug, 从而避免了`-ObjC`标志失效的麻烦:

```objc
/**
 *   NSObject+Addition.m
 */

// add a dummy class
@interface DUMMY_CLASS_NSObject_Addition : NSObject
@end
@implementation DUMMY_CLASS_NSObject_Addition
@end

@implementation NSObject (Addition)
// some category methods...
@end
```

但是**Xcode4.2之后**, 只要使用`-ObjC`即可. 具体可以参看stackoverflow的[这个回答](http://stackoverflow.com/questions/2567498/objective-c-categories-in-static-library/2615407#2615407).

