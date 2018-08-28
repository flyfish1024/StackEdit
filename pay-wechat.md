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
支付页面为 payType.jsp
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
	parameterMap.put("trade_type", "MWEB");//官方文档 写死
	
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
/**
 * 微信的异步通知
 * @param request
 * @param httpServletResponse
 * @throws IOException
 */
@RequestMapping("notify")
public void wxpaySucc(HttpServletRequest request,HttpServletResponse httpServletResponse,HttpSession session){
	InputStream inStream;
	ByteArrayOutputStream outSteam = null;
	Map<String, String> params = null;
	try {
		inStream = request.getInputStream();
		outSteam = new ByteArrayOutputStream();
		byte[] buffer = new byte[1024];
		int len = 0;
		while ((len = inStream.read(buffer)) != -1) {
			outSteam.write(buffer, 0, len);
		}
		String resultxml = new String(outSteam.toByteArray(), "utf-8");
		params = new HashMap<String, String>();
		params = WXPayUtil.doXMLParse(resultxml);
		outSteam.close();
		inStream.close();
	} catch (UnsupportedEncodingException e1) {
		// TODO Auto-generated catch block
		e1.printStackTrace();
	} catch (IOException e1) {
		// TODO Auto-generated catch block
		e1.printStackTrace();
	} catch (DocumentException e1) {
		// TODO Auto-generated catch block
		e1.printStackTrace();
	}
	
	String result = "";
	String wx_api_key = null;
	params.put("key", wx_api_key);
	if (!WXPayUtil.isTenpaySign(params)) {
		// 支付失败
		result = "<xml>"+
				  "<return_code><![CDATA[FAIL]]></return_code>"+
				  "<return_msg><![CDATA[OK]]></return_msg>"+
				"</xml>";
				
	} else {
		// ------------------------------
		// 处理业务开始
		// ------------------------------

		String total_fee = params.get("total_fee");
		
		BigDecimal totalFee = new BigDecimal(total_fee);
		BigDecimal totalFee_Yuan = totalFee.divide(new BigDecimal(100));
		
		String orderNo = params.get("out_trade_no"); //商户订单号
		String trxId = params.get("transaction_id"); //微信订单号
		
		payService.notifyDoJob(orderNo, trxId, totalFee_Yuan, "微信");//执行业务操作，锁表，防止多次操作
		//SELECT * FROM `ams_web_alipay_temp` WHERE vcOrderId = ? for UPDATE
		
		result = "<xml>"+
		  "<return_code><![CDATA[SUCCESS]]></return_code>"+
		  "<return_msg><![CDATA[OK]]></return_msg>"+
		"</xml>";
		
	}
	
	try {
		PrintWriter writer = httpServletResponse.getWriter();
		writer.print(result); 
		writer.flush();  
		writer.close();  
	} catch (Exception e) {
		// TODO: handle exception
		e.printStackTrace();
	}
}
/**
 * 微信同步回调
 * @param request
 * @param httpServletResponse
 * @param session
 * @return
 */
@RequestMapping("wapWXReturn")
public String wapWXReturn(HttpServletRequest request,HttpServletResponse httpServletResponse,HttpSession session,String out_trade_no){
	Map<String, Object> orderInfo = payService.getOrderInfo(out_trade_no);
	String exe = orderInfo.get("nExe").toString();
	if("1".equals(exe)){
		return "pay_success";
	}else{
		return "pay_fail";
	}
	
}
//工具类
public class WXPayUtil {

	// 请求xml组装
	public static String getRequestXml(SortedMap<String, Object> parameters) {
		parameters.remove("key");
		StringBuffer sb = new StringBuffer();
		sb.append("<xml>");
		Set<Map.Entry<String, Object>> es = parameters.entrySet();
		Iterator<Map.Entry<String, Object>> it = es.iterator();
		while (it.hasNext()) {
			Map.Entry entry = (Map.Entry) it.next();
			String key = (String) entry.getKey();
			String value = (String) entry.getValue();
			if ("attach".equalsIgnoreCase(key) || "body".equalsIgnoreCase(key) || "sign".equalsIgnoreCase(key)) {
				sb.append("<" + key + ">" + "<![CDATA[" + value + "]]></" + key + ">");
			} else {
				sb.append("<" + key + ">" + value + "</" + key + ">");
			}
		}
		sb.append("</xml>");
		return sb.toString();
	}

	// 生成签名
	public static String createSign(SortedMap<String, Object> parameters) {
		StringBuffer sb = new StringBuffer();
		Set es = parameters.entrySet();
		Iterator it = es.iterator();
		while (it.hasNext()) {
			Map.Entry entry = (Map.Entry) it.next();
			String k = (String) entry.getKey();
			Object v = entry.getValue();
			if (null != v && !"".equals(v) && !"sign".equals(k) && !"key".equals(k)) {
				sb.append(k + "=" + v + "&");
			}
		}
		if(parameters.get("key") != null){
			sb.append("key=" + parameters.get("key"));
		}
		String sign = WebUtil.getMD5Str(sb.toString()).toUpperCase();
		return sign;
	}

