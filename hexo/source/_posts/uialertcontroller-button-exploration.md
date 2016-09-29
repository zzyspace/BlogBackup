title: 【iOS 开发】UIAlertController 的那点事儿
date: 2016-09-28 21:55:25
categories: iOS开发
tags: [UIAlertController, UIAlertView]
---

本来一直很愉快地用着 UIAlertController, 一直到有一天, 测试妹纸把两台测试机摔在我面前, 一刀架在我脖子上质问我: 这两台手机的弹窗怎么一个是 Cancel 粗体, 一个是 OK 粗体?! 给我改! 本着~~小命为重~~与测试妹纸和谐发展的原则与理念, 我开始探究 UIAlertController 这个奇葩的现象...

<!--more-->

# UIAlertController
---

先来说说一个常规的 Alert 风格的 UIAlertController 在各个系统版本中的表现是怎么样的 (UIAlertView 的表现与 UIAlertController 相同):

```objc
// 创建 UIAlertController 与按钮
UIAlertController *alert = [UIAlertController alertControllerWithTitle:@"Title"
                                                               message:@"message"
                                                        preferredStyle:UIAlertControllerStyleAlert];
UIAlertAction *cancelAction = [UIAlertAction actionWithTitle:@"Cancel"
                                                       style:UIAlertActionStyleCancel
                                                     handler:nil];
UIAlertAction *okAction = [UIAlertAction actionWithTitle:@"OK"
                                                   style:UIAlertActionStyleDefault
                                                 handler:nil];
UIAlertAction *otherAction = [UIAlertAction actionWithTitle:@"Other"
                                                      style:UIAlertActionStyleDefault
                                                    handler:nil];
                                                 
// 添加按钮
[alert addAction:cancelAction];
[alert addAction:okAction];
/*
[alert addAction:otherAction]; // 三个按钮的情况
*/

// 显示 UIAlertController
[self presentViewController:alert animated:YES completion:nil];
```

- **iOS 7.0 - iOS 8.2 :**

<div style="text-align: left; float: left; vertical-align: top;width: 50%;"><img src="/img/alert-exploration/2Button_less_than_8_2.png"/>
</div><div style="text-align: left;  display: inline-block; vertical-align: top; width: 50%;"><img src="/img/alert-exploration/3Button.png"/>
</div>

- **iOS 8.3+ :**

<div style="text-align: left; float: left; vertical-align: top;width: 50%;"><img src="/img/alert-exploration/2Button_greater_then_8_3.png"/>
</div><div style="text-align: left;  display: inline-block; vertical-align: top; width: 50%;"><img src="/img/alert-exploration/3Button.png"/>
</div>


根据上面的测试, UIAlertController 的 `UIAlertActionStyleCancel` 类型按钮或 UIAlertView 的 `cancelButton` (后面统称为 `取消按钮`) 的行为表现可以总结为下表:

| | 底部2个Button | 底部3个Button及以上 |
| ---- | ---- | ---- |
| iOS 7.0 - iOS 8.2 | 左方 \ 细体 | 最下方 \ 粗体 |
| iOS 8.3 + | 左方 \ 粗体 | 最下方 \ 粗体 |

# 矛盾
---

上面提到三个及以上按钮的情况下, 取消按钮一直是在最下方且加粗这个没有什么问题了. 但是两个按钮的情况下, 可能就会有一些矛盾:

### 需求1:
假如你家产品对弹窗的要求是这样的: **确认按钮一定要是粗体**因为用户有时候看都不看就往加粗的按钮点. 那么你只能根据系统的版本来分别创建弹窗: iOS 8.3 及以上确认的文字和逻辑在取消按钮下进行, iOS 8.3以下的按照原逻辑... 或者直接用第三方的弹窗库.

### 需求2:
假如你家产品对弹窗的要求更高了: **确认按钮一定要是粗体且在右边**. ~~那么你只能辞职了.~~ iOS 8.3以下是符合这个需求的. 但是 iOS 8.3 及以上怎么办呢? iOS 9 之后 UIAlertController 新增了一个属性

```objc
@property (nonatomic, strong, nullable) UIAlertAction *preferredAction NS_AVAILABLE_IOS(9_0);
```

右边的按钮设置了这个属性之后, 这个按钮就会变成粗体的. 这样就能使 iOS 7.0 - iOS 8.2 和 iOS 9.0+ 的弹窗符合需求了. 那么 iOS 8.3 与 iOS 8.4 怎么办? ~~我也不知道~~目前似乎无法很好的做到, 如果你有好的方案, 欢迎留言交流.


# 小结
---

### 弹窗有三个及以上按钮的情况下:
- **iOS 7.0 之后所有系统版本**的, 取消按钮一直是在最下方且加粗.

### 弹窗有两个按钮的情况下:
- **iOS 8.3 以前**, **取消按钮可以根据添加的顺序决定其位置, 但左方的按钮始终为细体, 右方的按钮始终为粗体.** 
- **iOS 8.3 及以后**, 苹果统一了弹窗中取消按钮的规则: **无论取消按钮添加的顺序如何, 取消按钮始终在左方且为粗体.**
- **iOS 9.0 及以后**, 设置`-setPreferredAction:`之后, 该 Action 就变为加粗的字体, 且取消按钮将不会被加粗. 也就是说, **`-setPreferredAction:`之后, 该 Action 粗体的优先级大于取消按钮.**


