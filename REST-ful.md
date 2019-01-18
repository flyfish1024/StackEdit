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
	- **网络上的所有事物都被抽象为资源**

**每个资源都有一个唯一的资源标识符**

**同一个资源具有多种表现形式(xml,json等)**

**对资源的各种操作不会改变资源标识符**

**所有的操作都是无状态的**

**符合REST原则的架构方式即可称为RESTful**

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

# 1. Request 和 Response

RESTful API的开发和使用，无非是客户端向服务器发请求（request），以及服务器对客户端请求的响应（response）。[本真RESTful架构风格](http://blog.igevin.info/posts/restful-architecture-in-general/)具有统一接口的特点，即，使用不同的http方法表达不同的行为：

-   GET（SELECT）：从服务器取出资源（一项或多项）
-   POST（CREATE）：在服务器新建一个资源
-   PUT（UPDATE）：在服务器更新资源（客户端提供完整资源数据）
-   PATCH（UPDATE）：在服务器更新资源（客户端提供需要修改的资源数据）
-   DELETE（DELETE）：从服务器删除资源

客户端会基于`GET`方法向服务器发送获取数据的请求，基于`PUT`或`PATCH`方法向服务器发送更新数据的请求等，服务端在设计API时，也要按照相应规范来处理对应的请求，这点现在应该已经成为所有RESTful API的开发者的共识了，而且各web框架的request类和response类都很强大，具有合理的默认设置和灵活的定制性，Gevin在这里仅准备强调一下响应这些request时，常用的Response要包含的数据和状态码（status code），不完善的内容，欢迎大家补充:

-   当`GET`,  `PUT`和`PATCH`请求成功时，要返回对应的数据，及状态码`200`，即SUCCESS
-   当`POST`创建数据成功时，要返回创建的数据，及状态码`201`，即CREATED
-   当`DELETE`删除数据成功时，不返回数据，状态码要返回`204`，即NO CONTENT
-   当`GET`  不到数据时，状态码要返回`404`，即NOT FOUND
-   任何时候，如果请求有问题，如校验请求数据时发现错误，要返回状态码  `400`，即BAD REQUEST
-   当API 请求需要用户认证时，如果request中的认证信息不正确，要返回状态码  `401`，即NOT AUTHORIZED
-   当API 请求需要验证用户权限时，如果当前用户无相应权限，要返回状态码  `403`，即FORBIDDEN

最后，关于Request 和 Response，不要忽略了http header中的Content-Type。以json为例，如果API要求客户端发送request时要传入json数据，则服务器端仅做好json数据的获取和解析即可，但如果服务端支持多种类型数据的传入，如同时支持json和form-data，则要根据客户端发送请求时header中的Content-Type，对不同类型是数据分别实现获取和解析；如果API响应客户端请求后，需要返回json数据，需要在header中添加`Content-Type=application/json`。

# 2. Serialization 和 Deserialization

Serialization 和 Deserialization即序列化和反序列化。RESTful API以规范统一的格式作为数据的载体，常用的格式为`json`或`xml`，以json格式为例，当客户端向服务器发请求时，或者服务器相应客户端的请求，向客户端返回数据时，都是传输json格式的文本，而在服务器内部，数据处理时基本不用json格式的字符串，而是native类型的数据，最典型的如类的实例，即对象（object），json仅为服务器和客户端通信时，在网络上传输的数据的格式，服务器和客户端内部，均存在将json转为native类型数据和将native类型数据转为json的需求，其中，将native类型数据转为json即为`序列化`，将json转为native类型数据即为`反序列化`。虽然某些开发语言，如`Python`，其原生数据类型`list`和`dict`能轻易实现序列化和反序列化，但对于复杂的API，内部实现时总会以对象作为数据的载体，因此，**确保序列化和反序列化方法的实现，是开发RESTful API最重要的一步准备工作**

> _题外话，序列化和反序列化的便捷，造就了RESTful API跨平台的特点，使得REST取代RPC成为Web Service的主流_

序列化和反序列化是RESTful API开发中的一项硬需求，所以几乎每一种常用的开发语言都会有一个或多个优秀的开源库，来实现序列化和反序列化，因此，我们在开发RESTful API时，没必要制造重复的轮子，选一个好用的库即可，如python中的[marshmallow](http://marshmallow.readthedocs.io/en/latest/)，如果基于Django开发，[Django REST Framework](http://www.django-rest-framework.org/)中的`serializer`即可。

# 3. Validation

Validation即数据校验，是开发健壮RESTful API中另一个重要的一环。仍以json为例，当客户端向服务器发出`post`,  `put`或`patch`请求时，通常会同时给服务器发送json格式的相关数据，服务器在做数据处理之前，先做数据校验，是最合理和安全的前后端交互。如果客户端发送的数据不正确或不合理，服务器端经过校验后直接向客户端返回400错误及相应的数据错误信息即可。常见的数据校验包括：

-   数据类型校验，如字段类型如果是int，那么给字段赋字符串的值则报错
-   数据格式校验，如邮箱或密码，其赋值必须满足相应的正则表达式，才是正确的输入数据
-   数据逻辑校验，如数据包含出生日期和年龄两个字段，如果这两个字段的数据不一致，则数据校验失败

以上三种类型的校验，数据逻辑校验最为复杂，通常涉及到多个字段的配合，或者要结合用户和权限做相应的校验。Validation虽然是RESTful API 编写中的一个可选项，但它对API的安全、服务器的开销和交互的友好性而言，都具有重要意义，因此，Gevin建议，开发一套完善的RESTful API时，Validation的实现必不可少。

# 4. Authentication 和 Permission

Authentication指用户认证，Permission指权限机制，这两点是使RESTful API 强大、灵活和安全的基本保障。

常用的认证机制是`Basic Auth`和`OAuth`，RESTful API 开发中，除非API非常简单，且没有潜在的安全性问题，否则，认证机制是必须实现的，并应用到API中去。`Basic Auth`非常简单，很多框架都集成了`Basic Auth`的实现，自己写一个也能很快搞定，`OAuth`目前已经成为企业级服务的标配，其相关的开源实现方案[非常丰富](http://oauth.net/2/)（[更多](https://github.com/search?utf8=%E2%9C%93&q=oauth)）。

我在《[RESTful 架构风格概述](http://blog.igevin.info/posts/restful-architecture-in-general/)》中，对`认证机制`做了更加详细的描述，有兴趣的同学不妨阅读相关章节。

权限机制是对API请求更近一步的限制，只有通过认证的用户符合权限要求，才能访问API。权限机制的具体实现通常依赖于系统的业务逻辑和应用场景，generally speaking，常用的权限机制主要包含`全局型`的和`对象型`的，全局型的权限机制，主要指通过为用户赋予权限，或者为用户赋予角色或划分到用户组，然后为角色或用户组赋予权限的方式来实现权限控制，对象型的权限机制，主要指权限控制的颗粒度在object上，用户对某个具体对象的访问、修改、删除或其行为，要单独在该对象上为用户赋予相关权限来实现权限控制。

全局型的权限机制容易理解，实现也简单，有很多开源库可做备选方案，不少完善的web开发框架，也会集成相关的权限逻辑，object permission 相对难复杂一点，但也有很多典型的应用场景，如多人博客系统中，作者对自己文章的编辑权限即为object permission，其对应的开源库也有很多。

> 注： 我写过一篇《[Django权限机制的实现](http://blog.igevin.info/posts/django-permission/)》，Django 开发者可做延伸阅读。

开发一套完整的RESTful API，权限机制必须纳入考虑范围，虽然权限机制的具体实现依赖于业务，权限机制本身，是有典型的模式存在的，需要开发者掌握基本的权限机制实现方案，以便随时应用到API中去。

# 5. CORS

CORS即[Cross-origin resource sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)，在RESTful API开发中，主要是为js服务的，解决javascript 调用 RESTful API时的跨域问题。

由于固有的安全机制，js的跨域请求时是无法被服务器成功响应的。现在前后端分离日益成为web开发主流方式的大趋势下，后台逐渐趋向指提供API服务，为各客户端提供数据及相关操作，而网站的开发全部交给前端搞定，网站和API服务很少部署在同一台服务器上并使用相同的端口，js的跨域请求时普遍存在的，开发RESTful API时，通常都要考虑到CORS功能的实现，以便js能正常使用API。

目前各主流web开发语言都有很多优秀的实现CORS的开源库，我们在开发RESTful API时，要注意CORS功能的实现，直接拿现有的轮子来用即可。

_更多关于CORS的介绍，有兴趣的同学可以查看阮一峰老师的[跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)_

# 6. URL Rules

RESTful API 是写给开发者来消费的，其命名和结构需要有意义。因此，在设计和编写URL时，要符合一些规范。Url rules 可以单独写一篇博客来详细阐述，本文只列出一些关键点。

## 6.1 Version your API

规范的API应该包含版本信息，在RESTful API中，最简单的包含版本的方法是将版本信息放到url中，如：

```
/api/v1/posts/
/api/v1/drafts/

/api/v2/posts/
/api/v2/drafts/

```

另一种优雅的做法是，使用HTTP header中的`accept`来传递版本信息，这也是GitHub API 采取的[策略](https://developer.github.com/v3/media/#request-specific-version)。

## 6.2 Use nouns, not verbs

RESTful API 中的url是指向资源的，而不是描述行为的，因此设计API时，应使用名词而非动词来描述语义，否则会引起混淆和语义不清。即：

```
# Bad APIs
/api/getArticle/1/
/api/updateArticle/1/
/api/deleteArticle/1/


```

上面四个url都是指向同一个资源的，虽然一个资源允许多个url指向它，但不同的url应该表达不同的语义，上面的API可以优化为：

```
# Good APIs
/api/Article/1/


```

article 资源的获取、更新和删除分别通过  `GET`,  `PUT`  和  `DELETE`方法请求API即可。试想，如果url以动词来描述，用`PUT`方法请求  `/api/deleteArticle/1/`  会感觉多么不舒服。

## 6.3  `GET`  and  `HEAD`  should always be safe

[RFC2616](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html)已经明确指出，`GET`和`HEAD`方法必须始终是安全的。例如，有这样一个不规范的API:

```

# The following api is used to delete articles
# [GET]
/api/deleteArticle?id=1

```

试想，如果搜索引擎访问了上面url会如何？

## 6.4 Nested resources routing

如果要获取一个资源子集，采用  `nested routing`  是一个优雅的方式，如，列出所有文章中属于Gevin编写的文章：

```
# List Gevin's articles
/api/authors/gevin/articles/

```

获取资源子集的另一种方式是基于`filter`（见下面章节），这两种方式都符合规范，但语义不同：如果语义上将资源子集看作一个**独立的资源集合**，则使用  `nested routing`  感觉更恰当，如果资源子集的获取是出于**过滤**的目的，则使用`filter`更恰当。

至于编写RESTful API时到底应采用哪种方式，则仁者见仁，智者见智，语义上说的通即可。

## 6.5 Filter

对于资源集合，可以通过url参数对资源进行过滤，如：

```
# List Gevin's articles
/api/articles?author=gevin

```

分页就是一种最典型的资源过滤。

## 6.6 Pagination

对于资源集合，分页获取是一种比较合理的方式。如果基于开发框架（如Django REST Framework），直接使用开发框架中的分页机制即可，如果是自己实现分页机制，Gevin的策略是：

返回资源集合是，包含与分页有关的数据如下：

```
{
  "page": 1,            # 当前是第几页
  "pages": 3,           # 总共多少页
  "per_page": 10,       # 每页多少数据
  "has_next": true,     # 是否有下一页数据
  "has_prev": false,    # 是否有前一页数据
  "total": 27           # 总共多少数据
}

```

当想API请求资源集合时，可选的分页参数为：

参数

含义

page

当前是第几页，默认为1

per_page

每页多少条记录，默认为系统默认值

另外，系统内还设置一个`per_page_max`字段，用于标记系统允许的每页最大记录数，当`per_page`值大于  `per_page_max`  值时，每页记录条数为  `per_page_max`。

## 6.7 Url design tricks

（1）Url是区分大小写的，这点经常被忽略，即：

-   `/Posts`
-   `/posts`

上面这两个url是不同的两个url，可以指向不同的资源

（2）Back forward Slash (`/`)

目前比较流行的API设计方案，通常建议url以`/`作为结尾，如果API  `GET`请求中，url不以`/`结尾，则重定向到以`/`结尾的API上去（这点现在的web框架基本都支持），因为有没有  `/`，也是两个url，即：

-   `/posts/`
-   `/posts`

这也是两个不同的url，可以对应不同的行为和资源

（3）连接符  `-`  和 下划线  `_`

RESTful API 应具备良好的可读性，当url中某一个片段（segment）由多个单词组成时，建议使用  `-`  来隔断单词，而不是使用  `_`，即：

```
# Good
/api/featured-post/

# Bad
/api/featured_post/


```

这主要是因为，浏览器中超链接显示的默认效果是，[文字并附带下划线](http://blog.igevin.info/)，如果API以`_`隔断单词，二者会重叠，影响可读性。

# 总结

编写本文的初衷，是为了整理一套从零开始编写规范、安全的RESTful API的基本思路。本文介绍了开发RESTful API时，要考虑的基本内容，对于类似Flask这种天生支持RESTful风格的web框架，不依赖其他RESTful第三方库开发RESTful 服务时，可以从本文内容入手；不少强大的RESTful 库，虽然其相关接口基本涵盖了本文的全部或大部分内容，但本文的总结，相信对这些库的理解和使用也是有帮助的。

最后，关于如何开发RESTful API，欢迎大家与我交流~


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE5MTk4NjUyNDgsODM5NDA4NTQyLC05ND
Y5ODE1MDVdfQ==
-->