---
---
title: MVVM模式
date: 2018-12-20 19:02:44
abstract: 通过双向绑定，数据异步获取实现前后端分离
tags:
- mvvm
- front end
header_image: /intro/mvvm.jpg
---
---


# 简言之

Model–View–ViewModel(MVVM) 是一个软件架构设计模式，由微软 WPF 和 Silverlight 的架构师 Ken Cooper 和 Ted Peters 开发，是一种简化用户界面的事件驱动编程方式。由 John Gossman（同样也是 WPF 和 Silverlight 的架构师）于2005年在他的博客上发表。

MVVM 源自于经典的 Model–View–Controller（MVC）模式（期间还演化出了 Model-View-Presenter（MVP）模式，可忽略不计）。MVVM 的出现促进了 GUI 前端开发与后端业务逻辑的分离，极大地提高了前端开发效率。MVVM 的核心是 ViewModel 层，它就像是一个中转站（value converter），负责转换 Model 中的数据对象来让数据变得更容易管理和使用，该层向上与视图层进行双向数据绑定，向下与 Model 层通过接口请求进行数据交互，起呈上启下作用。如下图所示：

![](https://images2017.cnblogs.com/blog/882926/201711/882926-20171115175942921-775941263.png)


MVVM 已经相当成熟了，主要运用但不仅仅在网络应用程序开发中。KnockoutJS 是最早实现 MVVM 模式的前端框架之一，当下流行的 MVVM 框架有 Vue，Angular 等。

# 组成部分

简单画了一张图来说明 MVVM 的各个组成部分：

![](https://images2017.cnblogs.com/blog/882926/201711/882926-20171115175958671-1955710845.png)

MVVM分层示意图

分层设计一直是软件架构的主流设计思想之一，MVVM 也不例外。

##  View 层

View 是视图层，也就是用户界面。前端主要由 HTML 和 CSS 来构建，为了更方便地展现 ViewModel 或者 Model 层的数据，已经产生了各种各样的前后端模板语言，比如 FreeMarker、Marko、Pug、Jinja2等等，各大 MVVM 框架如 KnockoutJS，Vue，Angular 等也都有自己用来构建用户界面的内置模板语言。

## Model 层

Model 是指数据模型，泛指后端进行的各种业务逻辑处理和数据操控，主要围绕数据库系统展开。后端的处理通常会非常复杂：

![](https://images2017.cnblogs.com/blog/882926/201711/882926-20171115180046718-1139564648.png)

前后端对比

后端：我们这里的业务逻辑和数据处理会非常复杂！  
前端：关我屁事！

后端业务处理再复杂跟我们前端也没有半毛钱关系，只要后端保证对外接口足够简单就行了，我请求api，你把数据返出来，咱俩就这点关系，其他都扯淡。

## ViewModel 层

ViewModel 是由前端开发人员组织生成和维护的视图数据层。在这一层，前端开发者对从后端获取的 Model 数据进行转换处理，做二次封装，以生成符合 View 层使用预期的视图数据模型。需要注意的是 ViewModel 所封装出来的数据模型包括视图的状态和行为两部分，而 Model 层的数据模型是只包含状态的，比如页面的这一块展示什么，那一块展示什么这些都属于视图状态（展示），而页面加载进来时发生什么，点击这一块发生什么，这一块滚动时发生什么这些都属于视图行为（交互），视图状态和行为都封装在了 ViewModel 里。这样的封装使得 ViewModel 可以完整地去描述 View 层。由于实现了双向绑定，ViewModel 的内容会实时展现在 View 层，这是激动人心的，因为前端开发者再也不必低效又麻烦地通过操纵 DOM 去更新视图，MVVM 框架已经把最脏最累的一块做好了，我们开发者只需要处理和维护 ViewModel，更新数据视图就会自动得到相应更新，真正实现数据驱动开发。看到了吧，View 层展现的不是 Model 层的数据，而是 ViewModel 的数据，由 ViewModel 负责与 Model 层交互，这就完全解耦了 View 层和 Model 层，这个解耦是至关重要的，它是前后端分离方案实施的重要一环。

# 没有什么是一个栗子不能解决的

扯了这么多，并没有什么卵用。千言万语不如一个栗子来的干脆，下面用一个 Vue 实例来说明 MVVM 的具体表现。

Vue 的 View 模板：

```
<div id="app">
    <p>{{message}}</p>
    <button v-on:click="showMessage()">Click me</button>
</div>
 
```

Vue 的 ViewModel 层（下面是伪代码）：

```
var app = new Vue({
    el: '#app',
    data: {     // 用于描述视图状态（有基于 Model 层数据定义的，也有纯前端定义）
        message: 'Hello Vue!',  // 纯前端定义
        server: {}, // 存放基于 Model 层数据的二次封装数据
    },
    methods: {  // 用于描述视图行为（完全前端定义）
        showMessage(){
            let vm = this;
            alert(vm.message);
        }
    },
    created(){
        let vm = this;

        // Ajax 获取 Model 层的数据
        ajax({
            url: '/your/server/data/api',
            success(res){
                // TODO 对获取到的 Model 数据进行转换处理，做二次封装
                vm.server = res;
            }
        });
    }
})
 
```

服务端的 Model 层（省略业务逻辑处理，只描述对外接口）：

```
{
    "url": "/your/server/data/api",
    "res": {
        "success": true,
        "name": "IoveC",
        "domain": "www.cnblogs.com"
    }
}
 
```

这就是完整的 MVVM 编程模式。

代码执行之后双向绑定的效果如下:

![](https://images2017.cnblogs.com/blog/882926/201711/882926-20171115180107812-1002364123.gif)

转载自[DOM哥-前后端分手大师--MVVM](https://www.cnblogs.com/iovec/p/7840228.html)
<!--stackedit_data:
eyJoaXN0b3J5IjpbNzk1NzE0NjkzXX0=
-->