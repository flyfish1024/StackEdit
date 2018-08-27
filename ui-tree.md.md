---
---
title: Angular ui-tree 使用心得
date: 2018-05-06 20:25:06
abstract: 总结分享下我关于ui-tree的使用与体会(含案例代码)
tags:
- angular
- js
header_image: /intro/ui_tree.jpg
---
---
**总结分享下我关于这个插件的使用与体会，三个方面**

* 插件作用
* 使用流程
	* 数据库表数据
	* 获取所有数据
		* 数据处理
	* 页面显示
	* 搭载checkbox
		* 页面显示
		* 子父级联动
		* 回显
* 坑

## 插件作用
具备所有tree插件的特点，显示分支形式的数据，可折叠，可修改，添加同级别，子级别数据。实际应用：例如 *组织机构树*，本篇我们将用此案例进行演示。

**angular-ui-tree还有如下特点：**

1.使用本地AngularJS范围数据绑定

2.整个树都可以进行排序和移动项目

*官方demo截图*
![ui_tree_demo](../../../../assets/img/ui_tree_demo.png)

*数据源*
``` json
[
  {
    "id": 1,
    "title": "node1",
    "nodes": [
      {
        "id": 11,
        "title": "node1.1",
        "nodes": [
          {
            "id": 111,
            "title": "node1.1.1",
            "nodes": []
          }
        ]
      },
      {
        "id": 12,
        "title": "node1.2",
        "nodes": []
      }
    ]
  },
  {
    "id": 2,
    "title": "node2",
    "nodrop": true,
    "nodes": [
      {
        "id": 21,
        "title": "node2.1",
        "nodes": []
      },
      {
        "id": 22,
        "title": "node2.2",
        "nodes": []
      }
    ]
  },
  {
    "id": 3,
    "title": "node3",
    "nodes": [
      {
        "id": 31,
        "title": "node3.1",
        "nodes": []
      }
    ]
  }
]
```

