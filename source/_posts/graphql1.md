---
title: GrapgQL介绍(一)
date: 2017-04-05 22:46:39
tags: graphQL
reward: true
---

最近在学习[GraphQL](http://www.graphql.org/) API查询语言，想把官网的介绍文档翻译一下，学习的同时分享知识。



> 这一系列的文章是关于学习GraphQL怎么工作以及怎么使用。寻找怎么构建一个GraphQL服务？
> 这里有一些继承于GraphQL的多种语言的库。

<!--more-->

GraphQL是一种针对你的API的查询语言，也是一种通过你对你的数据定义范式执行查询的服务器端运行时。
GraphQL不会绑定特定的数据库或者存储引擎而是使用你当前的代码和数据。


一个GraphQL服务通过定义范式和范式里的字段来创建，然后为范式里的每一个字段提供功能。
例如，下面的GraphQL服务告诉我们谁登陆了(我)以及登陆用户的名称等一些信息：

	type Query {
		me: User
	}

	type User {
		id: ID
		name: String
	}

匹配范式里每一个字段功能：

	function Query_me(request) {
	  return request.auth.user;
	}

	function User_name(user) {
	  return user.getName();
	}

当一个GraphQL服务运行时(通常是一个web服务的一条URL)，它会接收GraphQL查询请求来验证和执行。
接收的查询首先会被检查是不是只关联到定义好的范式和字段，然后运行提供的函数产生结果。

一个查询的例子：

	{
	  me {
		name
	  }
	}

生成的JSON结果：

	{
	  "me": {
		"name": "Luke Skywalker"
	  }
	}
	
学习更多GraphQL -- 查询语言，范式系统，GraphQL服务如何工作以及
使用QraphQL解决问题的最佳实践 -- 请阅读以下章节。

# 查询和修改(Queries and Mutations)

在这个章节，你将详细地学习如何在一个GrapQL服务器上查询。

### 字段(Fields)

最基本的，GraphQL可以请求对象上的指定字段。让我们看一个非常简单的查询以及结果：

	{
	  hero {
		name
	  }
	}
	结果：
	{
	  "data": {
		"hero": {
		  "name": "R2-D2"
		}
	  }
	}

你可以直接地看到查询语句和结果是一样的结构。这是GraphQL最基本的功能，因为服务器明确地知道客户
需要哪些字段，所以你可以得到你预期的结果。

在上面的例子中，字段name作为一个string类型返回，那个名字是星际大战主英雄的名字，`R2-D2`。


> 顺便提一句 - 上面的查询语句是交互式的。也就是你可以按你的意愿改变结构并且得到对应的结果。
> 尝试在`hero`对象的查询语句里增加一个`appearsIn`字段并看看新的结果。

在上个例子中，我们得到了个字符串当我们要求返回hero的name字段，其实返回的字段也可以是对象。
在例子中，你可以为返回的对象设置级联字段*sub-selection*。GraphQL查询可以联通相关的对象以及字段，
让客户端在一个请求中得到所有需要的相关数据，而不是像传统REST架构那样向服务器往返请求多次。

	{
	  hero {
		name
		# Queries can have comments!
		friends {
		  name
		}
	  }
	}
	结果：
	{
	  "data": {
		"hero": {
		  "name": "R2-D2",
		  "friends": [
			{
			  "name": "Luke Skywalker"
			},
			{
			  "name": "Han Solo"
			},
			{
			  "name": "Leia Organa"
			}
		  ]
		}
	  }
	}

注意上面的例子，字段`friends`返回了一个数组。GraphQL查询对于单个项和列表项看起来相同，
但是我们根据给定的模式知道预期的内容。


### 参数(Arguments)

如果我们唯一可以做的只是遍历对象以及其字段，那GraphQL已经是获取数据非常有用的语言。但是，当你
将参数传递给字段时，这就变的更有趣了。

	{
	  human(id: "1000") {
		name
		height
	  }
	}
	结果：
	{
	  "data": {
		"human": {
		  "name": "Luke Skywalker",
		  "height": 1.72
		}
	  }
	}

像REST系统，你只能传递一组参数-通过查询参数和请求URL。但是在GraphQL中，每个字段和嵌套对象都
可以获得自己的一组参数。从而使GraphQL可以替代多个API提取。你甚至可以将参数传递到标量字段中，
以便在服务器上实现数据转换，而不是分别在每个客户端上执行数据转换。

	{
	  human(id: "1000") {
		name
		height(unit: FOOT)
	  }
	}
	结果：
	{
	  "data": {
		"human": {
		  "name": "Luke Skywalker",
		  "height": 5.6430448
		}
	  }
	}

参数可以是很多不同的类型。在上面的例子中，我们使用了一个枚举类型，它表示一组有限选项
（在这种情况下是长度单位，或者是METER或FOOT）之一。GraphQL带有自己的默认类型，但是在GraphQL上
也可以自定义类型，自定义类型也可以在传输数据的时候序列化。

[更多GraplQL类型内容](http://graphql.org/learn/schema/)


### 别名(Aliases)

如果你观察仔细，你可能已经发现了，返回的结果对象字段的名字匹配查询的字段但不包括参数，
所以你不能直接的通过不同的参数查询同样的字段。这样你就需要别名-它可以让你把返回的结果重命名
成任何其他你想要的。

	{
	  empireHero: hero(episode: EMPIRE) {
		name
	  }
	  jediHero: hero(episode: JEDI) {
		name
	  }
	}
	结果：
	{
	  "data": {
		"empireHero": {
		  "name": "Luke Skywalker"
		},
		"jediHero": {
		  "name": "R2-D2"
		}
	  }
	}

在上面的例子中，两个字段本应该有冲突，但是我们给他们定义了不同的别名后，
就可以在一个请求中获取所有的结果。
