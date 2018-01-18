* [浅谈命令查询职责分离(CQRS)模式](http://www.cnblogs.com/yangecnu/p/Introduction-CQRS.html)
* [CQRS revisited](https://lostechies.com/gabrielschenker/2015/04/07/cqrs-revisited/)
* [CQRS applied](https://lostechies.com/gabrielschenker/2015/04/12/cqrs-applied/)
* [在洋葱（Onion）架构中实现领域驱动设计](http://www.infoq.com/cn/news/2014/11/ddd-onion-architecture)
* [Understanding Onion Architecture](http://blog.thedigitalgroup.com/chetanv/2015/07/06/understanding-onion-architecture/)
* [The Onion Architecture](http://jeffreypalermo.com/blog/the-onion-architecture-part-1/)
* [Hexagonal architecture](http://alistair.cockburn.us/Hexagonal+architecture)
* [The Clean Architecture](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html)



![](https://lostechies.com/gabrielschenker/files/2015/04/Old-architecture1.png)

![](https://lostechies.com/gabrielschenker/files/2015/04/Event-sourcing.png)

![](https://8thlight.com/blog/assets/posts/2012-08-13-the-clean-architecture/CleanArchitecture.jpg)

![](http://jeffreypalermo.com/files/media/image/WindowsLiveWriter/TheOnionArchitecturepart1_70A9/image%7B0%7D%5B59%5D.png)

![](http://alistair.cockburn.us/get/2304)



[技术架构演变的全景图- 从单体式到云原生](https://mp.weixin.qq.com/s?__biz=MzAxNjgyODc5OA==&mid=2650730188&idx=1&sn=9414d894e5d3cbb7058d1a5e03a946d0&chksm=83e49eddb49317cb8f5c69df6dfdc26eaad6eac852adf0e4d7ac3181286e97989e4176daac4d&mpshare=1&scene=1&srcid=1224c2OstR5LymrPWpnDKEs5#rd)  包括微服务2015-2018的流行趋势图

![](https://mmbiz.qpic.cn/mmbiz_jpg/C98a4cEgAfLt2nrM4VdP5BpKUGfILwtfRxaibS8muFAr6Opia24T9jYZF8dYUwjt3KwyJThaf4SdJiciaaLse9lK7A/?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)



![](https://jimmysong.io/kubernetes-handbook/images/kubernetes-high-level-component-archtecture.jpg)



下图是[Bilgin Ibryam](https://developers.redhat.com/blog/author/bibryam/)给出的微服务中应该关心的主题，图片来自[RedHat Developers](https://developers.redhat.com/blog/2016/12/09/spring-cloud-for-microservices-compared-to-kubernetes/)。

![](https://raw.githubusercontent.com/rootsongjc/kubernetes-handbook/master/images/microservices-concerns.jpg)

CNCF（云原生计算基金会）给出了云原生应用的三大特征：
容器化包装：软件应用的进程应该包装在容器中独立运行。
动态管理：通过集中式的编排调度系统来动态的管理和调度。
微服务化：明确服务间的依赖，互相解耦。



From: https://www.gartner.com/smarterwithgartner/top-trends-in-the-gartner-hype-cycle-for-emerging-technologies-2017/

[Awesome Cloud Native](https://github.com/rootsongjc/awesome-cloud-native)

![](https://blogs.gartner.com/smarterwithgartner/files/2017/08/Emerging-Technology-Hype-Cycle-for-2017_Infographic_R6A.jpg)



![](http://5b0988e595225.cdn.sohucs.com/images/20171011/e8da96d00a264ef88e026119e1f02af6.jpeg)

![](http://scholarsupdate.hi2net.com/upload2013/2015121718124767.jpg)

> “快鱼吃慢鱼”是思科CEO钱伯斯的名言，他认为“在Internet经济下，大公司不一定打败小公司，但是快的一定会打败慢的。Internet与工业革命的不同点之一是，你不必占有大量资金，哪里有机会，资本就很快会在哪里重新组合。速度会转换为市场份额、利润率和经验”。“快鱼吃慢鱼”强调了对市场机会和客户需求的快速反应，但决不是追求盲目扩张和仓促出击，正相反，真正的快鱼追求的不仅是快，更是“准”，因为只有准确的把握住市场的脉搏，了解未来技术或服务的方向后，快速出击进行收购才是必要而有效的。
>
> https://baike.baidu.com/item/%E5%BF%AB%E9%B1%BC%E5%90%83%E6%85%A2%E9%B1%BC



精益大师Mary Poppendick提出的问题了——“如果只是改变了应用的一行代码，您的组织需要多长时间才能把应用部署到线上？“答案是几分钟或几秒钟。

云原生应用架构在快速变动的需求、稳定性、可用性和耐久性之间寻求平衡。

云原生应用程序架构还通过诸如API网关之类的设计模式来支持移动优先开发的概念，API网关将服务聚合负担转移回服务器端。

From: https://jimmysong.io/migrating-to-cloud-native-application-architectures/chapter1/why-cloud-native-application-architectures.html

[到了检讨“快鱼吃慢鱼”理论的时候](https://www.huxiu.com/article/3194.html)  Apple TV

> *Technology is exciting and challenging, but lessons learned from the industry are not to start there, but with your business mission, vision, and your people.*



>  [Pivotal](https://pivotal.io/) 是云原生应用的提出者，并推出了 [Pivotal Cloud Foundry](https://pivotal.io/platform) 云原生应用平台和 [Spring](https://spring.io/) 开源 Java 开发框架，成为云原生应用架构中先驱者和探路者。

### What are Cloud-Native Applications?

> > “One of the things we've learned is that if you can't get it to market more quickly, there is no doubt that the market will have changed and no matter how well you've engineered it or built it or deployed it or trained your folks, it's not going to be quite right because it's just a little too late.“
>
> **James McGlennon** 
> Executive VP and CIO, Liberty Mutual Insurance Group ！[From](https://pivotal.io/cloud-native)



>  The [Cloud Native Computing Foundation (CNCF)](https://cncf.io/) has defined cloud native as:
>  - Containerized
>  - Distributed Management and Orchestration
>  - Micro-services Architecture

![](https://d1fto35gcfffzn.cloudfront.net/images/topics/cloudnative/diagram-cloud-native.png)

From: https://pivotal.io/cloud-native



### [Cloud Native Reference architecture]()

![](https://www.cncf.io/wp-content/uploads/2017/05/1.png)

From: https://www.cncf.io/blog/2017/05/15/developing-cloud-native-applications/  

*OSS*:Operation support system

*BSS*:Business support system



# The Reactive Manifesto

![](https://www.reactivemanifesto.org/images/reactive-traits.svg)

From: https://www.reactivemanifesto.org/



1. [What are Cloud-Native Applications?](https://pivotal.io/cloud-native)  [Developing Cloud Native Applications](https://www.cncf.io/blog/2017/05/15/developing-cloud-native-applications/)
2. [Comparing PaaS and Container Platforms By Wikibon](https://www.openshift.com/container-platform/compare-openshift.html)
3. [The Journey to Cloud Native](https://blogs.cisco.com/cloud/the-journey-to-cloud-native)
4. [迁移到云原生应用架构ebook](https://jimmysong.io/migrating-to-cloud-native-application-architectures/) [Migrating to Cloud-Native Application Architectures](http://www.oreilly.com/programming/free/migrating-cloud-native-application-architectures.csp)
5. [2016 Container Report By Cloud Foundry](https://www.cloudfoundry.org/container-report-2016/)
6. [2017 Container Report By Cloud Foundry](https://cloudfoundry.org/container-report-2017/?utm_source=pr&utm_campaign=cr17&utm_content=cff)
7. [The Reactive Manifesto](https://www.reactivemanifesto.org/)