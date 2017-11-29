# DooD or DinD On K8S

K8S 中的 Pod 相对于 Docker Container 具备以下的优点：

1. 每个 Pod 有独立的在 Cluster 内部可以路由的地址

2. 运行在单一 Pod 中的 Containers 具有相同的 network namespace，可以通过 localhost 进行访问

3. 运行的 Pod 在结束后，可以被 Cluster 进行回收

   ​

在使用 Docker 技术做 CI/CD 过程中，一般会使用到两种技术：

1. **DinD **(Docker-in-Docker)，在 Docker 运行的 赋予 privileged 权限的 Container 中运行 Docker 进行版本编译或者 CI/CD 相关工作，与宿主机达到完全隔离，主要用于多个环境同时运行的测试；

2. **DooD** (Docker-outside-of-Docker)，在 Docker 运行的 继承宿主机 docker.sock 的 Container ，从而达到在宿主机 Docker Daemon 程序中创建其兄弟 Container 的，从而达到复用宿主机中镜像等各种资源的目的，主要用于编译版本或者编译镜像、推送镜像；

   ```shell
   $ docker run -d -v /var/run/docker.sock:/var/run/docker.sock \
                   -v $(which docker):/usr/bin/docker -p 8080:8080 myjenk
   ```

   ​

   ​

## 1. Pods and DooD

   ![](http://30ux233xk6rt3h0hse1xnq9f-wpengine.netdna-ssl.com/wp-content/uploads/2017/03/dood-1-1.png)



**Docker Outside Docker** [Running Docker in Jenkins (in Docker)](http://container-solutions.com/running-docker-in-jenkins-in-docker/)

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
    name: dood 
spec: 
    containers: 
      - name: docker-cmds 
        image: docker:1.12.6 
        command: ['docker', 'run', '-p', '80:80', 'httpd:latest'] 
        resources: 
            requests: 
                cpu: 10m 
                memory: 256Mi 
        volumeMounts: 
          - mountPath: /var/run 
            name: docker-sock 
    volumes: 
      - name: docker-sock 
        hostPath: 
            path: /var/run 
```



## 2. Pods and DinD 

由于在 Docker 中运行 Docker，能够达到与宿主机环境完全隔离的目的，但是也有一些存在的小问题：

1. 因此运行的外部 Container 的时候需要赋予 privileged 权限 (docker run --privileged)
2. Storage Drivers 叠加带来的问题：如果外部的 Docker 运行的文件系统（EXT4/BETRFS/AUFS..）与 内部 Docker 的文件系统（EXT4/BETRFS/Device Mapper）在叠加的时候可能存在不能够正常工作的情况，例如 不能在 AUFS 之上运行 AUFS；在 BETRFS 之上运行 BETRFS 可能一开始能够正常工作，但是在嵌套卷中不能够工作；Device Mapper不具备 namespaced 能力，同一个机器上的多个 Docker 实例可以相互影响 Images 和 Devices；备注：``上述问题可能随着 Docker 版本的演进，已经得到了解决``。
3. 不能够共享宿主机上的 Images 资源，每次运行都需要重新拉去相关的 Images

![](http://30ux233xk6rt3h0hse1xnq9f-wpengine.netdna-ssl.com/wp-content/uploads/2017/03/dind.png)


****Pods** and DinD** [Using Docker-in-Docker for your CI or testing environment? Think twice.](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/)

```yaml
apiVersion: v1 
kind: Pod 
metadata: 
    name: dind 
spec: 
    containers: 
      - name: docker-cmds 
        image: docker:1.12.6 
        command: ['docker', 'run', '-p', '80:80', 'httpd:latest'] 
        resources: 
            requests: 
                cpu: 10m 
                memory: 256Mi 
        env: 
          - name: DOCKER_HOST 
            value: tcp://localhost:2375 
      - name: dind-daemon 
        image: docker:1.12.6-dind 
        resources: 
            requests: 
                cpu: 20m 
                memory: 512Mi 
        securityContext: 
            privileged: true 
        volumeMounts: 
          - name: docker-graph-storage 
            mountPath: /var/lib/docker 
    volumes: 
      - name: docker-graph-storage 
        emptyDir: {}
```


## 3. 参考

1. [A Case for Docker-in-Docker on Kubernetes One](https://applatix.com/case-docker-docker-kubernetes-part/)
2. [A Case for Docker-in-Docker on Kubernetes Two](https://applatix.com/case-docker-docker-kubernetes-part-2/)