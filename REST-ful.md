---
---
title: REST风格特点与RESTful-api的使用
date: 201-01-15 22:35:11
abstract: 由繁到简地总结何为REST，如何编写REST-ful风格的代码与架构
tags:
- REST
- REST-ful
- API
header_image: /intro/RESTAPI.jpg
---
---

**Outline**

* 何为REST
	* REST架构风格的特点
	* ROA、SOA、REST与RPC
	* 本真REST与hybrid风格
* 如何REST-ful
	-   Request 和 Response
	-   Serialization 和 Deserialization
	-   Validation
	-   Authentication 和 Permission
	-   CORS
	-   URL Rules
- 总结
	- 看Url就知道要操作的资源是什么，是操作车辆还是围栏
	- 看Http Method就知道操作动作是什么，是添加（post）还是删除（delete）
	- 看Http Status Code就知道操作结果如何，是成功（200）还是内部错误（500）
	-  URL定位资源，用HTTP动词（GET,POST,DELETE,DETC）描述操作

# 何为REST

RESTful架构风格最初由Roy T. Fielding（HTTP/1.1协议专家组负责人）在其2000年的博士学位论文中提出。HTTP就是该架构风格的一个典型应用。从其诞生之日开始，它就因其可扩展性和简单性受到越来越多的架构师和开发者们的青睐。一方面，随着云计算和移动计算的兴起，许多企业愿意在互联网上共享自己的数据、功能；另一方面，在企业中，RESTful API（也称RESTful Web服务）也逐渐超越SOAP成为实现SOA的重要手段之一。时至今日，RESTful架构风格已成为企业级服务的标配。

	REST即Representational State Transfer的缩写，可译为"表现层状态转化”。REST最大的几个特点为：资源、统一接口、URI和无状态。

## RESTful架构风格的特点

### 资源

所谓"资源"，就是网络上的一个实体，或者说是网络上的一个具体信息。它可以是一段文本、一张图片、一首歌曲、一种服务，总之就是一个具体的实在。资源总要通过某种载体反应其内容，文本可以用txt格式表现，也可以用HTML格式、XML格式表现，甚至可以采用二进制格式；图片可以用JPG格式表现，也可以用PNG格式表现；JSON是现在最常用的资源表示格式。

结合我的开发实践，我对资源和数据理解如下：

资源是以json(或其他Representation)为载体的、面向用户的一组数据集，资源对信息的表达倾向于概念模型中的数据：

-   资源总是以某种Representation为载体显示的，即序列化的信息
-   常用的Representation是json(推荐)或者xml（不推荐）等
-   Representation 是REST架构的表现层

相对而言，数据（尤其是数据库）是一种更加抽象的、对计算机更高效和友好的数据表现形式，更多的存在于逻辑模型中

### 统一接口

RESTful架构风格规定，数据的元操作，即CRUD(create, read, update和delete,即数据的增删查改)操作，分别对应于HTTP方法：GET用来获取资源，POST用来新建资源（也可以用于更新资源），PUT用来更新资源，DELETE用来删除资源，这样就统一了数据操作的接口，仅通过HTTP方法，就可以完成对数据的所有增删查改工作。

即：

-   GET（SELECT）：从服务器取出资源（一项或多项）。
-   POST（CREATE）：在服务器新建一个资源。
-   PUT（UPDATE）：在服务器更新资源（客户端提供完整资源数据）。
-   PATCH（UPDATE）：在服务器更新资源（客户端提供需要修改的资源数据）。
-   DELETE（DELETE）：从服务器删除资源。

### URI

可以用一个URI（统一资源定位符）指向资源，即每个URI都对应一个特定的资源。要获取这个资源，访问它的URI就可以，因此URI就成了每一个资源的地址或识别符。

一般的，每个资源至少有一个URI与之对应，最典型的URI即URL。

### 无状态

