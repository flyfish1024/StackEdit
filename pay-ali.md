---
---
title: 阿里支付总结
date: 2018-08-28 19:02:44
abstract: 阿里支付（H5、APP原生）
tags:
- pay
- ali
header_image: /intro/pay_ali.jpg
---
---

## H5支付

### 逻辑思路

**官方时序图**
![ali_h5-w1000](https://gw.alipayobjects.com/os/skylark-tools/public/files/112977bc722510a4b846b1544778cafb)

**文字介绍**
-   如上图所示，用户在商户的H5网站下单支付后，商户系统按照[手机网站支付接口alipay.trade.wap.pay](https://docs.open.alipay.com/203/107090)API的参数规范生成订单数据，然后在前端页面通过Form表单的形式请求到支付宝。此时支付宝会自动将页面跳转至支付宝H5收银台页面，如果用户手机上安装了支付宝APP，则自动唤起支付宝APP。开发者需要关注安装了支付宝和未安装支付宝的两种测试场景，对于在手机浏览器唤起H5页面的模式下，如果安装了支付宝却没有唤起，大部分原因是当前浏览器不在支付宝配置的白名单内。
-   对于商户app内嵌webview中的支付场景，建议集成支付宝[App支付](https://docs.open.alipay.com/204/105297)产品。或者您可以使用[手机网站支付转Native支付](https://docs.open.alipay.com/203/106493)的方案，不建议在您的APP中直接接入手机网站支付。
-   用户在支付宝APP或H5收银台完成支付后，会根据商户在手机网站支付API中传入的前台回跳地址return_url自动跳转回商户页面，同时在URL请求中以Query String的形式附带上支付结果参数，详细回跳参数见“手机网站支付接口alipay.trade.wap.pay”[前台回跳参数](https://docs.open.alipay.com/203/107090#s2)。  
    注意：在ios系统中，唤起支付宝App支付完成后，不会自动回到浏览器或商户APP。用户可手工切回到浏览器或商户APP；支付宝H5收银台会自动跳转回商户return_url指定的页面。
-   支付宝还会根据原始支付API中传入的异步通知地址notify_url，通过POST请求的形式将支付结果作为参数通知到商户系统，详情见[支付结果异步通知](https://docs.open.alipay.com/203/105286)。

**代码逻辑**
（webview嵌套需要原生根据url拦截）
1. 下单
	1.	参数校验
	2. 生成本地订单
	3.	阿里下单
2. 同步回调
	1.	校验sign
	2.	根据结果返回页面
3. 异步回调（注意过滤器）
	1. 验签
	2. 保存信息

### 代码示例

```java

```

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTc5NzQ4MTg1Nl19
-->