---
---
title: 七月问题记录
date: 2018-08-02 21:19:04
abstract: 每月解决的错误集锦（18年7月）
tags:
- problem
header_image: /intro/problem-july.jpg
---
---

### org.apache.poi.poifs.filesystem.OfficeXmlFileException: The supplied data appears to be in the Office 2007+ XML. You are calling the part of POI that deals with OLE2 Office Documents. You need to call a different part of POI to process this data (eg XSSF instead of HSSF)

**问题原因**

```txt
1. 该错误意思是说，文件中的数据是用Office2007+XML保存的，而现在却调用OLE2 Office文档处理，应该使用POI不同的部分来处理这些数据，比如使用XSSF来代替HSSF。
2. XSSF不能读取Excel2003以前（包括2003）的版本
3. HSSF可以读取2003以前，包括2003
```

**解决办法**

```txt
a) 判断文档后缀（xls/xlsx），灵活调用HSSF或者XSSF API。
```

###  ASM ClassReader failed to parse class file,后面还会提示可能是因为java version不匹配，无法读取class

**问题原因**

```txt
1. Java编译版本，预设编译版本，tomcatjdk版本不一致
2. 要注意tomcat一般与jdk版本一致，也就是7对应7，spring4只支持1.8，spring3对应1.7
```

**解决办法**

```txt
将pom中 spring版本修改为4.0
```
### org.mybatis.generator.exception.XMLParserException: XML Parser Error on line 45: 元素类型为 "context" 的内容必须匹配 "(property*,plugin*,commentGenerator?,jdbcConnection,javaTypeResolver?,javaModelGenerator,sqlMapGenerator?,javaClientGenerator?,table+)"

**问题原因**

```txt
generatorConfig.xml里面context节点必须按顺序配置
```

**解决办法**

```txt
按照提示给的顺序调整标签。
```
### input val() 赋值失败

**问题原因**

```txt
hidde=‘true’后无法通过val赋值，可通过attr
```

**解决办法**

```txt
通过 type=‘hidden’设置隐藏
```
### Invalid bound statement (not found)

**问题原因**

```txt
1. 检查xml文件所在package名称是否和Mapper interface所在的包名
2. UserDao的方法在UserDao.xml中没有，然后执行UserDao的方法会报此错误
3. UserDao的方法返回值是List<User>,而select元素没有正确配置ResultMap,或者只配置ResultType!
4. 看下mapper的XML配置路径是否正确
6. 如果你确认没有以上问题,请任意修改下对应的xml文件,比如删除一个空行,保存.问题解决

```

**解决办法**

```txt
a) xml id与mapper的method大小写错误
b) Xml与mapper的文件名不一致
```

### 无法找到某个Service的Bean，自动注入controller失败

**问题原因**

```txt
未扫描对应包
```

**解决办法**

```txt
注入时使用了实现类声明，改为接口声明
```


<!--stackedit_data:
eyJoaXN0b3J5IjpbODMzMzI2MTA3XX0=
-->