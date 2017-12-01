# OpenShift Origin Singnal Node 安装

[openshift origin](https://github.com/openshift/origin/) 是用来支持 [openshift](https://www.openshift.org/) 产品的一个上游社区项目，围绕 Docker 容器和 Kubernetes 集群技术，一套来进行应用生命周期管理的 DevOps 工具，它提供了一个完整的开源容器应用平台。

与 openshift 类型的产品 有 [Pivotal](http://www.36dsj.com/archives/tag/pivotal) 公司 ([中国官网](https://pivotal.io/cn))的 [CloudFoundry](https://www.cloudfoundry.org/) 和 [Rancher](https://www.cnrancher.com/)，国内 Ranchard 的接受度是远高于 Openshift。



* Pivotal公司是由EMC和VMware联合成立的一家子公司，主要产品有  [CloudFoundry](https://www.cloudfoundry.org/)/[Spring](https://spring.io/)(SpringBoot/SpringCould)。

* Rancher是全球唯一提供Kubernetes、Mesos和Swarm三种调度工具的企业级分发版和商业技术支持的容器管理平台

* [Deis](https://deis.com/) Build powerful, open source tools that make it easy for teams to create and manage applications on Kubernetes. Projects: Workflow/Helm/Steward

* [Eldarion](https://eldarion.cloud/) Hands-free DevOps powered by Kubernetes. Eldarion uniquely offers expert, white-glove DevOps services together with a new, highly-scalable PaaS.

  ​

**扩展**：[**awesome-kubernetes**](https://github.com/ramitsurana/awesome-kubernetes)

**Kubernetes Platform as a Service providers**

- [Deis Workflow](https://deis.com/) - [Deprecated Public Releases](https://deis.com/blog/2017/deis-workflow-final-release/)
- [Kel](http://www.kelproject.com/)
- [WSO2](http://wso2.com/)
- [Kumoru](https://medium.com/@kumoru_io) - [Deprecated](https://www.youtube.com/watch?v=_5XQmE7rx9o) - Not Official
- [Rancher](http://rancher.com/running-kubernetes-aws-rancher/)
- [OpenShift Origin](https://www.openshift.org/)
- [Eldarion Cloud](http://eldarion.cloud/)
- [IBM Bluemix Container Service](https://www.ibm.com/cloud-computing/bluemix/containers)

![](http://soft.dog/images/openshift/openshift_architecture.png)



[在线文档地址](https://docs.openshift.org/index.html): https://docs.openshift.org/index.html  [OpenShift Origin 3.6 Documentation](https://docs.openshift.org/3.6/welcome/index.html)

版本关系图：

![](http://soft.dog/images/openshift/openshift_relationship.jpg)

**Docker K8S 与Openshift版本对应关系图：** （kubernets创建于 2014-06-06）

![](https://www.duyidong.com/images/container_timeline.png)

备注： Centos 7.2 版本，已经安装了 docker 版本

```shell
# docker version
Client:
 Version:      17.03.1-ce
 API version:  1.27
 Go version:   go1.7.5
 Git commit:   c6d412e
 Built:        Mon Mar 27 17:05:44 2017
 OS/Arch:      linux/amd64

Server:
 Version:      17.03.1-ce
 API version:  1.27 (minimum version 1.12)
 Go version:   go1.7.5
 Git commit:   c6d412e
 Built:        Mon Mar 27 17:05:44 2017
 OS/Arch:      linux/amd64
 Experimental: false

# 如果未安装则可用一下命令安装
# yum install docker
# systemctl start docker
```



安装步骤：

```shell
# hostnamectl
   Static hostname: teambition
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 99a0b5c74c754c0c8d94775a4d1a753a
           Boot ID: e4d29bee23a64794ac9c13408846e90a
    Virtualization: vmware
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.el7.x86_64
      Architecture: x86-64
      
# yum makecache fast
# setenforce 0
# yum  list all | grep openshift  # OpenShift 3.6的版本，集成的K8s 1.6
# yum info centos-release-openshift-origin36.noarch
# yum install centos-release-openshift-origin36
# yum install centos-release-openshift-origin36.noarch
# rpm -ql centos-release-openshift-origin36.noarch
# yum install origin
# openshift start
# openshift version
openshift v3.6.1+008f2d5
kubernetes v1.6.1+5115d708d7
etcd 3.2.1

```

登录：

[https://masterip:8443](https://masterip:8443/)  信任默认证书，默认用户名为 test:test



**启动过程中错误处理：**

misconfiguration: kubelet cgroup driver: "cgroupfs" is different from docker cgroup driver: "systemd"](https://github.com/kubernetes/kubernetes/issues/43805)

```shell
 $ systemctl cat docker
 $ vim /usr/lib/systemd/system/docker.service
 ExecStart=/usr/bin/dockerd --exec-opt native.cgroupdriver=systemd
 $ systemctl daemon-reload
 $ systemctl restart docker
```



## 参考

1. [OpenShift介绍](http://www.chenshake.com/openshift%e4%bb%8b%e7%bb%8d/)
2. [安装 OpenShift Origin](http://soft.dog/2017/07/27/install-openshift-origin/)
3. [Installing Kubernetes 1.6.4 on centos 7](https://gettech1.wordpress.com/2017/06/13/installing-kubernetes-1-6-4-on-centos-7/)
4. [PaaS 平台（一） -- Openshift 介绍](https://www.duyidong.com/2017/06/14/kubernetes-and-openshift/)
5. [PaaS 平台（三）-- Openshift 使用](https://www.duyidong.com/2017/06/15/openshift-quick-start/)
6. [Who Manages Cloud IaaS, PaaS, and SaaS Services](https://mycloudblog7.wordpress.com/2013/06/19/who-manages-cloud-iaas-paas-and-saas-services/)