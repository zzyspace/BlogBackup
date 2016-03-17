title: 【iOS开发】Apple Pay 的集成
date: 2016-02-18 21:19:14
categories: iOS开发
tags: [Apple Pay]
---
![](/img/ApplePay/header.jpg)
Apple Pay在今天终于在中国大陆地区正式开通. Apple Pay不仅仅在线下支付中可以让你通过指纹刷(zhuang)卡(bi), 更可以在线上支付(App中支付)时, 节省掉很多多余的步骤, 让支付宝与微信支付哭晕在厕所... 蛋扯远了, 接下来我们来看看怎么让你的App也支持Apple Pay吧!

<!--more-->

# **App集成Apple Pay**
---
### **1. 创建Merchant ID**
在iOS开发者中心的[Certificates, Identifiers & Profiles页面](https://developer.apple.com/account/ios/identifiers/merchant/merchantCreate.action)中创建一个Merchant ID. (PS:前提是你得有开发者帐号= =)
`Description`: 这个ID的描述, 建议填写App名称.
`ID`: Merchent ID的标识符, 一般填写"merchant.`BundleID`", 比如`merchant.com.apple.passbook`.

### **2. 工程中配置Apple Pay**
你的开发者帐号拥有Merchant ID之后, 就需要在工程中配置你刚刚创建的Merchant ID.
![](/img/ApplePay/image_1.png)
按照如图顺序即可在你的工程中激活你的Merchant ID.

---
**未完待续...**
---

Apple Pay需要在支付提供商上面注册商户. 目前Apple Pay支持的提供商有
1. [CUP](https://open.unionpay.com/ajweb/product/detail?id=80) (中国银联) 
2. [Lianlian Pay](https://apple.lianlianpay.com/OpenPlatform/) (连连支付) 
3. [PayEase](https://www.beijing.com.cn/product/ApplePay_ch.jsp) (首信易支付) 
4. [YeePay](https://www.yeepay.com/article/specialActivities/queryArticle/56c676b814a6d961550c90eb) (易宝支付) 


如果你的App要上架App Store的话, 在苹果官方[App Store Review Guidelines](https://developer.apple.com/app-store/review/guidelines/#apple-pay)中关于Apple Pay的条款需要特别注意:
> 29.1 使用Apple Pay的应用程序必须在出售任何商品或者服务之前为用户提供所有材料的购买信息，否则将会被拒绝。使用Apple Pay的应用程序提供多次付款的，至少要公开再次支付的时间长度，和这种状态将持续到取消为止的每一个时期需要用户提供什么，和将产生的费用，以及如何取消。

> 29.2 使用Apple Pay的应用程序必须正确使用 Apple Pay Human Interface Guidelines 中的Apple Pay标识和用户界面元素，否则将会被拒绝。
 
> 29.3 使用Apple Pay的应用程序不能提供触犯任何领域范围法律的用于交付的商品或者服务，也不能用作任何非法目的。
 
> 29.4 使用Apple Pay的应用程序必须提供隐私政策，否则将会被拒绝。 
 
> 29.5 只有为了促进或提高商品和服务的交付，或者依照法律要求，使用Apple Pay的应用程序才能与第三方分享通过Apple Pay获得的数据

# **设计规范**
---
1. [Apple Pay 标识指南](https://developer.apple.com/apple-pay/Apple-Pay-Identity-Guidelines.pdf)
2. [Apple Pay 图标资源包](https://developer.apple.com/services-account/download?path=/ios/apple_pay_resources/Apple_Pay_Resources.zip)