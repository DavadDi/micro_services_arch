# “微服务”架构 9大特征解读

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

如果能将单块应用的那些组件拆分成多个服务，那么在构建每个服务时，就可以有选择不同技术栈的机会。想要使用Node.js来搞出一个简单的报表页面？尽管去搞。想用C++来做一个特别出彩儿的近乎实时的组件？没有问题。想要换一种不同风格的数据库，来更好地适应一个组件的读取数据的行为？

不同的语言都有着各自的擅长范围和领域，例如实时性要求高的C++，机器学习或者数据处理则可以用Python，云计算相关领域Golang，一般的企业应用有一定性能要求的则可以选择Node.js和Java，中小型的性能略低的可以采用Ruby和PHP等，技术栈的选择需要根据团队的人员储备与当前招聘相关的需求结合起来，不盲目最新，也不能一直固定模式，通过为服务化为试用各种新的技术栈提供了一个良好的契机。



[编程语言排行榜](https://www.tiobe.com/tiobe-index/) 和 [九种编程语言大对比](https://zhuanlan.zhihu.com/p/20887949?refer=mengmengzhou)

![](http://img.mp.sohu.com/upload/20170808/82a4cee4c25b41cca895a65d24f1ff2c_th.png)



## 6. **“去中心化”地管理数据**

由于微服务采用 **“分而治之”** 的思想，因此每个微服务都可以自己选择技术栈和存储；我们以前开发的一款社交软件所选择的数据库就有MySQL、Mongodb、Neo4j、Redis等多种，和数据建模模型有关也和各自技术栈编程语言有关，在于前期的构建过程中还是比较顺利，但是等到了上线以后的数据维护上，就面临了不少挑战，由于涉及不同的数据库，数据维护和备份机制等不同，也导致研发人员的注意力分散，方案的不通用性也会导致后期的维护成本增加，因此该点的考虑还是需要根据当前研发团队的技术擅长度和深度两者结合，避免过于扩大，增加后期的技术债和维护的投入成本。

DDD(Domain-Driven Design) 将一个复杂的领域划分为多个限界上下文，并且将其相互之间的关系用图画出来。这一划分过程对于单块和微服务架构两者都是有用的，而且就像前面有关“业务功能”一节中所讨论的那样，在服务和各个限界上下文之间所存在的自然的联动关系，能有助于澄清和强化这种划分。

   ![去中心化的数据管理](https://martinfowler.com/articles/microservices/images/decentralised-data.png)



## 7. **“基础设施”自动化**

   ![CI/CD](https://martinfowler.com/articles/microservices/images/basic-pipeline.png)



由于微服务的运行在各自的系统空间内，且经过服务拆分后服务的数目数倍于原有的传统的发布方式，因此带给运维的压力也比传统方式下要大得多；由于多服务之间存在版本依赖、配合敏捷开发的高频发布、发布后的问题排查与监控都相比于传统运维了有了质的区分，甚至有时候一天发布几十次服务的情况发生，在没有自动化流程的辅助则是非常难以实现的。

微服务与Docker技术的相互促进发展，Docker技术为微服务的自动化发布提供了良好的解决方案，也促使了 Devops 文化的不断演进，更多可以参见 [《一篇文了解DevOps：从概念、关键问题、兴起到实现需求》](https://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA%3D%3D&mid=2650994236&idx=1&sn=d488ae3d66328eb4344eea421ca679be&chksm=bdbf0e6f8ac88779d4bc011a7d4c40f0501c19227128276385f4e739ebacc53440f2a1169f3f)。

整体的目的则是通过打造CI/CD平台，为微服务提供一个自动化的基础设置。

   ![](https://martinfowler.com/articles/microservices/images/micro-deployment.png)



## 8. **“容错”设计** [待补充]

使用各个微服务来替代组件，其结果是各个应用程序需要设计成能够容忍这些服务所出现的故障。如果服务提供方不可用，那么任何对该服务的调用都会出现故障。客户端要尽可能优雅地应对这种情况。与一个单块设计相比，这是一个劣势。因为这会引人额外的复杂性来处理这种情况。为此，各个微服务团队在不断地反思：这些服务故障是如何影响用户体验的。

因为各个服务可以在任何时候发生故障，所以下面两件事就变得很重要，即能够快速地检测出故障，而且在可能的情况下能够自动恢复服务。各个微服务的应用都将大量的精力放到了应用程序的实时监控上，来检查“架构元素指标”（例如数据库每秒收到多少请求）和“业务相关指标”（例如系统每分钟收到多少订单）。当系统某个地方出现问题，语义监控系统能提供一个预警，来触发开发团队进行后续的跟进和调查工作。

由于 service 之间的相互调用，导致服务调用链的复杂性，很容易在一个 service 出现问题的时候，出现链式的错误，将故障放大，从而导致雪崩的现象出现。

因此从服务设计的角度讲需要进行一下措施：

1. 系统过载保护，当系统出现了过载的情况下，及时丢弃过载数据，返回服务出错状态；
2. 断路器方式，对于出现问题的 service 进行短路，并持续跟踪，等待恢复后重新进行；

可以参考 [B站微服务实践](https://github.com/gopherchina/conference/tree/master/2017)



## 9. **“演进式”设计** [待补充]

可替换性组件，非常利于微服务架构的演进。演进式设计承认难以对边界进行正确定位，所以它将工作的重点放到了易于对边界进行重构之上。

那些微服务的从业者们，通常具有演进式设计的背景，而且通常将服务的分解，视作一个额外的工具，来让应用开发人员能够控制应用系统中的变化，而无须减少变化的发生。变化控制并不一定意味着要减少变化——在正确的态度和工具的帮助下，就能在软件中让变化发生得频繁、快速且经过了良好的控制。

每当试图要将软件系统分解为各个组件时，就会面临这样的决策，即如何进行切分——我们决定切分应用系统时应该遵循的原则是什么？一个组件的关键属性，是具有独立更换和升级的特点**[13]**——这意味着，需要寻找这些点，即想象着能否在其中一个点上重写该组件，而无须影响该组件的其他合作组件。事实上，许多做微服务的团队会更进一步，他们明确地预期许多服务将来会报废，而不是守着这些服务做长期演进。

英国卫报网站是一个好例子。原先该网站是一个以单块系统的方式来设计和构建的应用系统，然而它已经开始向微服务方向进行演进了。原先的单块系统依旧是该网站的核心，但是在添加新特性时他们愿意以构建一些微服务的方式来进行添加，而这些微服务会去调用原先那个单块系统的API。当在开发那些本身就带有临时性特点的新特性时，这种方法就特别方便，例如开发那些报道一个体育赛事的专门页面。当使用一些快速的开发语言时，像这样的网站页面就能被快速地整合起来。而一旦赛事结束，这样页面就可以被删除。在一个金融机构中，我们已经看到了一些相似的做法，即针对一个市场机会，一些新的服务可以被添加进来。然后在几个月甚至几周之后，这些新服务就作废了。

这种强调可更换性的特点，是模块化设计一般性原则的一个特例，通过“变化模式”(pattern of change)**[14]**来驱动进行模块化的实现。大家都愿意将那些能在同时发生变化的东西，放到同一个模块中。系统中那些很少发生变化的部分，应该被放到不同的服务中，以区别于那些当前正在经历大量变动(churn)的部分。如果发现需要同时反复变更两个服务时，这就是它们两个需要被合并的一个信号。

把一个个组件放入一个个服务中，增大了作出更加精细的软件发布计划的机会。对于一个单块系统，任何变化都需要做一次整个应用系统的全量构建和部署。然而，对于一个个微服务来说，只需要重新部署修改过的那些服务就够了。这能简化并加快发布过程。但缺点是：必须要考虑当一个服务发生变化时，依赖它并对其进行消费的其他服务将无法工作。传统的集成方法是试图使用版本化来解决这个问题。但在微服务世界中，大家更喜欢将版本化作为最后万不得已的手段(http://martinfowler.com/articles/enterpriseREST.html#versioning)来使用。我们可以通过下述方法来避免许多版本化的工作，即把各个服务设计得尽量能够容错，来应对其所依赖的服务所发生的变化。



## 微服务之测试（补充）

### 测试金字塔

![测试金字塔](https://www.testwo.com/attachments/12697/1479217406521.png)

**UI(End-to-End Test)**   **Service**   **Unit**

**Unit Tes**t: 聚焦于函数内部的测试

**Service Tests**： Mocking or Stubbing

[“Testing Strategies in a Microservice Architecture”](https://martinfowler.com/articles/microservice-testing/)

## 参考资料

* [微服务架构9大特征中文版](https://mp.weixin.qq.com/s?__biz=MjM5MjEwNTEzOQ==&mid=401500724&idx=1&sn=4e42fa2ffcd5732ae044fe6a387a1cc3#rd)
* [基于Spring Boot和Spring Cloud实现微服务架构学习](http://blog.csdn.net/enweitech/article/details/52582918)
* [39本架构类书籍](http://download.csdn.net/album/detail/4093/1/1)
* [用gomock进行mock测试](https://segmentfault.com/a/1190000009894570)
* [Go语言用mock server模拟调用(httptest)](http://www.cnblogs.com/aguncn/p/7102675.html)
* [mock.js-无需等待，让前端独立于后端进行开发](https://cnodejs.org/topic/53f718218f44dfa3511af923)
* [Mocks Aren't Stubs](https://www.martinfowler.com/articles/mocksArentStubs.html)

