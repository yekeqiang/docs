# 利用 Docke Llinks 实现 Virgo 应用到 MongoDB 的安全连接

#### 作者：[Florian Waibel](http://eclipsesource.com/blogs/author/fwaibel/)

#### 译者：[Chen Wang](https://github.com/csgtree)

---
几个月前，Docker 0.6.5 版本 [发布](http://blog.docker.io/2013/10/docker-0-6-5-links-container-naming-advanced-port-redirects-host-integration/)。自该版本开始，你可以命名自己的 container ，也更方便找到想要的container 。很快，这个功能我就上手了。

  
container 命名，配合新引入的链接（ links ）功能，大大改进了 Docker 的安全机制。
  

链接（ links ）通过使用标记 [-link name:alias](http://docs.docker.io/en/latest/use/working_with_links_names/) ，在 container 之间实现安全的通信。在必要之时，设置新的守护进程（ daemon ）标记 －icc=false ，则会禁止 container 间的交互。


我们不妨实践下，显式建立一个链接（ links ），从 Virgo container 中执行的应用，到另一个 container 里运行 MongoDB 。

```
	$ docker run -name mongodb -t eclipsesource/mongodb
	$ docker run -link mongodb:db -name virgo -t eclipsesource/virgo-spring-325
```

Docker提供了访问被链接 container （译者注：此处指运行着 MongoDB 的 container）环境变量的方式：

```
	DB_PORT_27017_TCP_ADDR  
	DB_PORT_27017_TCP_PORT
```

 
这些环境变量的名字来源于两部分：MongoDB container 所开放的端口；启动 Virgo container 的命令中，-link 选项的第二部分（译者注，即"-link_mongodb:db"中的db）。


更多关于链接的内容，可以参考 Docker 文档的 "Link Containers" [章节](http://docs.docker.io/en/latest/use/working_with_links_names/)。


 
在 Virgo 应用里，你可以在创建 MongoDB factory 时，访问上述的环境变量：

```
<context:property-placeholder location="classpath:mongodb.properties" />
<mongo:db-factory dbname="tabris-trial"
	host="${DB_PORT_27017_TCP_ADDR}:${DB_PORT_27017_TCP_PORT}" />
```   
```
<bean id="mongoTemplate" class="org.springframework.data.mongodb.core.MongoTemplate">
	<constructor-arg ref="mongoDbFactory" />
</bean>
```

如若环境变量不可用，须像上面这段一样，采用 mongodb.properties 声明的默认配置。

先写到这，自己发掘去吧。。。

---

##### 这篇文章由 [Florian Waibel](http://eclipsesource.com/blogs/author/fwaibel/) 发表，点击 [这里](http://eclipsesource.com/blogs/2014/02/27/how-to-use-docker-links-to-safely-connect-from-a-virgo-application-to-a-mongodb/) 可查阅原文。

##### The article was contributed by [Florian Waibel](http://eclipsesource.com/blogs/author/fwaibel/) , click [here](http://eclipsesource.com/blogs/2014/02/27/how-to-use-docker-links-to-safely-connect-from-a-virgo-application-to-a-mongodb/) to read the original publication.


