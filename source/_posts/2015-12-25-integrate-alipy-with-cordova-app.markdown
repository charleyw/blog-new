---
layout: post
title: "Cordova集成支付宝支付"
date: 2015-12-25 10:17:05 +0800
comments: true
categories: 
---
### 申请帐号
首先你需要从这个网站[支付宝商家页面](https://b.alipay.com/newIndex.htm)申请帐号，然后申请[无线支付](https://b.alipay.com/order/productDetail.htm?productId=2015110218010538)。

中间需要经历实名认证，对公帐号认证等等认证，全部成功之后会收到短信通知告诉你预约成功。

### 上传RSA公钥
* 按照这个[网页](http://doc.open.alipay.com/doc2/detail.htm?spm=0.0.0.0.WSkmo8&treeId=58&articleId=103242&docType=1)的说明生成RSA公钥  
**注意**：在`pkcs8 -topk8 -inform PEM -in rsa_private_key.pem -outform PEM -nocrypt`这一步会将生成的PKCS8格式的私钥打印到屏幕，需要手动保存。之后需要使用。
* 然后把生成好的RSA公钥上传到这个支付宝上，参照这个[网页](http://doc.open.alipay.com/doc2/detail.htm?spm=0.0.0.0.KOQ7dv&treeId=58&articleId=103578&docType=1)

**注意上传的时候记住删掉公钥文件前后的-----BEGIN PUBLIC KEY-----和-----END PUBLIC KEY-----，以及空格和换行，否则会告诉你公钥非法**
<!-- more -->
### 准备好集成资料

* 支付宝帐号
* 合作者身份ID(PID): [查看方法](http://doc.open.alipay.com/doc2/detail?treeId=58&articleId=103544&docType=1)
* 刚才生成的PKCS8格式的私钥：rsa_private_key.pem

### 集成Cordova支付宝插件 [cordova-plugin-alipay](https://github.com/charleyw/cordova-plugin-alipay.git)

以下以Mac系统为例

1. 使用git命令将插件下载到本地，并标记为$CORDOVA_PLUGIN_DIR

		git clone https://github.com/charleyw/cordova-plugin-alipay.git && cd cordova-plugin-alipay && export CORDOVA_PLUGIN_DIR=$(pwd)

2. 修改$CORDOVA_PLUGIN_DIR/plugin.xml，将

		<preference name="private_key" value="$PRIVATE_KEY" />
改成

		<preference name="PRIVATE_KEY" value="你生成的PKCS8格式的私钥"/>

	**注意**：注意和公钥一样，需要去掉-----BEGIN PUBLIC KEY-----和-----END PUBLIC KEY-----，以及空格和换行
3. 安装

		cordova plugin add $CORDOVA_PLUGIN_DIR --variable PARTNER_ID=[合作者身份ID(PID)] --variable SELLER_ACCOUNT=[你的商户支付宝帐号]

安装成功之后剩下的就是编程的问题了。

### 使用方法
```
window.alipay.pay({
	tradeNo: tradeNo,
	subject: "测试标题",
	body: "我是测试内容",
	price: 0.02,
	fromUrlScheme: "demoScheme://afterPaymentSuccess",
	notifyUrl: "http://your.server.notify.url"
});
```
* tradeNo 这个是支付宝需要的，应该是一个唯一的ID号
* subject 这个字段会显示在支付宝付款的页面
* body 订单详情，没找到会显示哪里
* price 价格，支持两位小数
* fromUrlScheme 支付完成跳转的URL Scheme（[iOS](https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Inter-AppCommunication/Inter-AppCommunication.html)，[Android](http://developer.android.com/training/basics/intents/filters.html)），可以使用这个[Cordova插件](https://github.com/EddyVerbruggen/Custom-URL-scheme)定义你的App的Scheme

调用`pay`方法会打开支付宝支付页面进行支付（如果有安装支付宝钱包的话会打开支付宝钱包），支付完成之后会跳回到程序，会跳到`fromUrlScheme`定义的程序页面，
如上面的例子会返回你的程序的`/afterPaymentSuccess`路径所定义的页面，并且支付结果会附加到这个url后面。你可以在程序中调试来确定该怎么处理。

**注意**：因为插件的作者比较懒一直没有加回掉函数（所以需要使用URL Scheme来返回结果），所以使用这个插件的基本条件是：1. 使用URL Scheme， 2. 你的应用有路由或类似的东西来代表页面状态(即可以通过URL跳转到某个页面)，因为支付成功之后会跳转到这个页面。

祝大家好运！