所谓无状态的，即所有的资源，都可以通过URI定位，而且这个定位与其他资源无关，也不会因为其他资源的变化而改变。有状态和无状态的区别，举个简单的例子说明一下。如查询员工的工资，如果查询工资是需要登录系统，进入查询工资的页面，执行相关操作后，获取工资的多少，则这种情况是`有状态`的，因为查询工资的每一步操作都依赖于前一步操作，只要前置操作不成功，后续操作就无法执行；如果输入一个url即可得到指定员工的工资，则这种情况是`无状态`的，因为获取工资不依赖于其他资源或状态，且这种情况下，员工工资是一个资源，由一个url与之对应，可以通过HTTP中的`GET`方法得到资源，这是典型的RESTful风格。
## ROA、SOA、REST与RPC

`ROA`即Resource Oriented Architecture，RESTful 架构风格的服务是围绕`资源`展开的，是典型的ROA架构（虽然“A”和“架构”存在重复，但说无妨），虽然ROA与SOA并不冲突，甚至把ROA看做SOA的一种也未尝不可，但由于RPC也是SOA，比较久远一点点论文、博客或图书也常把SOA与RPC混在一起讨论，因此，RESTful 架构风格的服务通常被称之为ROA架构，很少提及SOA架构，以便更加显式的与RPC区分。

RPC风格曾是Web Service的主流，最初是基于XML-RPC协议（一个远程过程调用（remote procedure call，RPC)的分布式计算协议），后来渐渐被SOAP协议（简单对象访问协议（Simple Object Access Protocol））取代；RPC风格的服务，不仅可以用`HTTP`，还可以用`TCP`或其他通信协议。但RPC风格的服务，受开发服务采用语言的束缚比较大，如.NET框架中，开发web service的传统方式是使用WCF，基于WCF开发的服务即RPC风格的服务，使用该服务的客户端通常要用C#来实现，如果使用python或其他语言，很难实现可以直接与服务通信客户端；进入移动互联网时代后，RPC风格的服务很难在移动终端使用，而RESTful风格的服务，由于可以直接以`json`或`xml`为载体承载数据，以HTTP方法为统一接口完成数据操作，客户端的开发不依赖于服务实现的技术，移动终端也可以轻松使用服务，这也加剧了REST取代RPC成为web service的主导。

RPC与RESTful的区别如下面两个图所示：

![blog-post-REST-vs-RPC1](https://gevin-zone.igevin.info/blog-post-rest-RPC-service.png "blog-post-REST-vs-RPC1")

![blog-post-REST-vs-RPC2](https://gevin-zone.igevin.info/blog-post-rest-RESTful-service.png "blog-post-REST-vs-RPC2")

## 本真REST与hybrid风格

通常开发者做服务相关的客户端开发时，使用的所谓RESTful服务，基本可分为`本真REST`和`hybrid风格`两类。`本真REST`即我上文阐述的RESTful架构风格，具有上述的4个特点，是真正意义上的RESTful风格；而`hybrid风格`，只是借鉴了RESTful的一些优点，具有一部分RESTful的特点，但对外依然宣称是RESTful风格的服务。（窃以为，正是由于hybrid风格服务混淆了RESTful的概念，才在RESTful架构风格提出了本真REST的概念，以为了划分界限 :P）

hybrid风格的最主流的用法是，使用`GET`方法获取资源，用`POST`方法实现资源的创建、修改和删除。hybrid风格之所以存在，据我了解有两种来源：一种情况是因为，某些开发者并没有真正理解何为RESTful架构风格，导致开发的服务貌合神离；而主流的原因是由于历史包袱 —— 服务本来是RPC风格的，由于上文提到的RPC的劣势及RESTful的优势，开发者在RPC风格的服务上又包装了一层RESTful的外壳，通常这层外壳只为获取资源服务，因此会按RESTful风格实现`GET`方法，如果客户端提出一些简单的创建、修改或删除数据的需求，则通过HTTP协议中最常用的`POST`方法实现相应功能。

因此，开发RESTful 服务，如果没有历史包袱，不建议使用hybrid风格。




<!--stackedit_data:
eyJoaXN0b3J5IjpbODM5NDA4NTQyLC05NDY5ODE1MDVdfQ==
-->