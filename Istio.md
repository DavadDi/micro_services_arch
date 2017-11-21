# Istio

## 1. 整体架构

Istio为希腊语，意思是“启航”。

![](https://avatars3.githubusercontent.com/u/23534644?s=200&v=4)

主要特点：

- HTTP、gRPC和TCP网络流量的自动负载均衡；
- 提供了丰富的路由规则，实现细粒度的网络流量行为控制；
- 流量加密、服务间认证，以及强身份声明；
- 全范围（Fleet-wide）策略执行；
- 深度遥测和报告。



设计目标:

* Maximize Transparency：最大透明化，可自动注入到服务中，对于服务透明，运维配置既可

* Incrementality： 能够增量增加新功能，扩展策略系统，完成新的控制策略

* Portability： 可移植性，能够花费很少的代价在各种生态环境下运行

* Policy Uniformity:  策略一致性，策略系统作为独特服务来进行维护

  ​
  <img src="https://istio.io/docs/concepts/what-is-istio/img/architecture/arch.svg" width="600"  align=center/>


从架构上 Istio 分成了两个层面：

* Control Plane： Pilot/Mixer/Istio-Auth，主要用于控制数据流的控制工作；主要参与者为Google、IBM
* Data Plane： Envoy，对于流经的数据流进行代理，并结合控制层的策略进行后续的动作; 主要参与者为 Lyft (Envoy)


Istio 的底层使用了[Envoy](https://lyft.github.io/envoy/)。Envoy 是 Lyft 于2016年9月份开源的一种服务代理和通信总线，已用于生产系统中，"管理了上万台虚拟机间的一百多个服务，每秒可处理近两百万次请求”。在近期的GlueCon 2017大会上，来自IBM的Shriram Rajagopalan和来自Google的Louis Ryan介绍了 Istio 的 [技术细节](https://istio.io/talks/istio_talk_gluecon_2017.pdf)（PDF）。

Envoy 以 Sidecar 的方式透明部署在应用 Service 前面，为服务提供了服务治理（Service Discovery）、断路器（Circuit Breaker）、流量管理分发（Traffic Management）等；将以前部署于 Service 中的普通规则和通用功能提炼到 Sidecar 中处理，极大地降低了 Service 开发的难度和成本。

在线体验 https://www.katacoda.com/courses/istio/deploy-istio-on-kubernetes

## 2. Pilot

核心组件 Pilot 主要用于控制 Envoy 进行流量管控，Pilot 管理Istio服务网格中的 Envoy 代理实例。Pilot 允许指定用于在Envoy代理之间路由流量的规则和故障恢复策略如超时（Timeout），重试（Retries）和熔断器（Circuit Breakers）。

通过 Pilot 设置的流量管理规则，服务 Service 可以方便实现基于比例和特定内容信息的流量分发。

<img src="https://istio.io/docs/concepts/traffic-management/img/pilot/TrafficManagementOverview.svg" width="600"  align=center/>

Pilot 的整体架构上分为 `Rules API` `Envoy API`  `Abstract Modle` 和 `Platform Adapter`。

* Rules API： 为使用者提供 Restful 接口，方便与通过接口控制流量规则策略
* Envoy API： 负责与各 Envoy 之间通信
* Abstract Modle： 为不同的平台 Service Discovery 提供抽象层
* Platform Adapter： 为不同的平台提供适配器，例如基于 K8S 平台，适配器需要通过监听 API Server的与流量规管理相关规则的变化情况，比如 pod 注册信息、ingress 资源、第三方资源等。

流量管控规则可以参见 [Traffic Management Rules](https://istio.io/docs/reference/config/traffic-rules/)。

<img src="https://istio.io/docs/concepts/traffic-management/img/pilot/PilotAdapters.svg" width="600"  align=center/>

**Discovery and Load Balancing**
<img src="https://istio.io/docs/concepts/traffic-management/img/pilot/LoadBalancing.svg" width="500" align=center>



Pilot 作为服务发现和治理情况下，会提供 Restfule 接口供 Envoy 获取服务后面对应的实例

```
- GET /v1/registration/:service
- GET /v1/registration/repo/:service_repo_name
- POST /v1/registration/:service
- DELETE /v1/registration/:service/:ip_address
- POST /v1/loadbalancing/:service/:ip_address
```

参见： [lyft discovery](https://github.com/lyft/discovery)



## Mixer

Mixer 负责在服务网格上执行访问控制和使用策略，并收集Envoy代理和其他服务的遥测数据。Mixer 旨在改变层之间的界限，以减少系统复杂性，从服务代码中消除策略逻辑，并替代为让运维人员控制。

Mixer 提供三个核心功能:

- **前提条件检查**。允许服务在响应来自服务消费者的传入请求之前验证一些前提条件。前提条件可以包括服务使用者是否被正确认证，是否在服务的白名单上，是否通过ACL检查等等。
- **配额管理**。使服务能够在多个维度上分配和释放配额，配额被用作相对简单的资源管理工具，以便在争取有限的资源时在服务消费者之间提供一些公平性。限速是配额的例子。
- **遥测报告**。使服务能够上报日志和监控。在未来，它还将启用针对服务运营商以及服务消费者的跟踪和计费流。

这些机制的应用是基于一组 [属性](https://istio.io/docs/concepts/policy-and-control/attributes.html) 的,这些属性为每个请求物化到 Mixer 中。在Istio内，Envoy重度依赖Mixer。在网格内运行的服务也可以使用Mixer上报遥测或管理配额。（注意：从Istio pre0.2起，只有Envoy可以调用Mixer。）定义的具体公共属性参见 [Attribute vocabulary](https://istio.io/docs/reference/config/mixer/attribute-vocabulary.html)。

<img src="https://istio.io/docs/concepts/policy-and-control/img/mixer/traffic.svg" width="600"  align=center/>

Istio 0.2 新的Adpter模型 [Mixer Adapter Model](https://istio.io/blog/mixer-adapter-model.html)，每个Request需要调用Mixer两次：Precondition Check和Report；

为了将构建服务的支撑功能与程序解耦合，Mixer 在 Envoy 与 支撑服务之间充当了一个中间层，Mixer 本身则采用 Adapter 机制与各种支撑的服务进行对接，同时允许运维操作人员注入和控制策略来控制服务与支撑服务交互，运维人员可以决定数据流向那个支撑服务进行认证、计费、监控等各种控制；

<img src="https://istio.io/docs/concepts/policy-and-control/img/mixer/adapters.svg" width="200"  align=center/>

Adapters 为 go 的 pkg 直接链接到 Mixer 的程序内，因此创建自己的 Adapter 也非常容易；



<img src="https://istio.io/docs/concepts/policy-and-control/img/mixer-config/machine.svg" width="600"  align=center/>

**Handlers**: Configuring Adapters
**Templates**: [Adapter Input Schema](https://istio.io/docs/reference/config/mixer/template/), [metric](https://istio.io/docs/reference/config/mixer/template/metric.html) 和 [logentry](https://istio.io/docs/reference/config/mixer/template/logentry.html)为两个常用的模板
**Instances**: Attribute Mapping
**Rules**: Delivering Data to Adapters



<img src="https://istio.io/docs/concepts/policy-and-control/img/mixer/phases.svg" width="500"  align=center/>



## istioctl

istioctl 命令用于创建、修改、删除配置等相关的 istio 系统的资源，包括以下：

* route-rule        连接到upstream的路由规则（client side)
* ingress-rule     ingress 相关的规则
* egress-rule      egress 相关的规则
* destination-policy   连接到destination的策略



## 部署 Bookinfos on K8S

[Install](https://istio.io/docs/guides/bookinfo.html) 手动注入方式：

```
/v1/registration/

[lyft discovery](https://github.com/lyft/discovery)
```

```shell
$ kubectl apply -f <(istioctl kube-inject -f samples/bookinfo/kube/bookinfo.yaml)

$ kubectl get services
$ kubectl get pods

$ export GATEWAY_URL=$(kubectl get po -n istio-system -l istio=ingress -o 'jsonpath={.items[0].status.hostIP}'):$(kubectl get svc istio-ingress -n istio-system -o 'jsonpath={.spec.ports[0].nodePort}')

$ echo $GATEWAY_URL

$ curl -o /dev/null -s -w "%{http_code}\n" http://${GATEWAY_URL}/productpage
```



在线体验 https://www.katacoda.com/courses/istio/deploy-istio-on-kubernetes



```
while true; do
>   curl -s https://2886795303-80-frugo01.environments.katacoda.com/productpage > /dev/null
>   echo -n .;
>   sleep 0.2
> done

```



http_proxy=$L5D_INGRESS_LB curl -s http://hello



## Linkerd

Linkerd 启动后会在每个 Node 上通过 DeamonSet 起一个代理或者作为 Sidercar 的方式和程序部署在一起。通过导出 http_proxy 环境变量的方式，为Application 提供透明的代理。HelloWorld的 [yaml](https://raw.githubusercontent.com/linkerd/linkerd-examples/master/k8s-daemonset/k8s/hello-world.yml)文件。

**No external LoadBalancer IPs：minikube**

在 Minikube 中 NODE_NAME 不能够获取，具体操作可以参见： https://discourse.linkerd.io/t/flavors-of-kubernetes/53

minikube ip: 192.168.99.100

```shell
$ kubectl apply -f https://raw.githubusercontent.com/linkerd/linkerd-examples/master/k8s-daemonset/k8s/linkerd.yml
$ kubectl apply -f https://raw.githubusercontent.com/linkerd/linkerd-examples/master/k8s-daemonset/k8s/hello-world-legacy.yml

# 通过http代理测试
$ OUTGOING_PORT=$(kubectl get svc l5d -o jsonpath='{.spec.ports[?(@.name=="outgoing")].nodePort}')
#$ L5D_INGRESS_LB=http://192.168.99.100:$OUTGOING_PORT
$ L5D_INGRESS_LB=http://${minikube ip}:$OUTGOING_PORT
$ http_proxy=$L5D_INGRESS_LB curl -s http://hello
$ http_proxy=$L5D_INGRESS_LB curl -s http://world

#or
$curl -v http://$L5D_INGRESS_LB/ -H 'Host: hello'

# 管理页面查看
$ ADMIN_PORT=$(kubectl get svc l5d -o jsonpath='{.spec.ports[?(@.name=="admin")].nodePort}')
$ open http://$(minikube ip):$ADMIN_PORT
```

由于 Minikube 中不能够获取到真实的 NODE_NAME，因此 [HelloWorld](https://github.com/linkerd/linkerd-examples/tree/master/docker/helloworld) 的样例中通过在本地起一个 K8S Cluster集群的代理 proxy，而 [hostIP.sh](https://github.com/linkerd/linkerd-examples/blob/master/docker/helloworld/hostIP.sh) 的脚本则通过自身的 POD_NAME，获取对应的IP地址。

```shell
$ cat hostIP.sh
#!/bin/sh

set -e

sleep 10
curl -s "${K8S_API:-localhost:8001}/api/v1/namespaces/$NS/pods/$POD_NAME" | jq '.status.hostIP' | sed 's/"//g'
```

[hello-world-legacy.yml](https://raw.githubusercontent.com/linkerd/linkerd-examples/master/k8s-daemonset/k8s/hello-world-legacy.yml) 完整的样例如下： （导出的环境变量方法参见：https://kubernetes.io/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/）

```yaml
---
apiVersion: v1
kind: ReplicationController
metadata:
  name: hello
spec:
  replicas: 3
  selector:
    app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      dnsPolicy: ClusterFirst
      containers:
      - name: service
        image: buoyantio/helloworld:0.1.4
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: NS
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        command:
        - "/bin/sh"
        - "-c"
        - "http_proxy=`hostIP.sh`:4140 helloworld -addr=:7777 -text=Hello -target=world"
        ports:
        - name: service
          containerPort: 7777
      - name: kubectl
        image: buoyantio/kubectl:v1.4.0
        args:
        - proxy
        - "-p"
        - "8001"
---
apiVersion: v1
kind: Service
metadata:
  name: hello
spec:
  selector:
    app: hello
  clusterIP: None
  ports:
  - name: http
    port: 7777
---
apiVersion: v1
kind: ReplicationController
metadata:
  name: world-v1
spec:
  replicas: 3
  selector:
    app: world-v1
  template:
    metadata:
      labels:
        app: world-v1
    spec:
      dnsPolicy: ClusterFirst
      containers:
      - name: service
        image: buoyantio/helloworld:0.1.4
        env:
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: TARGET_WORLD
          value: world
        args:
        - "-addr=:7778"
        ports:
        - name: service
          containerPort: 7778
---
apiVersion: v1
kind: Service
metadata:
  name: world-v1
spec:
  selector:
    app: world-v1
  clusterIP: None
  ports:
  - name: http
    port: 7778
```



删除 DaemonSet

```shell
$ kubectl delete daemonset -l app=l5d
```



查看监控指标：

```shell
$ curl http://192.168.99.100:32407/admin/metrics.json\?pretty=1

# promethus相关的参数
$ http://192.168.99.100:32407/admin/metrics/prometheus
```



Application的配置需要添加一下相关配置：

```
        env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: http_proxy
          value: $(NODE_NAME):4140
```

其中：

- `$(NODE_NAME):4140` (outgoing) for HTTP   4041(incoming)  4042 (ingress)
- `$(NODE_NAME):4240` for HTTP/2
- `$(NODE_NAME):4340` for gRPC



统计相关的服务：

 [Prometheus](https://prometheus.io/), [InfluxDB](https://www.influxdata.com/), and [StatsD](https://github.com/etsy/statsd)

http://121.196.214.67:30173/?router=outgoing



http_proxy=http://121.196.214.67:31393/ curl -s http://hello

# 参考

1. [Service Mesh：下一代微服务](https://mp.weixin.qq.com/s?__biz=MzA3MDg4Nzc2NQ==&mid=2652136254&idx=1&sn=bba9bbd24ac8e5c1f6ef5d1125a6975b&chksm=84d53304b3a2ba12f88675c1bf51973aa1210d174da9e6c2ddcd1f3c84ec7e25987b3bce1071&mpshare=1&scene=1&srcid=1020GPmfbEVP9QDNlZBHg47I&pass_ticket=a%2B3t43zt60SHoI6fLsq80dbx%2FKCTnp9%2Bg1DgmORXY0hwwje1mB3uFmK9f9%2BSNZ2v#rd): QCON 2017 上海站的演讲，系统介绍Service Mesh技术
2. [ 服务网格新生代--Istio](https://mp.weixin.qq.com/s?__biz=MzA3MDg4Nzc2NQ==&mid=2652136078&idx=1&sn=b261631ffe4df0638c448b0c71497021&chksm=84d532b4b3a2bba2c1ed22a62f4845eb9b6f70f92ad9506036200f84220d9af2e28639a22045&mpshare=1&scene=1&srcid=0922JYb4MpqpQCauaT9B4Xrx&pass_ticket=F8CjNuTDg%2Fskt94bwJ%2B1yiPKpHJhaaRYpxDCqtNGMrMGkGsZDLF5EW1HCByba35u#rd): 介绍isito的文章
3. [Envoy Docs](https://www.envoyproxy.io/docs/envoy/latest/)
4. [Istio Lab](https://www.katacoda.com/courses/istio)
5. [Monitoring Microservices with Weavescope](https://www.youtube.com/watch?v=aQcXOajWwE4)
6. http://blog.fleeto.us/content/istio-de-zi-dong-zhu-ru istio 的自动注入
7. [Microservices Patterns With Envoy Sidecar ](http://blog.christianposta.com/microservices/01-microservices-patterns-with-envoy-proxy-part-i-circuit-breaking/)
8. [istio 三日谈](https://www.kubernetes.org.cn/2449.html)
9. [KUBERNETES-NATIVE API GATEWAY FOR MICROSERVICES BUILT ON THE **ENVOY PROXY**](https://www.getambassador.io/)
10. [Docker应用的可视化监控管理](http://blog.csdn.net/horsefoot/article/details/51749528)
11. [[Generating code](https://blog.golang.org/generate)](https://blog.golang.org/generate)
12. [Linkerd中文文档](https://linkerd.doczh.cn/doc/overview/) [官方文档](https://linkerd.io/)
13. [A SERVICE MESH FOR KUBERNETES](https://buoyant.io/2016/10/04/a-service-mesh-for-kubernetes-part-i-top-line-service-metrics/) 介绍了Linkerd 在 k8s上的集成测试方案
14. [CNCF Landscape](https://github.com/cncf/landscape)