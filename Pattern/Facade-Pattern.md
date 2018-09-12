---
---
title: (二)JAVA设计模式之外观模式-Facade Pattern
date: 2018-09-07 19:21:20
abstract: 设计模式系列第二篇，介绍结构型模式-外观模式的概念，细节，案例
tags:
- Pattern
- Facade Pattern
header_image: /intro/facade_pattern.jpg
---
---

## 概念

外观模式，一般用在子系统与访问之间，用于对访问屏蔽复杂的子系统调用，采用耳目一新的外观类提供的简单的调用方法，具体的实现由外观类去子系统调用。

外观模式仍然是一种中间件类型的模式，使用外观模式之后子系统的方法调用并非完全屏蔽，只是为访问者提供了一种更佳的访问方式，如果你不嫌麻烦，任然可以直接进行子系统方法调用。

甚至于在子系统与子系统之间进行调用时也可以通过各自的外观类来进行调用，这样代码方便管理。

概念理解举例：电脑使用
- 电脑整机是 CPU、内存、硬盘的外观。有了外观以后，启动电脑和关闭电脑都简化了。
- 直接 new 一个电脑。
- 在 new 电脑的同时把 cpu、内存、硬盘都初始化好并且接好线。
- 对外暴露方法（启动电脑，关闭电脑）。
- 启动电脑（按一下电源键）：启动CPU、启动内存、启动硬盘
- 关闭电脑（按一下电源键）：关闭硬盘、关闭内存、关闭CPU

## 案例

```java
//子系统方法1
public class SubMethod1 {
	public void method1(){        
	    System.out.println("子系统中类1的方法1");
	}
}

//子系统方法2
public class SubMethod2 {
	public void method2(){
		System.out.println("子系统中类2方法2");
	} 
}

//子系统方法3
public class SubMethod3 {
	public void method3(){
		System.out.println("子系统类3方法3");
	}
}

//外观类：
public class Facader {  
    private SubMethod1 sm1 = new SubMethod1();  
    private SubMethod2 sm2 = new SubMethod2();  
    private SubMethod3 sm3 = new SubMethod3();  
    public void facMethod1(){  
        sm1.method1();  
        sm2.method2();  
    }  
    public void facMethod2(){  
        sm2.method2();  
        sm3.method3();  
        sm1.method1();  
    }  
}
//测试类：
public class Clienter {  
    public static void main(String[] args) {  
        Facader face = new Facader();  
        face.facMethod1();  
        face.facMethod2();  
    }
}
```
  
其实直接调用也会得到相同的结果，但是采用外观模式能规范代码，外观类就是子系统对外的一个总接口，我们要访问子系统是，直接去子系统对应的外观类进行访问即可！

## 应用场景
当我们访问的子系统拥有复杂额结构，内部调用繁杂，初接触者根本无从下手时，不凡由资深者为这个子系统设计一个外观类来供访问者使用，统一访问路径（集中到外观类中），将繁杂的调用结合起来形成一个总调用写到外观类中，之后访问者不用再繁杂的方法中寻找需要的方法进行调用，直接在外观类中找对应的方法进行调用即可。

还有就是在系统与系统之间发生调用时，也可以为被调用子系统设计外观类，这样方便调用也，屏蔽了系统的复杂性。

## 注意事项
>外观模式的目的不是给予子系统添加新的功能接口，而是为了让外部减少与子系统内多个模块的交互，松散耦合，从而让外部能够更简单地使用子系统。
>外观模式的本质是：封装交互，简化调用。



<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyNTg4MjEzOTZdfQ==
-->