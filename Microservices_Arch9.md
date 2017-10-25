# “微服务”架构 9大特征

> 2011年5月在威尼斯附近的一个软件架构工作坊中，大家开始讨论“微服务”这个术语，因为这个词可以描述参会者们在架构领域进行探索时所见到的一种通用的架构风格。2012年5月，这群参会者决定将“微服务”作为描述这种架构风格的最贴切的名字。在2012年3月波兰的克拉科夫市举办的“33rd Degree”技术大会上，本文作者之一James在其[“Microservices - Java, the Unix Way”](http://2012.33degree.org/talk/show/67)演讲中以案例的形式谈到了这些微服务的观点，与此同时，Fred George也表达了[同样的观点](http://www.slideshare.net/fredgeorge/micro-service-architecure)。Netflix公司的Adrian Cockcroft将这种方法描述为“细粒度的SOA”，并且作为先行者和本文下面所提到的众人已经着手在Web领域进行了实践——Joe Walnes, Dan North, Evan Botcher 和 Graham Tackley。

![Monoliths_VS_MicroServices](https://martinfowler.com/articles/microservices/images/sketch.png) 

以一组 **services** 的方式来建造 **applications** ，起源于 Unix 的[设计哲学原则](https://my.oschina.net/u/184206/blog/370218)，详见[《Unix编程艺术》](http://download.csdn.net/download/peiyingli/2562620)。[Microservices Resource Guide](https://martinfowler.com/microservices/) 


![](https://martinfowler.com/bliki/images/microservice-verdict/productivity.png)

​                                                         图 单体开发与微服务开发的复杂度对比

### Services 的优点：

* 不同的技术栈独立开发，甚至不同的团队开发

* 独立部署

* 独立扩展

* 提供了清晰的边界




## 1. **组件化与服务化**

 一直以来我们开发系统的时候，都希望能够基于可插拔式的 **component** 来建立系统，追求功能的 “**内聚**” 和 “**解耦**” 的目的；就像我们在现实生活中采用拼积木方式来组装不同的玩具一样，每个积木都具备清楚的功能和边界， [OSGi(pen Service Gateway Initiative)](https://www.osgi.org/) 规范做了非常好的尝试，Java开发者可以基于规范构建更好模块化，动态性，热插拔性的系统；知名C++开源网络框架 [ACE](http://www.cs.wustl.edu/~schmidt/ACE-overview.html) 中的 [“ACE Service Configurator”](http://www.cs.wustl.edu/~schmidt/PDF/O-Service-Configurator.pdf) 模式也实现了类似的功能。

从软件复用的层次来讲，软件复用方式可以依次为：  **libraries** **->** **components** **->** **services**，其中以 **libraries** 与 **components** 一般情况情况下受制于相同技术栈，由于技术层次的耦合性，比较难以发现或者定义出比较清晰的边界；通过 **services** 方式的共享灵活性最大，调用成本也最高，但是容易形成比较清晰的边界，具备了 "**Services** 的优点" 章节描述的独有优势。

对于 **Services** 调用方式的功能演进，则可以通过内聚的服务边界和服务协议方面的演进机制来提供最大的兼容性；由于 **Services** 的通信成本更加昂贵，因此 API Interface 定义必须是 **粗粒度**，更多的接口面向业务并聚合多次调用，参见[使用DDD来构建你的REST API，而不是CRUD](https://mp.weixin.qq.com/s/251ql2WhDi-InUgVtIQ6_Q)，如果简单地将系统原有调用方式修改成远端的调用，则会让当前程序的代码逻辑更加凌乱，不易于维护（将数据从服务器发送给客户端时，应该一次性的发送所有内容，让客户端能完成当前的任务），参见[软件组件:粗粒度与细粒度](https://www.ibm.com/developerworks/cn/webservices/ws-soa-granularity/)。



## 2. **围绕 "业务功能” 组织团队**

 **康威定律**: *设计的结构与该组织的沟通结构相一致。*

> Any organization that designs a system (defined broadly) will produce a design whose structure is a copy of the organization's communication structure.
>
> -- Melvyn Conway, 1967

   ![组织架构变化](https://martinfowler.com/articles/microservices/images/conways-law.png)

如果按照传统的职能性技术栈来组织微服务开发，那么会经常出现跨多个团队的变更和延期，而且也会导致业务逻辑的组织散落在不同的组织代码库中，因此在面对微服务开发过程中，需要根据实际情况按照业务功能的逻辑来划分不同的而研发团队，从而形成 **敏捷性组织**[《架构即未来》3.3.3 章节]，例如按照浏览服务、用户服务、结账服务等不同的业务场景独立划分为研发团队，团队内部形成一个小的独立生态圈，包括独立的产品、研发和Devops人员。团队成员的规模，可以 [Amazon's Two Pizz Team](https://www.qianzhan.com/people/detail/268/141021-26c5bf4e.html) 的模式。

   ![服务边界的变化](https://martinfowler.com/articles/microservices/images/PreferFunctionalStaffOrganization.png)

由于按照业务功能来组织开发，组织的交流边界也会从原有的职能性转变层业务功能边界来进行，更加容易定义出团队的边界与责任。



## 3. **产品而非项目**

微服务的 ”敏捷组织团队“ 可以摆脱传统的开发与维护脱节的问题，”一个团队在一个产品的整个生命周期中都应该保持对其拥有“， 这种模式下因为业务相关的产品有一个团队负责，对于产品的整个生命周期管理（包括线上的日常运维数据的关注等），能够很好提升团队的创新性和团队开发的责任心，更加容易产出高质量的优秀产品。Amazon's 理念 ["you build, you run it"](https://queue.acm.org/detail.cfm?id=1142065)。



## 4. **”智能端点“与”哑巴管道“**

 当在不同的进程之间构建各种通信结构时，我们已经看到许多产品和方法，来强调将大量的智能特性纳入通信机制本身。这种状况的一个典型例子，就是 **“SOA架构下企业服务总线”** (Enterprise Service Bus, ESB)。ESB产品经常包括高度智能的设施，来进行消息的路由、编制(choreography)、转换，并应用业务规则。

微服务社区主张采用另一种做法：**智能端点** (Smart Endpoints)和**哑巴管道**(Dumb Pipes)。使用微服务所构建的各个应用的目标，都是尽可能地实现 **“高内聚和低耦合”** —他们拥有自己的领域逻辑，并且更多地是像经典Unix的 **“过滤器”** (filter)那样来工作—即接收一个请求，酌情对其应用业务逻辑，并产生一个响应。这些应用通过使用一些简单的REST风格的协议来进行编制，而不去使用诸如下面这些复杂的协议，即"WS-编制"(WS-Choreography)、BPEL或通过位于中心的工具来进行编排(orchestration)。

微服务最常用的两种协议是：带有资源API的HTTP “Request－Response” 协议和轻量级的消息发送协议，如 RabbitMQ或ZeroMQ。

**”智能端点“** 表达的意思是强化各端点服务的调用和编排消息的能力，**哑巴管道** 说表达的意思则是弱化通信机制的业务性与灵活性，定位在于消息路由与传递，功能更加简单。

   > Be of the web, not behind the web
   >
   > ​        [-- Ian Robinson](https://www.amazon.com/gp/product/0596805829?ie=UTF8&tag=martinfowlerc-20&linkCode=as2&camp=1789&creative=9325&creativeASIN=0596805829)

将一个单块系统改造为若干微服务的最大问题，在于对通信模式的改变。仅仅将内存中的方法调用转换为RPC调用这样天真的做法，会导致微服务之间产生繁琐的通信，使得系统表现变糟。取而代之的是，需要用更粗粒度的协议来替代细粒度的服务间通信。



## 5. **“去中心化”地治理技术**

总结：可以灵活选择不同的技术实现不同的问题，但是同步考虑公司内部与团队的技术积累与沉淀，选择更加合理的技术栈进行开发。

> Experience shows that this approach is constricting - not every problem is a nail and not every solution a hammer.

使用中心化的方式来对开发进行治理，其中一个后果，就是趋向于在单一技术平台上制定标准。经验表明，这种做法会带来局限性——不是每一个问题都是钉子，不是每一个方案都是锤子。我们更喜欢根据工作的不同来选用合理的工具。尽管那些单块应用系统能在一定程度上利用不同的编程语言，但是这并不常见。

如果能将单块应用的那些组件拆分成多个服务，那么在构建每个服务时，就可以有选择不同技术栈的机会。想要使用Node.js来搞出一个简单的报表页面？尽管去搞。想用C++来做一个特别出彩儿的近乎实时的组件？没有问题。想要换一种不同风格的数据库，来更好地适应一个组件的读取数据的行为？可以重建。



## 6. **“去中心化”地管理数据**

DDD将一个复杂的领域划分为多个限界上下文，并且将其相互之间的关系用图画出来。这一划分过程对于单块和微服务架构两者都是有用的，而且就像前面有关“业务功能”一节中所讨论的那样，在服务和各个限界上下文之间所存在的自然的联动关系，能有助于澄清和强化这种划分。

   ![去中心化的数据管理](https://martinfowler.com/articles/microservices/images/decentralised-data.png)



## 7. **“基础设施”自动化**

   ![CI/CD](https://martinfowler.com/articles/microservices/images/basic-pipeline.png)

   ![](https://martinfowler.com/articles/microservices/images/micro-deployment.png)



## 8. **“容错”设计**



## 9. **“演进式”设计**

可替换性组件，非常利于微服务架构的演进。演进式设计承认难以对边界进行正确定位，所以它将工作的重点放到了易于对边界进行重构之上。





测试

**UI(End-to-End Test)**   **Service**   **Unit**



**Service Tests**

Mocking or Stubbing



![测试金字塔](https://www.testwo.com/attachments/12697/1479217406521.png)

## 参考资料

* [微服务架构9大特征中文版](https://mp.weixin.qq.com/s?__biz=MjM5MjEwNTEzOQ==&mid=401500724&idx=1&sn=4e42fa2ffcd5732ae044fe6a387a1cc3#rd)
* [基于Spring Boot和Spring Cloud实现微服务架构学习](http://blog.csdn.net/enweitech/article/details/52582918)
* [39本架构类书籍](http://download.csdn.net/album/detail/4093/1/1)
* [用gomock进行mock测试](https://segmentfault.com/a/1190000009894570)
* [Go语言用mock server模拟调用(httptest)](http://www.cnblogs.com/aguncn/p/7102675.html)
* [mock.js-无需等待，让前端独立于后端进行开发](https://cnodejs.org/topic/53f718218f44dfa3511af923)
* [Mocks Aren't Stubs](https://www.martinfowler.com/articles/mocksArentStubs.html)   **

