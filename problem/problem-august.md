---
---
title: 八月问题记录
date: 2018-09-03 20:12:31
abstract: 每月解决的错误集锦（18年8月）
tags:
- problem
header_image: /intro/problem-august.jpg
---
---

### 微信APP支付，APP唤起微信客户端失败

**问题原因**

```txt
a) 要生成两次签名，每次签名都需要商户key
b) 大小写
c) 时间戳为秒，需要毫秒除以1000。App接收注意容器大小。
```

**解决办法**

```txt
两次生成签名都需要商户秘钥(key)参数的参与。
```

###  Eclipse开启tomcat报错：Could not publish to the server. java.lang.NullPointerException


**问题原因**

```txt
未知
```

**解决办法**

```txt
a) 找到Tomcat的配置文件“context.xml”，在Context标签中添加两个属性（ reloadable=true” privileged=true”）
b）在自己设置的workspace目录下面，打开目录：.metadata\.plugins\org.eclipse.wst.server.core\
	删除“temp0”文件夹
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE0MDUyOTgxOTldfQ==
-->