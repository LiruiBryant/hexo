---
title: 从敲下www.baidu.com到浏览器显示页面，都发生了什么？
date: 2017-09-12 18:16:00
tags:
---
# 名词定义
## URL
    URL（Uniform Resource Locator），统一资源定位符，实际就是网站网址，又称域名。在茫茫网络世界中，浏览器就是靠URL来查找资源位置。URL包含协议部分，是浏览器和www万维网之间的沟通方式，它会告诉浏览器正确在网路上找到资源位置。常见的协议有http、https、ftp、file、telnet等。其中http是最常见的网络传输协议，而https则是进行加密的网络传输。

## IP
    为了便于记忆或辨识，人们使用域名来登录网站，每个域名背后有对应的IP地址。每个网站就是靠IP来定位的。IP是因特网中的每台连接到网络的计算机为实现相互通信而遵循的规则协议。IP分为局域网IP和全网IP。办公中常用的飞秋工具，就是使用办公室局域网IP进行通信的典型表现。每台计算机的本机IP都是127.0.0.1（即localhost）。浏览器并不能识别URL是什么，因此从输入URL开始，浏览器就要做域名解析，也就是IP寻址的工作。
## DNS
    DNS（Domain Name System，域名系统）——记录域名和IP地址相互映射的信息，人们可以免于记住IP数串。全国DNS信息可在网上查找到，各省都有对应分配不同的IP网段。大型企业会有自己的DNS服务器，专门存储域名和IP的映射关系，例如为大多数人熟知的谷歌DNS服务器，地址8.8.8.8。

# 查找
## 域名解析
  当用户在浏览器中输入url后,你使用的电脑会发出一个DNS请求到本地DNS服务器。本地DNS服务器一般都是你的网络接入服务器商提供，比如中国电信，中国移动,DNS请求到达本地DNS服务器之后会有以下几个步骤：
### 1. 浏览器缓存
  浏览器对本地缓存进行查找，查看是否有该域名历史记录，如果有首先直接对缓存地址访问。
  Chrome浏览器看本身的DNS缓存时间比较方便，在地址栏输入chrome://net-internals/#dns，就可以看到了
### 2. 本地hosts文件查找ip地址
  浏览器若无缓存或者缓存地址访问无效，则首先对hosts文件查找对应域名对应ip地址。
### 3. 查找路由器缓存
  如果系统缓存中也找不到，那么查询请求就会发向路由器，路由器一般会有自己的DNS缓存。
### 4. 查找ISP(Internet Service Provider) DNS 缓存
如果路由器缓存中也找不到，那么就查询ISP DNS 缓存服务器了。
我们都知道在我们的网络配置中都会有”DNS服务器地址”这一项，操作系统会把这个域名发送给这里设置的DNS，比如114.114.114.114,也就是本地区的域名服务器，通常是提供给你接入互联网的应用提供商。而114.114.114.114是国内移动、电信和联通通用的DNS。
### 5. 迭代查询
  如果前面都找不到DNS缓存的话，会有以下几个步骤：
  1. 本地 DNS服务器将该请求转发到互联网上的根域（根域没有名字，在DNS系统中就用一个空字符串来表示。例如www.baidu.com.现在的DNS系统都不会要求域名以.来结束，即www.baidu.com就可以解析了，但是现在很多DNS解析服务商还是会要求在填写DNS记录的时候以.来结尾域名。）
  2. 根域将所要查询域名中的顶级域（比如要查询www.baidu.com，该域名的顶级域就是com）的服务器IP地址返回到本地DNS。
  3. 本地DNS根据返回的IP地址，再向顶级域（就是com域）发送请求， com域服务器再将域名中的二级域（即www.baidu.com中的baidu.com）的IP地址返回给本地DNS。
  4. 本地DNS再向二级域发送请求进行查询。
  5. 之后不断重复这样的过程，直到本地DNS服务器得到最终的查询结果，并返回到主机。这时候主机才能通过域名访问该网站。
