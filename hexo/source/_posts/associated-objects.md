title: 【iOS开发】Associated Objects-为分类添加属性
date: 2016-01-25 15:06:34
categories: iOS开发
tags: [runtime]
------
分类(`category`)在iOS开发中的应用非常广泛, 优点譬如给现有的类拓展更多的方法、对一个类的多种功能进行局部化封装等等, 都是非常方便的. 但是也有一个痛点, 就是分类中无法添加属性. 但是`Objective-C`的`runtime`中有许多黑科技可以帮我们实现很多常规方法下几乎不可能的事情--比如在分类中添加属性.这个黑科技叫做**关联对象**(`Associated Objects`). 

# **Associated Objects**
---
关联对象相关的函数有以下3个:
- `objc_setAssociatedObject` : 设置关联对象
- `objc_getAssociatedObject ` : 获取关联对象
- `objc_removeAssociatedObjects ` : 移除某个对象的所有关联对象

<!--more-->

从`<objc/runtime.h>`中可以找到它的相关函数定义:
### **1. 设置关联对象**
```objc
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
```
- `object` : 需要设置关联对象的对象, `id`类型
- `key` : 关联对象的key, `const void *`类型 (_详细请看下文第4点_)
- `value` : 关联对象的值, `id`类型
- `policy ` : 关联对象的策略, `objc_AssociationPolicy`类型
 - `policy`是一个枚举类型, 用于修饰关联对象:
```objc
enum {
        OBJC_ASSOCIATION_ASSIGN = 0,          // 等价于 @property (assign) 或 @property (unsafe_unretained)
        OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1,// 等价于 @property (nonatomic, strong)
        OBJC_ASSOCIATION_COPY_NONATOMIC = 3,  // 等价于 @property (nonatomic, copy)
        OBJC_ASSOCIATION_RETAIN = 01401,      // 等价于 @property (atomic, strong)
        OBJC_ASSOCIATION_COPY = 01403         // 等价于 @property (atomic, copy)
};
```

### **2. 获取关联对象**
```objc
id objc_getAssociatedObject(id object, const void *key)
```
- `object` : 获取关联对象的对象, `id`类型
- `key` : 关联对象的key, `const void *`类型

### **3. 移除某个对象的所有关联对象**
```objc
void objc_removeAssociatedObjects(id object)
```
- `object` : 需要移除所有关联对象的对象, `id`类型

 _注: 这个函数是用来移除对象的**所有**关联对象, 而非移除对象的某个关联对象. 这个函数Apple官方文档是这么说的_ :
> You should not use this function for general removal of associations from objects, since it also removes associations that other clients may have added to the object. Typically you should use objc_setAssociatedObject with a nil value to clear an association.

 *意思是如果要移除对象的某个关联对象, 应该使用`objc_setAssociatedObject`对参数`value`置nil.*

### **4. 关于参数-key**
这个key一般只要赋值一个`static char`的地址就行, 比如:
```objc
static char kAssociatedObjectKey;

objc_getAssociatedObject(self, &kAssociatedObjectKey);
```
但是还有更简单的方法, 可以使用`selector`:
```objc
objc_getAssociatedObject(self, @selector(associatedObject));
```
或者直接使用`_cmd`:
```objc
objc_getAssociatedObject(self, _cmd);
```
* *关于`_cmd`* :

  *Apple的文档是是这么解释的: *
> The _cmd variable is a hidden argument passed to every method that is the current selector

  *意思就是`_cmd`在Objective-C的方法中表示当前方法的`selector`, 正如同`self`表示调用当前方法的对象(类)一样.*  

# **Simple Example**
---
```objc
@interface NSObject (AssociatedObject)
@property (nonatomic, strong) id associatedObject;
@end

@implementation NSObject (AssociatedObject)

/**
 *  对象的setter
 */
- (void)setAssociatedObject:(id)object {
     objc_setAssociatedObject(self, @selector(associatedObject), object, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}

/**
 *  对象的getter
 */
- (id)associatedObject {
    return objc_getAssociatedObject(self, @selector(associatedObject));
}

@end
```

# **Tips**
---
最近看到[Sunny大神](http://blog.sunnyxx.com)说的一个关于Associated Object的巧妙运用:
> 利用 association 防止多次调进一个方法，总觉得为这种事情去加一个成员变量会让一段逻辑的代码过于分散，喜欢能 self-managed 的函数

```objc
@interface Engine : NSObject
@end

@implementation Engine

- (void)launch {
     // 在对象生命周期内, 不增加 flag 属性的情况下, 放置多次调进这个方法
     if (objc_getAssociatedObject(self, _cmd)) return;
     else objc_setAssociatedObject(self, _cmd, @"Launched", OBJC_ASSOCIATION_RETAIN);

     NSLog(@"launch only once");
}

@end
```

这个做法相当于是动态的添加了flag属性, 相对与直接使用flag属性来说简直是优雅多了. 我们甚至可以把这两行代码写作一个宏, 更方便于每次的使用.

### 参考
1.[Associated Objects](http://nshipster.com/associated-objects/) by Mattt
2.[Objective-C Runtime Reference](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ObjCRuntimeRef/index.html#//apple_ref/c/func/objc_getAssociatedObject) by Apple