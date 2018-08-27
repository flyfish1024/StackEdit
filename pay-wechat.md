
---
---
title: 微信支付总结
date: 2018-08-24 21:20:11
abstract: 微信支付（H5、APP原生）
tags:
- pay
- we chat
header_image: /intro/pay_wechat.jpg
---
---

## H5支付

### 逻辑思路

**官方时序图**
![wechat-h5](https://pay.weixin.qq.com/wiki/doc/api/img/chapter15_1.png)

**流程理解**
1. 用户操作浏览器触发下单操作
2. 商户后台保存本地订单，调用统一下单接口向微信后台下单，返回支付URL（可编入同步回调地址）
3. 浏览器跳转
4. 页面验权，返回结果到浏览器，唤起微信客户端支付
5. 异步通知，微信后台向商户后台发送支付结果，后台商户根据情况向微信后台返回数据
6. 同步回调，支付完成后，微信客户端访问同步回调地址
7. 根据客户选择，查询支付结果
8. 先查询本地订单状态
9. 向微信后台查询订单状态
10. 展示最终支付结果 

**代码逻辑**

1. 下单
	1. 本地下单  
	2. 微信下单
		1. 组织数据，生成sign
			1. 确保服务器域名添加到微信支付后台  
			2. 回调地址前缀为此域名  
		2. http访问，返回跳转的支付界面 url，拼接同步回调  
2. 用户访问，支付
3. 异步回调（注意过滤器）  
	1. 锁定表  
	2. 判断是否当前订单已处理  
	3. 执行业务，修改订单状态，生成日志记录  
4. 同步回调，调起用户支付结果中间页 
	1. 已完成支付，查询订单状态
	2. 未完成，返回支付页面

中间页示例
![中间页](../../../../assets/img/pay_wait.png)

### 代码示例
支付页面为 jsp
```java
/**
 * 微信支付
 * 	参数校验
 * 	生成本地订单
 *  微信下单
 * 		拼接微信数据
 * 		http访问，返回跳转的支付界面 url与同步回调
 * @param session 
 * @param amount 缴纳金额
 * @return
 */
@RequestMapping("/weiXinPay")
public String weiXinPay(HttpServletRequest request,Model model,HttpSession session,String amount){
	//参数校验
	//金额 当前登录状态
	try{
		DecimalFormat decimalFormat = new DecimalFormat("0.00");
		amount = decimalFormat.format(new BigDecimal(amount));
	}catch(Exception e){
		e.printStackTrace();
		model.addAttribute("nRes", 0);
		model.addAttribute("vcRes", "参数非法~");			
	}
	
	//数据不合理
	if(model.containsAttribute("nRes")){
		return "payType";//返回提交页面，提示错误信息
	}
			
	//生成本地订单
	String vcOrderId = UUID.generateSequenceNo();
	boolean addTempOrderMark = payService.addTempOrder(vcOrderId,vcUserTel,vcUserTel,
			new BigDecimal(amount), "党费缴纳",vcSource,"微信",amount);//本地下单，代码就不展示了
	if(!addTempOrderMark){
		//本地订单生成失败
		model.addAttribute("nRes", 0);
		model.addAttribute("vcRes", "网络异常~");
		return "payType";//返回提交页面，提示错误信息
	}
	
	//拼接微信数据
	SortedMap<String, Object> parameterMap = new TreeMap<String, Object>();
	parameterMap.put("key", wx_api_key);//密钥
	parameterMap.put("appid", wx_app_id);//appid
	parameterMap.put("mch_id", wx_mch_id);//商户号
	String nonce_str =  WebUtil.getRandomStr();
	parameterMap.put("nonce_str",nonce_str);
	parameterMap.put("sign_type", "MD5");
	parameterMap.put("body", "缴纳");
	parameterMap.put("out_trade_no", vcOrderId);//商户系统 本地订单号
	parameterMap.put("fee_type", "CNY");
	BigDecimal total = new BigDecimal(amount).multiply(new BigDecimal(100));
	java.text.DecimalFormat df = new java.text.DecimalFormat("0");
	parameterMap.put("total_fee", df.format(total));
	String ip = WebUtil.getIp(request);//获取调用者的ip
	parameterMap.put("spbill_create_ip", ip);
	parameterMap.put("notify_url", SysSetting.wx_notify_url);
	parameterMap.put("trade_type", "MWEB");//这个是文档 写死的
	
	String sign = WXPayUtil.createSign(parameterMap);
	parameterMap.put("sign", sign);
	
	//http访问，返回跳转的支付界面 
	String requestXML = WXPayUtil.getRequestXml(parameterMap);
	String result = WXPayUtil.httpsRequest("https://api.mch.weixin.qq.com/pay/unifiedorder", "POST", requestXML);
	Map<String, String> map = new HashMap<String,String>();
	try {
		map = WXPayUtil.doXMLParse(result);
	} catch (Exception e) {
		e.printStackTrace();
		model.addAttribute("nRes", 0);
		model.addAttribute("vcRes", "网络信息转换异常~");
		return "payType";//返回提交页面，提示错误信息
	}
	
	String result_code = map.get("result_code");
	if("SUCCESS".equals(result_code)){
		//url与同步回调
		String redirectUrl = SysSetting.wx_return_url+"?out_trade_no="+vcOrderId;
		return "redirect:"+map.get("mweb_url")+"&redirect_url="+URLEncoder.encode(redirectUrl);
	}else{
		model.addAttribute("nRes", 0);
		model.addAttribute("vcRes", "网络信息异常~");
		return "payType";//返回提交页面，提示错误信息
	}
}
```

### 注意点

## APP原生支付

### 逻辑思路
**官方时序图**
**流程理解**
**代码逻辑**

### 代码示例
```java

Controller
/**
 * 微信支付-下单
 * @param amount 支付金额
 * @return
 */
@RequestMapping("/weiXinPay")
@ResponseBody
public AppResult weiXinPay(HttpServletRequest request,HttpSession session,String amount){
	//参数校验
	try{
		DecimalFormat decimalFormat = new DecimalFormat("0.00");
		amount = decimalFormat.format(new BigDecimal(amount));
	}catch(Exception e){
		e.printStackTrace();
		return AppResult.validateFail();//AppResult 为自己封装的返回值类
	}
	//执行业务
	try {
		AppResult weChatPay = weChatPayService.weChatPay(request, session, new BigDecimal(amount));
		return weChatPay;
	} catch (Exception e) {
		e.printStackTrace();
		return AppResult.failMessage("下单失败~");
	}
}

Service
@Override
@Transactional
public AppResult weChatPay(HttpServletRequest request,HttpSession session,BigDecimal amount) {
	String vcUserTel = WebUtil.getUserId(session);
	//1. 生成本地订单
	String vcOrderNo = payService.addPayLog("微信",amount,session);
	//2. 微信下单
	//2.1 拼接微信数据
	SortedMap<String, Object> parameterMap = new TreeMap<String, Object>();
	parameterMap.put("key", SysSetting.wx_api_key);//密钥
	parameterMap.put("appid", SysSetting.wx_app_id);//appid
	parameterMap.put("mch_id",SysSetting. wx_mch_id);//商户号
	String nonce_str =  WebUtil.getRandomStr();
	parameterMap.put("nonce_str",nonce_str);
	parameterMap.put("sign_type", "MD5");
	parameterMap.put("body", "自费充值");
	parameterMap.put("out_trade_no", vcOrderNo);//商户本地订单号
	parameterMap.put("fee_type", "CNY");
	BigDecimal total = amount.multiply(new BigDecimal(100));
	java.text.DecimalFormat df = new java.text.DecimalFormat("0");
	parameterMap.put("total_fee", df.format(total));
	String ip = WebUtil.getIp(request);//获取调用者的ip
	parameterMap.put("spbill_create_ip", ip);//ip
	parameterMap.put("notify_url", SysSetting.wx_notify_url);
	parameterMap.put("trade_type", "APP");
	//2.2 生成下单sign
	String sign = WXPayUtil.createSign(parameterMap);
	parameterMap.put("sign", sign);
	
	//2.3 http访问、解析返回值，获取 prepay_id
	String requestXML = WXPayUtil.getRequestXml(parameterMap);//WXPayUtil 封装工具类，下面放代码
	String result = WXPayUtil.httpsRequest("https://api.mch.weixin.qq.com/pay/unifiedorder", "POST", requestXML);
	Map<String, String> map = new HashMap<String,String>();
	try {
		map = WXPayUtil.doXMLParse(result);
	} catch (DocumentException | IOException e) {
		e.printStackTrace();
		throw new RuntimeException("xml数据转换Map异常~");
	}
	//3. 生成新的数据组
	SortedMap<String,Object> parameters = new TreeMap<String,Object>();
	String noncestr = WebUtil.getRandomStr();
	String timestamp = String.valueOf(new Date().getTime() / 1000);
	parameters.put("appid", SysSetting.wx_app_id);
	parameters.put("partnerid", SysSetting.wx_mch_id);
	parameters.put("prepayid", map.get("prepay_id"));
	parameters.put("package", "Sign=WXPay");
	parameters.put("noncestr", noncestr);
	parameters.put("timestamp", timestamp );
	parameters.put("key", SysSetting.wx_api_key );
	//3.1 生成 app签名
	String signApp = WXPayUtil.createSign(parameters);
	parameters.put("sign", signApp );
	parameters.put("vcOrderNo", vcOrderNo);
	
	return AppResult.build("本地下单成功~", 1, parameters);
}
```
### 注意点
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE2MjgwOTQzNDJdfQ==
-->