下图能很好的说明这个迭代查询:
![](./dns寻址过程.gif)
  当查找到对应的IP地址之后，通过IP地址查找到对应的服务器，浏览器将用户发起的http请求发送给服务器。例如：GET http://www.baidu.com/ HTTP/1.1
### 6. CDN
  CDN的基本原理是广泛采用各种缓存服务器，将这些缓存服务器分布到用户访问相对集中的地区或网络中，在用户访问网站时，利用全局负载技术将用户的访问指向距离最近的工作正常的缓存服务器上，由缓存服务器直接响应用户请求。
# 服务器处理用户请求
## 代理
### 正向代理
  正向代理是一种最终用户知道并主动使用的代理方式，比如三台支付服务服务器在微服务SpringCloud中的服务注册中心中注册了三个地址（1.1.1.1,2.2.2.2,3.3.3.3），用户服务服务器访问服务注册中心请求地址正向代理返回某一个地址（1.1.1.1），支付服务缓存到本地，在缓存有效期时间内，直接访问1.1.1.1服务器就可以了。
### 反向代理
  在计算机世界里，由于单个服务器的处理客户端（用户）请求能力有一个极限，当用户的接入请求蜂拥而入时，会造成服务器忙不过来的局面，可以使用多个服务器来共同分担成千上万的用户请求，这些服务器提供相同的服务，对于用户来说，根本感觉不到任何差别。比如你只需要敲下www.baidu.com这一个域名，其实背后有很多服务器支持着，每个人的请求会被分发到不同的服务器中。

  反向代理需要有一个负载均衡设备来分发用户请求，将用户请求分发到空闲的服务器上，负载均衡设备本身也可以有多个。

  **好处**：
  用户服务做域名解析时，解析到的ip地址是负载均衡设备的ip地址，不会暴露服务器地址。当需要增加、减少服务器数量时，用户一点都察觉不到。
## 负载均衡（反向代理的实现）
### 硬件负载均衡
  F5
### 软件负载均衡
 * LVS(七层协议第四层)
     LVS是Linux Virtual Server。lvs工作在4层，所以它可以对几乎所有应用做负载均衡，包括http、数据库、聊天室等等。同时，若跟硬件负载均衡相比它的缺点也不容忽视，LVS要求技术水平很高，操作上也比较复杂，配置也很繁琐，没有赖以保障的服务支持，稳定性来说也相对较低（人为和网络环境因素更多一些）。
 * nginx(七层协议第七层)
     Nginx是工作在第七层，对于网络的依赖性就小的多。与LVS相比，Nginx的安装和配置也相对简单一些，另外测试方面也更简单，主要还是因为对网络依赖性小的缘故。Nginx有一点不好的就是应用要比LVS少。一般我们做软件负载均衡的时候，通常会先考虑LVS，但是遇到比较复杂的网络环境时，用LVS可能会遇到很多麻烦，不妨就考虑尝试一下Nginx。

### 容器处理

  * 常用Java EE容器
    * tomcat
    * jetty
    * jBoss
  tomcat服务器启动main方法，当command命令行是start，就执行start方法。反射调用Catalina类的start()方法。这里面会弄出一个StandardServer对象（内部会使用Digester库去解析server.xml，那个里面的东西），并调用它的start()方法，把那些组件都启动起来。
  其中在连接器的启动过程中，会弄出一个叫做endpoint的东西去和底层的网络IO打交道，会调用其bind()方法，绑定端口号。
  connector将请求交给他所在的service里对应的engine，来处理；
  ```xml
    <Engine defaultHost="localhost" name="Catalina">
  ```
  每当HttpConnector的ServerSocket得到客户端的连接时，会创建一个Socket。
  考虑到客户端同时发来的请求数可能有很多， 所以tomcat 中默认维护着一个连接池
  NioChannel channel = nioChannels.pop();
  channel.setIOChannel(socket);
  channel.reset();
  getPoller0().register(channel);
  context随后在自己的mapping table里寻找servlet-mapping拦截路径为*.jsp的servlet，这里找到得是JSPServlet处理
  找到JSPServlet后，调用JSPServlet的doGet或doPOST方法；
  context执行完毕后，将HttpServletResponse 返回给Host；
  host将HttpServletResponse 返回给engine；
  engine 将HttpServletResponse 返回给 connector；
  connector 将 HttpServletResponse 返回给用户的browser；

  tomcat容器在启动后，context对应的项目在初始化的时候，要加载相应的servlet，加载的顺序为：
  $CATALINA_HOME/conf/web.xml里定义的servlet（对应于上面第5步的JSPServlet）；
  context对应的项目WEB-INF下的web.xml中定义的servlet；
