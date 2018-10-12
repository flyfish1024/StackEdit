---
---
title: 九月问题记录
date: 2018-10-12 10:51:14
abstract: 每月解决的错误集锦（18年9月）
tags:
- problem
header_image: /intro/problem-september.jpg
---
---

### Html上按钮IOS无法点击，安卓可以

**问题原因**

```txt
a) 用jquery的live方法绑定的click事件点击无效
```

**解决办法**

```txt
a) 将 click 事件直接绑定到目标元素（即 .target）上
b) 将目标元素换成 <a> 或者 button 等可点击的元素
c) 将 click 事件委托到非 document 或 body 的父级元素上
d) 给目标元素加一条样式规则 cursor: pointer
荐后两种。从解决办法来看，推测在 safari 中，不可点击的元素的点击事件不会冒泡到父级元素。
通过添加 cursor: pointer 使得元素变成了可点击的了。
```

###  spring错误<context:property-placeholder>：Could not resolve placeholder XXX in string value XXX


**问题原因**

```txt
明明配置了properties来加载db配置文件的数据，但是spring提示无法找到对应的key

Spring容器采用反射扫描的发现机制，在探测到Spring容器中有一个org.springframework.beans.factory.config.PropertyPlaceholderConfigurer的Bean就会停止对剩余PropertyPlaceholderConfigurer的扫描
（Spring 3.1已经使用PropertySourcesPlaceholderConfigurer替代PropertyPlaceholderConfigurer了）。 

而<context:property-placeholder/>这个基于命名空间的配置，其实内部就是创建一个PropertyPlaceholderConfigurer Bean而已。
换句话说，即Spring容器仅允许最多定义一个PropertyPlaceholderConfigurer(或<context:property-placeholder/>)，其余的会被Spring忽略掉。 
```

**解决办法**

```txt
a) <context:property-placeholder location="classpath:properties/db.properties,classpath:properties/mongodb.properties"/>privileged=true”）
b）见图1
```
![图1](../../../../assets/img/problem-september-1.png)

### nginx 出现504 Gateway Time-out

**问题原因**

```txt
a) 一般是由于程序执行时间过长导致响应超时，例如程序需要执行90秒，而nginx最大响应等待时间为30秒，这样就会出现超时。 
1. 程序在处理大量数据，导致等待超时。 
2. 程序中调用外部请求，而外部请求响应超时。 
3. 连接数据库失败而没有停止，死循环重新连。
```

**解决办法**

```txt
a) Sql查询时间过长，调整数据结构，添加字段
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTcwMDUzMzA1XX0=
-->