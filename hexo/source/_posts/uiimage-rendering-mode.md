title: 【iOS Tip】UIImage的renderingMode
date: 2015-03-29 14:32:22
categories: iOS Tips
tags: [UI]
---
很多人在第一次使用UITabBar的时候, 都会遇到一个摸不着头脑的问题:
明明UI给的图片是这样的 ↓
![](/img/UIImageRenderingMode/renderingMode_0.png)
可是用到tabBar中却变成了这样 ↓
![](/img/UIImageRenderingMode/renderingMode_1.png)
**WTF!** 咋还自己变色了呢? ![](/img/UIImageRenderingMode/renderingMode_2.jpg)
其实以上的现象都是源于UIImage的一个属性`renderingMode `.

# **UIImageRenderingMode**
---

### **1. 概念**
 
```objc
@property(nonatomic, readonly) UIImageRenderingMode renderingMode NS_AVAILABLE_IOS(7_0);
```

<!--more-->

`UIImageRenderingMode`是一个枚举值, 如下:
```objc
typedef NS_ENUM(NSInteger, UIImageRenderingMode) {
    UIImageRenderingModeAutomatic,      // 根据图片的使用环境和所处的绘图上下文自动调整渲染模式.
    UIImageRenderingModeAlwaysOriginal, // 始终渲染图片的原始状态, 不会将其当做一个模板(template).
    UIImageRenderingModeAlwaysTemplate, // 始终把图片当做模板来渲染, 忽略掉了图片的颜色信息.
} NS_ENUM_AVAILABLE_IOS(7_0);
```

创建一个UIImage的时候, 默认的**renderingMode**为`UIImageRenderingModeAutomatic`. 这种情况下, 图片会根据当前所处的上下文来决定是渲染图片的原始状态或是当做模板来渲染. 
例如`UINavigationBar`、`UITabBar`、`UIToolBar`、`UISegmentedControl`这些控件, 会自动把其上面的图片(foreground images)当做模板来渲染; 而`UIImageView`、`UIWebView`则会渲染图片的原始状态.

> 关于**模板(template)**: 上文中提到的模板, 其实作用就是忽略掉了图片的所有不透明的颜色信息, 取而代之的是它所在的控件的`tintColor`. 

### **2. 应用**

根据上面对**renderingMode**的描述, 我们就可以很容易联想到导致文章开头那个现象的原因:
- UITabBar会自动将图片当做模板来渲染
- UITabBar默认的`tintColor`是系统的亮蓝色

所以, 相应的解决方法也有两种:
**1.设置图片的renderingMode为`UIImageRenderingModeAlwaysOriginal`**
```objc
UIImage *selectedImage = [UIImage imageNamed:selectedImageName];
originalSelectedImage = [selectedImage imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal];

viewController.tabBarItem.selectedImage = originalSelectedImage;
```
**2.设置UITabBar的`tintColor`为我们所要的颜色.**
```objc
tabBar.tintColor = kAppMainColor;
```

### **3. 其他**

如果总是在代码中设置**renderingMode**也是比较麻烦的, 还有一个更佳便捷的设置方法, 如下图在`Images.xcassets`里面选中相应的图片, 在右侧的工具栏中的`Render As`字段选择相应的**renderingMode**就可以了.
![](/img/UIImageRenderingMode/renderingMode_3.png)