### 应用处理
  1. Controller接收到请求，转发给service；
  2. service进行逻辑处理，如果有redis、ehcache等缓存则优先查询缓存；
  3. 如果是dubbo或者springcloud分布式，则通过dubbo服务或者restful请求调用其他服务模块请求信息；
  4. 调用dao查询数据库；
  5. 返回数据给service再给controller，controller再组装数据以及view返回；
  6. view层jsp或者velocity等模板引擎解析数据

### Mysql架构
  1. 服务器先会检查查询缓存，如果命中了缓存，则立即返回存储在缓存中的结果。
  2. 服务器端进行SQL解析、预处理，再由优化器生成对应的执行计划；
  3. MySQL根据优化器生成的执行计划，调用存储引擎的API来执行查询；
  4. 将结果返回给客户端。
#### MySQL能够处理的优化类型：
1. 重新定义关联表的顺序
数据表的关联并不总是按照在查询中指定的顺序进行，决定关联的顺序是优化器很重要的一部分功能。
2. 将外连接转化成内连接
并不是所有的outer join语句都必须以外连接的方式执行。诸多因素，例如where条件、库表结构都可能会让外连接等价于一个内连接。MySQL能够识别这点并重写查询，让其可以调整关联顺序。
3. 使用等价变换规则
MySQL可以使用一些等价变换来简化并规范表达式。它可以合并和减少一些比较，还可以移除一些恒成立和一些恒不成立的判断。例如：（5=5 and a>5）将被改写为a>5。类似的，如果有（a<b and b=c）and a=5，则会被改写为 b>5 and b=c and a=5。
4. 优化count()、min()和max()
索引和列是否为空通常可以帮助MySQL优化这类表达式。例如，要找到一列的最小值，只需要查询对应B-tree索引最左端的记录，MySQL可以直接获取索引的第一行记录。在优化器生成执行计划的时候就可以利用这一点，在B-tree索引中，优化器会讲这个表达式最为一个常数对待。类似的，如果要查找一个最大值，也只需要读取B-tree索引的最后一个记录。如果MySQL使用了这种类型的优化，那么在explain中就可以看到“select tables optimized away”。从字面意思可以看出，它表示优化器已经从执行计划中移除了该表，并以一个常数取而代之。
类似的，没有任何where条件的count(\*)查询通常也可以使用存储引擎提供的一些优化，例如，MyISAM维护了一个变量来存放数据表的行数。
5. 预估并转化为常数表达式
6. 覆盖索引扫描
当索引中的列包含所有查询中需要使用的列的时候，MySQL就可以使用索引返回需要的数据，而无需查询对应的数据行。
7. 子查询优化
MySQL在某些情况下可以将子查询转换成一种效率更高的形式，从而减少多个查询多次对数据进行访问。
8. 提前终止查询
在发现已经满足查询需求的时候，MySQL总是能够立即终止查询。一个典型的例子就是当使用了limit子句的时候。除此之外，MySQL还有几种情况也会提前终止查询，例如发现了一个不成立的条件，这时MySQL可以立即返回一个空结果。
上面的例子可以看出，查询在优化阶段就已经终止。
9. 等值传播
10. 列表in()的比较
在很多数据库系统中，in()完全等同于多个or条件的字句，因为这两者是完全等价的。在MySQL中这点是不成立的，MySQL将in()列表中的数据先进行排序，然后通过二分查找的方式来确定列表中的值是否满足条件，这是一个o(log n)复杂度的操作，等价转换成or的查询的复杂度为o(n)，对于in()列表中有大量取值的时候，MySQL的处理速度会更快。

# 浏览器渲染
