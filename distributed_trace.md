# 分布式调用跟踪

## 1. Google 
[Google 论文翻译](http://bigbully.github.io/Dapper-translation/)   [英语原文](http://static.googleusercontent.com/media/research.google.com/en//pubs/archive/36356.pdf)

具体设计目标：

1. 低消耗：跟踪系统对在线服务的影响应该做到足够小。在一些高度优化过的服务，即使一点点损耗也会很容易察觉到，而且有可能迫使在线服务的部署团队不得不将跟踪系统关停。

2. 应用级的透明：对于应用的程序员来说，是不需要知道有跟踪系统这回事的。如果一个跟踪系统想生效，就必须需要依赖应用的开发者主动配合，那么这个跟踪系统也太脆弱了，往往由于跟踪系统在应用中植入代码的bug或疏忽导致应用出问题，这样才是无法满足对跟踪系统“无所不在的部署”这个需求。面对当下想Google这样的快节奏的开发环境来说，尤其重要。

3. 延展性：Google至少在未来几年的服务和集群的规模，监控系统都应该能完全把控住。

做到真正的应用级别的透明，这应该是当下面临的最挑战性的设计目标，我们把核心跟踪代码做的很轻巧，然后把它植入到那些无所不在的公共组件种，比如线程调用、控制流以及RPC库。

简单实用的分布式跟踪的实现，就是为服务器上每一次发送和接收动作来收集跟踪标识符(message identifiers)和时间戳(timestamped events)。

两种解决方案：

1. 黑盒方案（black-box）：假定需要跟踪的除了上述信息之外没有额外的信息，这样使用统计回归技术来推断两者之间的关系。轻便，需要更多的数据，以获得足够的精度，因为他们依赖于统计推论

2. 标注方案(annotation-based)：依赖于应用程序或中间件明确地标记一个全局ID，从而连接每一条记录和发起者的请求。需要代码植入。代码植入限制在一个很小的通用组件库中，从而实现了监测系统的应用对开发人员是有效地透明。

![Dapper跟踪树种短暂的关联关系](http://bigbully.github.io/Dapper-translation/images/img2.png)

![一个单独的span的细节图](http://bigbully.github.io/Dapper-translation/images/img3.png)

任何一个span可以包含来自不同的主机信息，也要记录下来。事实上，每一个RPC span可以包含客户端和服务器两个过程的注释，使得链接两个主机的span会成为模型中所说的span。由于客户端和服务器上的时间戳来自不同的主机，必须考虑到时间偏差。分析工具利用了这个事实：RPC客户端发送一个请求之后，服务器端才能接收到，对于响应也是一样的（服务器先响应，然后客户端才能接收到这个响应）。这样一来，服务器端的RPC就有一个时间戳的一个上限和下限。

![Dapper收集管道总览](http://bigbully.github.io/Dapper-translation/images/img5.png)

Dapper的跟踪记录和收集管道的过程分为三个阶段。首先，span数据写入（1）本地日志文件中。然后Dapper的守护进程和收集组件把这些数据从生产环境的主机中拉出来（2），最终写到（3）Dapper的Bigtable仓库中。一次跟踪被设计成Bigtable中的一行，每一列相当于一个span。Bigtable的支持稀疏表格布局正适合这种情况，因为每一次跟踪可以有任意多个span。跟踪数据收集（即从应用中的二进制数据传输到中央仓库所花费的时间）的延迟中位数少于15秒。第98百分位的延迟(The 98th percentile latency)往往随着时间的推移呈现双峰型;大约75%的时间，第98百分位的延迟时间小于2分钟，但是另外大约25%的时间，它可以增涨到几个小时。

![](http://bigbully.github.io/Dapper-translation/images/img6.png)

## 2. Twitter Zipkin
### 2.1 综述


[zipkin官方](http://zipkin.io/)  [Github地址](https://github.com/openzipkin/)  [DockerHub](https://hub.docker.com/u/openzipkin/)

![总体图](http://zipkin.io/public/img/web-screenshot.png)

[目前支持的lib库语言](http://zipkin.io/pages/existing_instrumentations.html)

官方：go/java/js/ruby/scala 等
社区：c#/go/java/python 等

### 2.2 运行

使用Docker运行Zipkin：
	
	# git clone https://github.com/openzipkin/docker-zipkin
	# docker-compose up
	

## 2.3 测试

采用 Zipkin go lib进行测试：[zipkin-go-opentracing](https://github.com/openzipkin/zipkin-go-opentracing)

	# git clone https://github.com/openzipkin/zipkin-go-opentracing.git
	# cd zipkin-go-opentracing/examples/cli_with_2_services/cli && go build
	# cd zipkin-go-opentracing/examples/cli_with_2_services/svc1/cmd && go build
	# cd zipkin-go-opentracing/examples/cli_with_2_services/svc1/cmd && go build
	
分别启动svc1和svc2，然后启动运行cli
	
登录到Zipkin后显示效果如下：
	
![Summary](http://www.do1618.com/wp-content/uploads/2016/12/zipkin_cli_summary.png)

![one_recorde](http://www.do1618.com/wp-content/uploads/2016/12/zipkin_one.png)

![detail](http://www.do1618.com/wp-content/uploads/2016/12/zipkin_one_detail.png)

![dep](http://www.do1618.com/wp-content/uploads/2016/12/zipkin_dep.png)
	

## 3. 大众点评 CAT

### 3.1 介绍

[Github地址](https://github.com/dianping/cat)

CAT基于Java开发的实时应用监控平台，包括实时应用监控，业务监控。eBay的CAL是其原型。

CAT支持的监控消息类型包括：

1. <b>Transaction</b> 适合记录跨越系统边界的程序访问行为,比如远程调用，数据库调用，也适合执行时间较长的业务逻辑监控，Transaction用来记录一段代码的执行时间和次数。

2. <b>Event</b> 用来记录一件事发生的次数，比如记录系统异常，它和transaction相比缺少了时间的统计，开销比transaction要小。

3. <b>Heartbeat</b> 表示程序内定期产生的统计信息, 如CPU%, MEM%, 连接池状态, 系统负载等。

4. <b>Metric</b> 用于记录业务指标、指标可能包含对一个指标记录次数、记录平均值、记录总和，业务指标最低统计粒度为1分钟。

5. <b>Trace</b> 用于记录基本的trace信息，类似于log4j的info信息，这些信息仅用于查看一些相关信息

CAT 在提供实时应用监控和业务监控的基础上，提供了 <b>消息树</b> 的功能能够提供完整的调用关系和序列。 

CAT监控系统将每次URL、Service的请求内部执行情况都封装为一个完整的消息树、消息树可能包括Transaction、Event、Heartbeat、Metric和Trace信息。	

![完整消息树](https://camo.githubusercontent.com/9056837192598119dbd8f4559a74aef9a58bb186/68747470733a2f2f7261772e6769746875622e636f6d2f6469616e70696e672f6361742f6d61737465722f6361742d686f6d652f7372632f6d61696e2f7765626170702f696d616765732f6c6f6776696577416c6c30312e706e67)

目前客户端语言支持： Java和.net

[透过CAT，来看分布式实时监控系统的设计与实现](http://www.tuicool.com/articles/z2u2Mn)
[大众点评CAT架构分析](http://blog.csdn.net/szwandcj/article/details/51025669)


## 4. 淘宝 鹰眼

基于HFS开发，目前未开源。
	
![鹰眼整体结构](http://images2015.cnblogs.com/blog/524341/201607/524341-20160727210517138-692387667.png)

[相关连接1](https://github.com/Percy0601/log-extension)
[鹰眼下的淘宝](http://club.alibabatech.org/resource_detail.htm?topicId=102)
	
## 5. 京东 Hydra

[Github链接](https://github.com/odenny/hydra)

Hydra是java开发的分布式跟踪系统。可以接入各种基础组件，完成对业务系统的跟踪。已接入的基础组件是阿里开源的分布式服务框架Dubbo。

Hydra可以针对并发量和数据量的大小选择（需要手动配置），是否使用消息中间件，使用Hbase或是Mysql存储跟踪数据。

Hydra自身提供跟踪数据展现功能，基于angularJS和D3.js

## 6. 新浪 Watchman

[微博平台的链路追踪及服务质量保障系统——Watchman系统](http://www.infoq.com/cn/articles/weibo-watchman)

[亿级用户下的新浪微博平台架构(pdf)](http://qndbp.qiniudn.com/weibo.pdf)
[亿级用户下的新浪微博平台架构(html)](https://linux.cn/article-4715-1.html)

## 7. 唯品会 Microscope

[唯品会Microscope——大规模分布式系统的跟踪、监控、告警平台](http://blog.csdn.net/alex19881006/article/details/24381109)

未看到开源信息。

## 8. 窝窝网 Tracing

未有详细信息

## 9. eBay Centralized Activity Logging (CAL)

[eBay架构演进](http://www.addsimplicity.com/downloads/eBaySDForum2006-11-29.pdf)


## 常用 RPC 框架


[gRPC](http://www.grpc.io/)
[Thrift](https://thrift.apache.org/)
[Hessian](http://hessian.caucho.com/index.xtp)
[Dubbo](http://dubbo.io/) 阿里Java高性能优秀的服务框架
[Hprose](http://www.hprose.com/) 高性能远程对象服务引擎
[ICE](https://zeroc.com/products/ice)

参考：
[RPC框架性能基本比较测试](http://www.useopen.net/blog/2015/rpc-performance.html)


## 参考链接

1. [老司机的微服务架构实现，照亮你的人生](http://mp.weixin.qq.com/s?__biz=MzI3MzEzMDI1OQ==&mid=2651815259&idx=1&sn=52c3a730dbbcda8b878de718c79a17e0&chksm=f0dc2b27c7aba23177fa9370028fcc5ddf75edc9ea5cb7bb4f359d74b5aa6d9b5b9af2b66ef6&scene=21#wechat_redirect)
2. [分布式跟踪系统调研](http://www.zenlife.tk/distributed-tracing.md)
3. [分布式行为追踪系统-Zipkin](http://nezhazheng.com/2014/01/14/try-zipkin.html)
4. [阿里RPC开源框架](http://dubbo.io/)
