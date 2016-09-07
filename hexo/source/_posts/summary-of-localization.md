title: 【iOS Tip】总结本地化的一些坑
date: 2016-08-07 18:31:18
categories: iOS Tips
tags: [localization]
---


# 默认语言
---

如果一台设备的语言为法语, 但是你所做的本地化只有英语\中文\德语, 那么应用安装后会默认显示Info.plist中`Localization native development region`字段所设置的地区语言(如果没有这个字段就默认英语).

另一种情况是应用安装后切换语言, 若切换后的语言不存在本地化文件, 则保留前一语言的本地化显示. 比如你所做的本地化只有英语\中文\德语, 设备从英语切换到法语, 那么应用还是显示为英语; 如果再从法语切换到日语, 应用还是现实为英语. 一直到你切换至英语\中文\德语中的某一个语言之后, 应用才会显示响应的语言.

<!--more-->

如果要自己设置默认语言的话, 可以使用以下的宏:

```objc
// 判断系统语言是否是中文@"zh-Hans"或者英文@"en", 如果不是中文或者英文其他一概用英文@"en"
#define MYLocalizedString(key, comment) \
(([[[NSLocale preferredLanguages] objectAtIndex:0] isEqual:@"zh-Hans"] ||
  [[[NSLocale preferredLanguages] objectAtIndex:0] isEqual:@"en"]) ?
 ([[NSBundle mainBundle] localizedStringForKey:key value:@"" table:nil]) :
 ([[NSBundle bundleWithPath:[[NSBundle mainBundle] pathForResource:@"en" ofType:@"lproj"]] localizedStringForKey:key value:@"" table:nil]))
```

但是由于宏替换的特性, 上面的方法会增加很多的代码量(想象一下你的代码里面有多少本地化字符串, 就会有多少份上面的代码), 作为处女座简直不能忍. 所以想到了hook`-localizedStringForKey:value:table:`方法来实现自定义的选择加载的语言包:

```objc
@implementation NSBundle (DBSwizzle)

+ (void)load
{
    static dispatch_once_t NSBundleOnceToken;
    dispatch_once(&NSBundleOnceToken, ^{
        ZY_SwizzlesMethod([self class], @selector(localizedStringForKey:value:table:), @selector(zy_localizedStringForKey:value:table:));
    });
}

- (NSString *)zy_localizedStringForKey:(NSString *)key
                                 value:(nullable NSString *)value
                                 table:(nullable NSString *)tableName
{
    // 判断系统语言是否是中文@"zh-Hans"或者英文@"en", 如果不是中文或者英文其他一概用英文@"en"
    if ([[[NSLocale preferredLanguages] objectAtIndex:0] isEqual:@"zh-Hans"] ||
        [[[NSLocale preferredLanguages] objectAtIndex:0] isEqual:@"en"])    
    { 
        return [self DB_localizedStringForKey:key value:value table:tableName];
    } else {
        NSString *bundlePath = [[NSBundle mainBundle] pathForResource:@"en" ofType:@"lproj"];
        return [[NSBundle bundleWithPath:bundlePath] DB_localizedStringForKey:key value:value table:tableName];
    }
}

@end
```

# CFBundleDisplayName
---

如果使用以下方法来获取应用名称的话, 是获取不到`InfoPlist.strings`中的本地化应用名称的.

```objc
[[[NSBundle mainBundle] infoDictionary] objectForKey:@"CFBundleDisplayName"];
```
	
可以使用以下方法来获取

```objc
[[[NSBundle mainBundle] localizedInfoDictionary] objectForKey:@"CFBundleDisplayName"];
```

或者更加直接的使用以下方法(此方法会优先取本地化的值, 若没有本地化的值, 则取原值) `推荐`

```objc
[[NSBundle mainBundle] objectForInfoDictionaryKey:@"CFBundleDisplayName"];
```

# UILabel调整字体适应Label宽度
---

通过以下两个属性即可让字体根据UILabel的长度自适应大小

```objc
@property(nonatomic) BOOL adjustsFontSizeToFitWidth;
@property(nonatomic) CGFloat minimumScaleFactor;
```

# 格式化字符串的坑
---

很多情况下, 如果一个需要本地化的字符串中有多个格式化占位符的话, 因为各个语言的语法不同, 会有占位符顺序需要调整的问题. 比如以下栗子:

```objc
[NSString stringWithFormat:@"您的账号%@中仍有%@美元未使用", account, balance];
```

英文本地化字符串替换进去之后变成了

```objc
[NSString stringWithFormat:@"%@ dollars unused in your account %@", account, balance];
```

很明显语意已经跑偏了. 那么如何调整格式化占位符来适配不同的语法呢? 用以下方法即可:

```objc
[NSString stringWithFormat:@"%1$@ dollars unused in your account %2$@", account, balance];
```



