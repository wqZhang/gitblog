---
title: 初次使用GraphQL
date: 2017-04-28 14:28:07
tags: graphql
reward: true
---
第一次使用graphQL进行数据的查询和读取
总的来说，graphQL本身挺好理解的,它是定义了新的一种数据交流方式
由于我用的是`java`语言，所以这里用了[graphql-java](https://github.com/graphql-java/graphql-java)的实现
但是中间有个关键点被我误解了，耽误了挺久的 --！
<!--more-->

构建了一个简单的书本信息和分类信息表进行测试，表结构如下：
```java
	//book
	@Id
	private String id;
	private String name;
	private String book_category_id;
	
	//book_category
	@Id
	private String id;
	private String name;
```
主要想用graphQL来显示单个查询和关联查询，并且对返回的字段可控制
项目使用了Spring-boot以及JPA来构建

在graphQL中，首先根据数据库结构定义返回结构体，也就是`GraphQLOutputType`对象
在我们的项目中，有两个返回结构体需要定义
```java
	private GraphQLOutputType bookType;
	private GraphQLOutputType categoryType;
```
然后定义一个初始化方法对上面两个`GraphQLOutputType`初始化
```java
	/**
	 * book
	 */
	categoryType = newObject()
	        .name("BookCategory")
	        .field(newFieldDefinition().name("id").type(GraphQLString).build())
	        .field(newFieldDefinition().name("name").type(GraphQLString).build())
	        .build();
		
	/**
	 * book
	 */
	bookType = newObject()
	         .name("Book")
	         .field(newFieldDefinition().name("id").type(GraphQLString).build())
	         .field(newFieldDefinition().name("name").type(GraphQLString).build())
	         .field(newFieldDefinition().name("book_category_id").type(GraphQLString).build())
	         .field(newFieldDefinition().name("bookCategory").type(categoryType).build())
	         .build();
	}
```
然后根据上面定义的结构，定义所需要返回的结构
这里有两点需要注意，定义的`name`属性对应了返回的具体结构体
另一个点就是坑我挺久的`dataFetcher`,它只提供了结构，不会帮你去获取数据
具体数据信息还是需要自己去获取，不管你是使用原生的jdbc，还是jpa，或者其他持久层框架
并且返回的数据结构需要和定义的一致

我这里定了两个返回结构，book对应单本书籍信息，books对应书本列表信息：
```java
	private GraphQLFieldDefinition createBookField() {
        return GraphQLFieldDefinition.newFieldDefinition()
                .name("book")
                .type(bookType)
                .argument(newArgument().name("id").type(GraphQLString).build())
                .dataFetcher(environment -> {
                	String id = environment.getArgument("id");
                    logger.debug(id);

                	Book book = bookService.getBookById(id);
                	return book;
                })
                .build();
    }
	
    private GraphQLFieldDefinition createBooksField() {
        return GraphQLFieldDefinition.newFieldDefinition()
                .name("books")
                .type(new GraphQLList(bookType))
                .dataFetcher(environment -> {
                    
                    List<Book> list = new ArrayList<Book>();
                    list = bookService.getAllBooks();
                	return list;
                })
                .build();
    }
```

`tyep`属性里对象名就是对应了我们上面定义的`GraphQLOutputType`对象
如果返回的是数组，就需要用`GraphQLList`对象进行包裹
在`book`返回结构体重还定义了`argument`参数，这样可以通过参数来获取对应的数据

最后我们再把定义好的返回方法与GraphQLSchema对象设置好，这个对象才是我们在controller
里的操作对象，再将设置好的对象返回：
```java
	public GraphQLSchema getGraphSchema() {
        initOutputType();
        schema = GraphQLSchema.newSchema().query(newObject()
                .name("GraphQuery")
                .field(createBookField())
                .field(createBooksField())
                .build()).build();
        return schema;
    }
```

最后我们在定义一个控制器用来访问数据，由于通过graphQL读取数据需要给出一个数据结构，
我提供了一个默认的结构：
```java
	@RequestMapping(value = "/books", method = RequestMethod.GET, produces = "application/hal+json;charset=UTF-8")
	public HttpEntity<?> getBatch(@RequestParam(required = false, 
		defaultValue = "{books {id,name,bookCategory{name}}}") String query) {
		
		logger.debug(query);
		GraphQLSchema schema = graphQLTest.getGraphSchema();
		
		Map<String, Object> result2 = (Map<String, Object>) new GraphQL(schema).execute(query).getData();
		
        return new ResponseEntity<>(result2, HttpStatus.OK);
    }
```
把整个项目运行起来就可以看到相关的结果了：

1.采用默认的query语句`{books {id,name,bookCategory{name}}}`
![pic1](http://img.wqzhang.top/graphqlresult1.png)

2.更换query语句结构为`{books {id,name}}`
![pic2](http://img.wqzhang.top/graphqlresult2.png)

3.更换query查询类型，通过id查询单个book信息`{book (id:"1") {id,name,bookCategory{name}}}`
![pic3](http://img.wqzhang.top/graphqlresult3.png)

至于grapgQL的其他特性，包括更新数据等等，我将会在后面更新，thanks！
