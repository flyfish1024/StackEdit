---
---
title: MyBatis  insert时获取自增主键
date: 2018-08-06 20:21:44
abstract: 使用mybatis插入数据时，需要拿到新的条目的主键，这里整理下三种方式。
tags:
- mybatis
- primarykey
header_image: /intro/mybatis_getid.jpg
---
---

1. 设置insert标签属性`useGeneratedKeys`
2. sql语句插入新标签`<selectKey>`
3. 手动生成唯一主键

## 方法一：设置insert标签属性`useGeneratedKeys`

在mapper.xml中为inser添加两个属性：

```xml
<insert id="insert" useGeneratedKeys="true" keyProperty="nId" parameterType="cn.xxx.xxx.User">
	insert ...
</insert>
```
**keyProperty：指明主键**
**useGeneratedKeys：启用jdbc自增机制，并将数据赋值给主键**

当调用完`insert`之后，传入的User对象的id属性已经自动赋值。可通过get方法获取新的主键。
> 注：此种方法不适用oracle



## 方法二：sql语句加入新标签`<selectKey>`

在mapper.xml中为`insert`内的sql语句前加入`selectKey`标签

```xml
<!-- mysql,order为AFTER -->
<insert id="insert" parameterType="cn.xxx.xxx.User" > 
	<selectKey keyProperty="nId" resultType="java.lang.Long"  order="AFTER">
		SELECT LAST_INSERT_ID()
	</selectKey> 
	insert into User(id, name, age) values(#{id}, #{name}, #{age}) 
</insert>

<!-- oracle,order为BEFORE -->
<insert id="insert" parameterType="cn.xxx.xxx.User" > 
	<selectKey keyProperty="nId" resultType="java.lang.Long"  order="AFTER">
		select user_seq.nextval from dual
	</selectKey> 
	insert into User(id, name, age) values(#{id}, #{name}, #{age}) 
</insert>
```

**LAST_INSERT_ID()**：MYSQL中获取最新的增长id
**user_seq.nextval**：序列自增，并返回新值
**order="AFTER"**：如果设置为 BEFORE,那么它会首先选择主键,设置 keyProperty 然后执行插入语句。如果设置为 AFTER,那么先执行插入语句,然后是 selectKey 元素，这也是上面mysql与oracle的区别。

## 方法三：手动生成唯一主键

这个网上有许多工具类，我这里贴个自己在用的。

```java

public class UUID {
	//机器码
	private static final String machineNum = SysSetting.sys_machineNum;
    /** The FieldPosition. */
    private static final FieldPosition HELPER_POSITION = new FieldPosition(0);
 
    /** This Format for format the data to special format. */
    private final static Format dateFormat = new SimpleDateFormat("yyMMddHHmmssS");
 
    /** This Format for format the number to special format. */
    private final static NumberFormat numberFormat = new DecimalFormat("00");
 
    /** This int is the sequence number ,the default value is 0. */
    private static int seq = 0;
 
    private static final int MAX = 99;
    
	 /**
     * 时间格式生成序列
     * @return String
     */
    public static synchronized String generateSequenceNo() {
 
        Calendar rightNow = Calendar.getInstance();
 
        StringBuffer sb = new StringBuffer();
 
        dateFormat.format(rightNow.getTime(), sb, HELPER_POSITION);
        numberFormat.format(seq, sb, HELPER_POSITION);
 
        if (seq == MAX) {
            seq = 0;
        } else {
            seq++;
        }
        return machineNum + sb.toString();
    }
}
```


> 有疑问与瑕疵请联系我~


<!--stackedit_data:
eyJoaXN0b3J5IjpbMjA0NzcxOTUzOF19
-->