[ui-tree官方demo](http://angular-ui-tree.github.io/angular-ui-tree/#/basic-example)

## 使用流程
本小节按照以下流程进行：
1.  数据库表结构
2.  获取所有数据
	*  数据处理
4.  页面显示
5.  搭载checkbox
    1.  页面显示
    2.  子父级联动
    3.  回显

```txt
为了便于大家的学习与使用，我以一个完整的案例进行演示。大家可酌情选看适合自己的小小节。
```

### 数据库表结构
table_name :party_group

|机构id|上级id|机构名称|级别|能否有下一级|
|:----    |:---|:---|:---|:---|
|nId|nPid|vcName|nLevel|bLeaf|
|1|	0|万源实业集团党委| 1|	1|
|2|	0|航天一院院党委|	1|	1|
|3|	1|建筑公司党支部|	2|	1|
|4|	3|建筑第一党小组|	3|	0|
|5|	3|建筑第二党小组|	3|	0|
|6|	3|建筑第三党小组|	3|	0|
|7|	3|建筑第四党小组|	3|	0|

### 获取所有数据
```js
 select * from party_group
```
后台简单的查询所有就可以，数据处理交给自由的js。
```js
//获取机构  
$scope.getPartyGroup = function () {  
	$http.get(BASEURL+'/partyGroup/getPartyGroup').success(function(resData){  
	    $scope.groups = resData.list;//将数据存入$scope.groups；list是我后台存储的键
		//获取所有一级列表
		var topArray = [];  
		//一级列表就是上级id为0的数据，存入数组 topArray 
		for(var i=0 ;i < $scope.groups.length; i++){  
			if($scope.groups[i].nPid == 0){  
	            topArray.push($scope.groups[i]);  
			}  
		}  
        //递归装配子集合  
		$scope.fillTree(topArray);  
		console.log(topArray);//打印验证一下
		$scope.data = topArray;//储存处理好的数据，交由模板处理生成页面
		console.log($scope.data);//打印验证一下
    });  
};
//递归获取当前集合的下一级别数据
$scope.fillTree = function (array) {  
	if(array!=null){  
		for(var item in array){  
			var childArray = [];//当前的子集合  
			for(var i =0;i < $scope.groups.length; i++){  
                if($scope.groups[i].nPid == array[item].nId){  
                    childArray.push($scope.groups[i]);  
				}  
            }  
            array[item].subGroup = childArray; //tree的关键子集合属性
			$scope.fillTree(childArray);  
		}  
    }  
};
```
这里有三个注意的点
* js 既不是按值传递(call by value)，也不是按引用传递(call by reference)，而是按**共享传递 （call by sharing）**，这决定了`topArray `的数据为第一级列表与所有后代数据的集。
* 交由`template`处理的数据`data`，类型**必须**为`array`。
* 每层数据对象都要包含自己的子对象数组属性，我这里起名为`subGroup`，以供页面模板嵌套生成。

### 页面显示

```html
<!-- 模板部分，包含折叠，添加，修改，删除功能 -->
<script type="text/ng-template" id="nodes_renderer">  
	<div ui-tree-handle class="tree-node tree-node-content" >  
		<a class="btn btn-success btn-xs " ng-if="group.subGroup&& group.subGroup.length > 0" data-nodrag ng-click="toggle(this)">  
			<span class="glyphicon" ng-class="{'glyphicon-chevron-right': collapsed,'glyphicon-chevron-down': !collapsed}"></span>
		</a>  
		<div ng-click="getPersonalForGroupId(group.nId)" style="display:inline-block; width:66%;">{{group.vcName}}</div>  
		<a ng-show="group.bLeaf !=1 || group.subGroup.length == 0" class="pull-right btn btn-danger btn-xs" data-nodrag ng-click="removeThis(this)">
			<span class="glyphicon glyphicon-remove"></span>
		</a>  
		<a class="pull-right btn btn-danger btn-xs" data-nodrag ui-sref="addPartyGroup({group:group})">
			<span class="glyphicon glyphicon-edit"></span></a>  
		<a ng-show="group.bLeaf == 1" class="pull-right btn btn-primary btn-xs" data-nodrag ng-click="newSubGroup(this)" style="margin-right: 8px;">
			<span class="glyphicon glyphicon-plus"></span>
		</a>  
	</div>
	<ol ui-tree-nodes="" ng-model="group.subGroup" ng-class="{hidden: collapsed}">  
		<li ng-repeat="group in group.subGroup" ui-tree-node ng-include="'nodes_renderer'" ></li>
	</ol>
</script>
<!-- 如果没有后代，则不能折叠；如果没有下级字段为1，则可添加下级；通过data-nodrag设置不可拖动 -->
<!-- 这里添加，修改，删除的代码就不再贴了，修改是直接跳转路由 -->

<!-- 设置起点，引用模板 -->
<div class="row">  
	<div class="col-sm-12">  
		<div ui-tree id="tree-root" data-drag-enabled="false">  
			<ol ui-tree-nodes ng-model="data">  
				<li ng-repeat="group in data" ui-tree-node ng-include="'nodes_renderer'" collapsed="false"></li>
			</ol>  
		</div>
	</div>
</div>
<!-- 传入数据data，设置折叠属性collapsed -->
```
好了，现在就可以查看效果了
![group_tree](../../../../assets/img/机构树.png)

### 搭载checkbox
在模板上添加`checkbox`
```html
<!-- 模板部分，包含折叠，勾选功能 -->
<script type="text/ng-template" id="nodes_renderer">  
	<div ui-tree-handle class="tree-node tree-node-content" style="margin-left: 0px">  
		<a class="btn btn-success btn-xs " ng-if="group.subGroup && group.subGroup.length > 0" data-nodrag ng-click="toggle(this)">  
			<span class="glyphicon" ng-class="{'glyphicon-chevron-right': collapsed,'glyphicon-chevron-down': !collapsed}"></span>  
		</a> 
		<input name="groupCheck" type="checkbox" style="vertical-align: bottom;width:15px;height:15px;margin-right:3px;"  
  ng-click="checkClick(this)" ng-checked="group.isCheck" value="{{group.nId}}" >{{group.vcName}}  
	</div>  
	<ol ui-tree-nodes="" ng-model="group.subGroup" ng-class="{hidden: collapsed}">  
		<li ng-repeat="group in group.subGroup" ui-tree-node ng-include="'nodes_renderer'">  
		</li>
	</ol>
</script>
<!-- 通过新属性 isCheck 判断是否勾选-->

<!-- 瘦身，紧凑的css，可不写 -->
<style>  
  .tree-node-content {  
        margin: 1px;  
  }  
    .angular-ui-tree-handle {  
        padding: 2px 0;  
  }  
</style>
```
效果图

```js

//子父级联动
//一、多级关系保持一致
//处理一级选择一致  
$scope.checkClick = function (obj) {  
    var model = obj.$nodeScope.$modelValue;//通过this形式，找到当前对象数据
	if (model.bLeaf){//如果有下一级
		var obj_=document.getElementsByName('groupCheck');//为每个checkbox设置的name 
		for(var j=0; j<obj_.length; j++){  

			if(model.nId == obj_[j].value){  
	            //有子节点  
				$scope.checkSubLeaf(model.subGroup,obj_[j].checked);  
				return;  
			}  
	    }  
	}  
};  
//迭代勾选全部后代 state为上一级的状态
$scope.checkSubLeaf = function (subGroup,state) {  
    if(subGroup != undefined){
        var obj_=document.getElementsByName('groupCheck');  
		for(var i = 0; i < obj_.length; i++){  
	        for(var j = 0; j < subGroup.length; j++){  
		        if(obj_[i].value == subGroup[j].nId) {  
					obj_[i].checked = state;  
					$scope.checkSubLeaf(subGroup[j].subGroup, state);  
				}
            }
        }
    }
};
//二、两级关系保持一致
//前提，点击事件绑定不再用 this 改为：ng-click="checkClick($event)"
$scope.checkClick = function () {  
	//后代同 祖宗选择一致  
	var isChecked = event.target.checked;  
	var checkArray = $(event.target).closest("li").find(".tree-check");//通过标签关系找到所有后代checkbox  
	for(var i = 0;i < checkArray.length; i++){  
		checkArray[i].checked = isChecked;  
	}  
    //子选父必选，可不写
	if(isChecked){  
        var parentCheck = $(event.target).closest("ol").prev().find(".tree-check");//父checkbox  
		if(parentCheck.length > 0){  
	        parentCheck[0].checked = isChecked;  
		}  
	}  
}
```

回显的思想
1. 获取已选择的机构的集
2. 根据1的集，标记所有机构对象的`isChecked`属性
3. 模板生成

## 坑
提供给模板的数据必须是数组Array类型，单个元素的话，页面生成会看不到数据，只有空白的条条，试了很久才发现，有点难受。

就写到这里，欢迎指教~

>最后更新于 2018-05-10


<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE2ODQyMjYzOThdfQ==
-->