	/**
	 * 验证回调签名
	 * 
	 * @param packageParams
	 * @param key
	 * @param charset
	 * @return
	 */
	public static boolean isTenpaySign(Map<String, String> map) {
		String charset = "utf-8";
		String signFromAPIResponse = map.get("sign");
		if (signFromAPIResponse == null || signFromAPIResponse.equals("")) {
			System.out.println("API返回的数据签名数据不存在，有可能被第三方篡改!!!");
			return false;
		}
		//System.out.println("服务器回包里面的签名是:" + signFromAPIResponse);
		// 过滤空 设置 TreeMap
		SortedMap<String, String> packageParams = new TreeMap<>();
		for (String parameter : map.keySet()) {
			String parameterValue = map.get(parameter);
			String v = "";
			if (null != parameterValue) {
				v = parameterValue.trim();
			}
			packageParams.put(parameter, v);
		}

		StringBuffer sb = new StringBuffer();
		Set es = packageParams.entrySet();
		Iterator it = es.iterator();
		while (it.hasNext()) {
			Map.Entry entry = (Map.Entry) it.next();
			String k = (String) entry.getKey();
			String v = (String) entry.getValue();
			if (!"sign".equals(k) && null != v && !"".equals(v) && !"key".equals(k)) {
				sb.append(k + "=" + v + "&");
			}
		}
		sb.append("key=" + packageParams.get("key"));

		// 将API返回的数据根据用签名算法进行计算新的签名，用来跟API返回的签名进行比较
		// 算出签名
		String resultSign = "";
		String tobesign = sb.toString();
		if (null == charset || "".equals(charset)) {
			resultSign = WebUtil.getMD5Str(tobesign).toUpperCase();
		} else {
			resultSign = WebUtil.getMD5Str(tobesign).toUpperCase();
		}
		String tenpaySign = ((String) packageParams.get("sign")).toUpperCase();
		return tenpaySign.equals(resultSign);
	}

	// 请求方法
	public static String httpsRequest(String requestUrl, String requestMethod, String outputStr) {
		try {

			URL url = new URL(requestUrl);
			HttpURLConnection conn = (HttpURLConnection) url.openConnection();

			conn.setDoOutput(true);
			conn.setDoInput(true);
			conn.setUseCaches(false);
			// 设置请求方式（GET/POST）
			conn.setRequestMethod(requestMethod);
			conn.setRequestProperty("content-type", "application/x-www-form-urlencoded");
			// 当outputStr不为null时向输出流写数据
			if (null != outputStr) {
				OutputStream outputStream = conn.getOutputStream();
				// 注意编码格式
				outputStream.write(outputStr.getBytes("UTF-8"));
				outputStream.close();
			}
			// 从输入流读取返回内容
			InputStream inputStream = conn.getInputStream();
			InputStreamReader inputStreamReader = new InputStreamReader(inputStream, "utf-8");
			BufferedReader bufferedReader = new BufferedReader(inputStreamReader);
			String str = null;
			StringBuffer buffer = new StringBuffer();
			while ((str = bufferedReader.readLine()) != null) {
				buffer.append(str);
			}
			// 释放资源
			bufferedReader.close();
			inputStreamReader.close();
			inputStream.close();
			inputStream = null;
			conn.disconnect();
			return buffer.toString();
		} catch (ConnectException ce) {
			System.out.println("连接超时：{}" + ce);
		} catch (Exception e) {
			System.out.println("https请求异常：{}" + e);
		}
		return null;
	}


	// xml解析
	public static Map doXMLParse(String strxml) throws DocumentException,IOException {
		strxml = strxml.replaceFirst("encoding=\".*\"", "encoding=\"UTF-8\"");

		if (null == strxml || "".equals(strxml)) {
			return null;
		}

		Map<String,String> m = new HashMap<String,String>();

		InputStream in = new ByteArrayInputStream(strxml.getBytes("UTF-8"));
		SAXReader reader = new SAXReader();
		Document doc = reader.read(in);
		Element root = doc.getRootElement();
		List<Element> list = root.elements();
		Iterator<Element> it = list.iterator();
		while (it.hasNext()) {
			Element e = (Element) it.next();
			String k = e.getName();
			String v = e.getStringValue();
			m.put(k, v);
		}

		// 关闭流
		in.close();

		return m;
	}

}


```

### 注意点

1. 基本参数不要搞错，商户秘钥、商户号、appid、secret，否则会报sign验证失败，而且找不到问题
2. ip地址获取
3. 异步回调，防止业务流程执行多次
4. 同步回调，加入用户选择页面

## APP原生支付

### 逻辑思路

**官方时序图**
![wechat-h5](https://pay.weixin.qq.com/wiki/doc/api/img/chapter8_3_1.png)

**流程理解**
1. 用户进入app
2. 选择商品，下单
3. 确认支付，向商户后台确认订单
4. 商户后台生成本地订单，再调用微信统一下单接口
5. 商户后台获取微信返回数据（prepay_id）
6. 商户后台生成新的sign，返回app
7. app接收数据，唤起微信客户端
8. 用户与微信app支付流程不用考虑
9. 异步回调：微信后台向商户后台发送支付结果，后台商户根据情况向微信后台返回数据
10. 同步，微信再唤起商户app时，会返回支付状态，根据支付状态去后台查真正有效的支付状态，再展示给用户

**代码逻辑**
1. 下单（事务）
	1.	生成本地订单
	2.	微信统一接口下单
		1. 拼接sign签名下单
		2. 下单成功，生成新的sign交给app
2.	异步回调
	1. 解析xml
	2. 校验sign
		1. 返回失败
	3.	处理本地订单
		1. 是否已处理
		2. 锁定记录
		3. 处理
3. 同步回调
	1. App调用订单支付状态查询接口


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
	parameters.put("vcOrderNo", vcOrderNo);//app同步回调查询支付状态使用
	
	return AppResult.build("本地下单成功~", 1, parameters);
}
```
### 注意点

1. 下单需要生成签名两次，每次生成签名都需要在末尾追加 key
2. 两次签名的数据源依据官网文档补全数据，全小写
3. 因异步回调延迟，app查询订单状态可延迟或循环查询
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIxMzgxODUwNjVdfQ==
-->