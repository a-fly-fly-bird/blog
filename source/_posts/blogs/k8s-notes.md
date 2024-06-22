---
title: Docker容器化 -> K8s -> Openshift笔记
toc: true
cover: https://source.unsplash.com/random
tags: ['Docker', 'K8s']
categories: ['后端']
date: 2024-06-20 21:56:31
---

缺乏系统的实践，每次都会遗忘很多关键内容，于是我有了梳理整个容器化的发展和如何配置的想法。由于我参考的文档已经非常详细和完备了，所以本文更多是作为实践记录和补充材料。

<!-- more -->

<div class="alert-box">
  <span class="alert-icon">⚠️</span>
  <div class="alert-text">
    <p>
    注意：这更像一篇笔记而不是博客，重心是查缺补漏而没有遵守阮一峰老师的<a href="https://github.com/ruanyf/weekly/blob/master/docs/issue-288.md#技术写作的首要诀窍">技术写作的首要诀窍</a>。
    </p>
    <p>
    你可能不会从这里收获什么东西。
    </p>
  </div>
</div>

<style>
  .alert-box {
    display: grid;
    grid-template-columns: auto 1fr;
    align-items: start;
    padding: 15px;
    margin: 20px 0;
    border: 1px solid #f5c6cb;
    background-color: #f8d7da;
    color: #721c24;
    border-radius: 5px;
    font-family: Arial, sans-serif;
  }

  .alert-icon {
    font-size: 1.5em;
    margin-right: 20px;
  }

  .alert-text {
    align-self: start;
  }
</style>

# 容器化

容器是一种操作系统虚拟化形式。可以使用一个容器来运行从小型微服务或软件进程到大型应用程序的所有内容。容器包含所有必要的可执行文件、二进制代码、库和配置文件。但是，与服务器或计算机虚拟化方法不同，**容器不包含操作系统映像**。因此，它们更轻便且可移植，其开销很小。在大型应用程序部署中，可以将多个容器部署为一个或多个容器集群。此类集群可由 Kubernetes 等容器编排程序管理。

![alt text](container-vs-vm-inline1_tcm19-82163.png.avif)

了解更多: [什么是容器？](https://www.netapp.com/zh-hans/devops-solutions/what-are-containers/) 和 [Containers vs. Virtual Machines (VMs): What's the Difference?](https://www.netapp.com/blog/containers-vs-vms/)。

# Docker
入门资料：
- [Docker — 从入门到实践](https://github.com/yeasy/docker_practice)

Docker是一种容器的运行时实现，可以说是事实上的行业标准。

> Docker Engine is the industry’s de facto container runtime that runs on various Linux (CentOS, Debian, Fedora, RHEL, and Ubuntu) and Windows Server operating systems.

## Docker Desktop

根据[Getting Started with Docker Desktop](https://www.docker.com/blog/getting-started-with-docker-desktop/):
> Docker Desktop is an easy-to-install application and includes Docker Engine, Docker CLI client, Docker Compose, Docker Content Trust, Kubernetes, and Credential Helper.

所以Docker Desktop 集成好了所有关键的要素，比如Engine, CLI, GUI等等，对于个人用户来说，直接安装Desktop是最省时省力的。

## 安装

对于Mac和Windows用户来说，不太好单独安装Engine（Docker Engine on Linux, also known as Docker CE）和CLI，直接安装Desktop即可。Linux暂时还没有Desktop，可以通过`docker-ce docker-ce-cli`使用。

各个平台的安装都可以查看[ocker — 从入门到实践 - 安装 Docker](https://yeasy.gitbook.io/docker_practice/install)，非常细致。

## Docker Engine 支持平台
查看官方的[Install Docker Engine](https://docs.docker.com/engine/install/)文档。

## 底层实现
Docker 底层的核心技术包括 Linux 上的命名空间（Namespaces）、控制组（Control groups）、Union 文件系统（Union file systems）和容器格式（Container format）。

### Docker architecture on MacOS
Docker在MacOS上的底层实现有一些区别，简而言之底层是一个Linux。

根据 [Under the Hood: Demystifying Docker Desktop For Mac](https://collabnix.com/how-docker-for-mac-works-under-the-hood/)：
> There is no docker0 bridge on macOS. Because of the way networking is implemented in Docker for Mac, you cannot see a docker0 interface on the host. This interface is actually within the virtual machine.

# k8s
入门资料：
- [Kubernetes Tutorials ｜ k8s 教程](https://github.com/guangzhengli/k8s-tutorials)

## minikube
> [minikube](https://github.com/kubernetes/minikube) is local Kubernetes, focusing on making it easy to learn and develop for Kubernetes.

所以minikube非常适合用户用来学习入门Kubernetes。

### minukube dashboard

`minukube dashboard`命令可以提供一个Web的Dashboard，很像我平时使用的Openshift的Dashboard。

### kubectl
By default, kubectl gets configured to access the kubernetes cluster control plane inside minikube when the minikube start command is executed.

安装minikube会默认安装并配置好kubectl。

## Docker Compose

Docker Compose is a tool for defining and running multi-container applications. It is the key to unlocking a streamlined and efficient development and deployment experience.

简而言之，Docker Compose也是用来做容器编排的，Docker seamless 支持，[Docker Compose 和 Kubernetes的对比](https://stackoverflow.com/a/47537046/23382462):

> Docker Compose is the declarative version of the docker cli
> - It can start one or more containers
> - It can create one or more networks and attach containers to them
> - It can create one or more volumes and configure containers to mount them
> - All of this is for use on a single host
> 
> Docker Classic Swarm
> 
> - Docker swarm has been abandoned by Docker Inc. and is not being actively maintained or supported.
> - Docker Swarm is for running and connecting containers on multiple hosts.
> - Docker Swarm is a container cluster management and orchestration tool.
> - It manages containers running on multiple hosts and does things like scaling, starting a new container when one crashes, networking containers ...
> - The Docker Swarm file named stack file is very similar to a Docker Compose file
> - The only comparison between Kubernetes and Compose is at the most trivial and unimportant level: they both run containers, but this says nothing to help one understand what the two tools are and where they are useful. They are both useful for different things
> 
> Kubernetes
> 
> - Kubernetes (K8S) is a distributed container orchestration tool initially created by Google
> - It was open-sourced in 2014 and handed over to the Cloud Native Computing Foundation (CNCF) the following year
> - The CNCF is an industry body with hundreds of members drawn from the majority of large cloud, software and hardware companies
> - At the time of writing (late 2021) there are nearly a thousand K8S related projects split into around twenty classes with a total of over $21 billion dollars in funding
> - Kubernetes (2021) is the most popular distributed system orchestrator in the world with 88% adoption
> - Because of its near ubiquity, K8S has become the most popular contemporary platform for innovative system development in 2021
> - Kubernetes is a competitor (more or less) to Docker swarm but does more stuff than docker swarm i.e a popular choice.

Docker 提供的 Docker Compose 案例：[awesome-compose](https://github.com/docker/awesome-compose/)

## Multi-platform images

[Multi-platform images](https://docs.docker.com/build/building/multi-platform/) 是指一个image有适配多个平台的镜像，拉取镜像会自动选择合适架构的镜像。build时可以添加`--platform`来指定build镜像支持的架构。

# Openshift
入门资料：
- [OpenShift Guide](https://github.com/mikeroyal/OpenShift-Guide)

## OpenShift 和 Kubernetes

根据[Red Hat OpenShift vs. Kubernetes](https://www.redhat.com/en/technologies/cloud-computing/openshift/red-hat-openshift-kubernetes)：

> Red Hat® OpenShift® is an enterprise grade open source application platform for accelerating the development and delivery of cloud-native applications in a consistent way across the hybrid and multi cloud, all the way to the edge.
> It is powered by Kubernetes as the container orchestration engine, and many more features from the CNCF open source ecosystem, all tested, packaged, and supported together as a comprehensive application platform by Red Hat.

简而言之，OpenShift 是 Kubernetes 的 商业版。我目前所在的公司就使用Openshift作为基础设施，所以对于我来说，还是很有必要单独拿出来。

## OpenShift 和 Kubernetes 一些作用相同但实现不同的概念

阅读Openshift的博客，类似Route和Ingress，template和helm，很多时候是因为Openshift作为商业版需要这些功能，而k8s还没有提供，因此Openshift走在k8s前面，并显著影响了k8s这部分功能的设计和实现。

> Where OpenShift differs, it is often where no equivalent feature existed in the upstream core version at the time Red Hat needed to deliver it. OpenShift Routes, for example, predate the related Ingress resource that has since emerged in upstream Kubernetes. In fact, Routes and the OpenShift experience supporting them in production environments helped influence the later Ingress design, and that’s exactly what participation in a community like Kubernetes is all about.
> Another concept unique to Openshift is the Template object. An OpenShift Template bundles some number of objects that are related from a functional point of view.
> Kubernetes is a vibrant laboratory, and so there are similar efforts in the community to group an “application”’s objects and resources in a way convenient for deployment, management, and organization. One of the other popular systems in this space is Microsoft/Deis’s Helm. In this post, I’ll show how an OpenShift Template can be converted into a Helm Chart with just a few changes.

对此，网络上有很多的讨论。

### Kubernetes Helm charts or OpenShift templates?
- [What is better to use in OpenShift: Kubernetes Helm charts or OpenShift templates?](https://www.reddit.com/r/openshift/comments/fyo8rb/what_is_better_to_use_in_openshift_kubernetes/)
- [From Templates to Openshift Helm Charts](https://www.redhat.com/en/blog/from-templates-to-openshift-helm-charts)
- [Kubernetes Deployments Versus Deployment Configurations](https://docs.openshift.com/container-platform/3.9/dev_guide/deployments/kubernetes_deployments.html#kubernetes-deployments-vs-deployment-configurations)
  
还有将templates转化为Helm charts的工具：
- [Template2Helm](https://github.com/redhat-cop/template2helm)

## OpenShift Template

> 您可以定义新模板，以便轻松重新创建应用程序的所有对象。该模板将定义由其创建的对象以及一些元数据，以指导创建这些对象。

通过Openshift提供的模板，可以在一个文件里定义所有资源，同时使其模板化。

Openshift提供的[编写模板](https://docs.redhat.com/zh_hans/documentation/openshift_container_platform/4.4/html/images/templates-writing_using-templates)和[k8s组织资源配置](https://kubernetes.io/zh-cn/docs/concepts/workloads/management/#组织资源配置)类似但功能更强大。除此之外，还有一个解决方案是[Helm 和 Helm Charts](https://helm.sh)

# 实践篇
## Docker 和 Container
作者使用的是Go项目作为Demo，那我将以我熟悉的Angular项目作为案例（不熟悉Angular的读者直接参考[我们的旅程从一段代码开始](https://github.com/guangzhengli/k8s-tutorials?tab=readme-ov-file#container)）。

首先`ng new k8s_angular_demo`, VS Code打开，在根目录新建`Dockerfile`，内容如下：

### v1
```Dockerfile
# Multi-platform images
FROM --platform=$BUILDPLATFORM node:20-alpine as builder

RUN mkdir /project
WORKDIR /project

RUN npm install -g @angular/cli@17

# 构建缓存
COPY package.json package-lock.json ./
RUN npm ci

COPY . .
CMD ["ng", "serve", "--host", "0.0.0.0"]
```

然后

```sh
docker build . -t terry/k8s_angular_demo:v1

docker run --rm --name terry_k8s_angular_demo -p 4200:4200 terry/k8s_angular_demo:v1
```
然后访问`localhost:4200`就可以访问到了。

### v2
更改，使用nginx静态代理。

```dockerfile
# Build Stage
# Multi-platform images
FROM --platform=$BUILDPLATFORM node:20-alpine as builder

RUN mkdir /project
WORKDIR /project

# 构建缓存
COPY package.json package-lock.json ./
RUN npm ci

COPY . .
CMD ["npm", "run", "build"]

# Stage 2
FROM nginx:stable-alpine-perl

COPY --from=builder /project/dist/k8s_angular_demo/browser /usr/share/nginx/html

EXPOSE 80
```

```sh
docker build . -t terry/k8s_angular_demo:v2

docker run --rm --name terry_k8s_angular_demo -p 4200:80 terry/k8s_angular_demo:v2
```
然后再次访问`localhost:4200`，可以得到一样的界面。

可以看到两次构建镜像的大小差距：

```txt
➜  ~ docker images | grep terry/k8s_angular_demo
terry/k8s_angular_demo                          v2        9a7bbdcd00a2   26 minutes ago   89.3MB
terry/k8s_angular_demo                          v1        6fde77d66add   48 minutes ago   770MB
```
## pod
```yaml
# nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx-container
      image: k8s_angular_demo:v2
```
### 找不到镜像问题

遇到一个[Failed to pull image pull access denied , repository does not exist or may require 'docker login'](https://stackoverflow.com/a/68456885/23382462)的问题，最后发现是Minikube有独立的daemon, 为了让Minikube能够找到镜像，需要在执行`eval $(minikube -p minikube docker-env)`的shell中执行构建命令，然后执行`kubectl apply -f deploy/nginx_pod.yaml`。

```sh
eval $(minikube -p minikube docker-env)
docker build . -t terry/k8s_angular_demo:v2
kubectl apply -f deploy/nginx_pod.yaml
```

## Pod 和 Container 区别

![](pod_and_container.png)

container (容器) 的本质是进程，而 pod 是管理这一组进程的资源。pod是k8s资源中的最小单位。

## Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: angular-k8s-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
        # 和下面的 template 中的标签一致，使用选中的pod
      app: angular-k8s
  template:
    metadata:
      labels:
        app: angular-k8s
    spec:
      containers:
        - image: terry/k8s_angular_demo:v2
          name: angular-container-name

```

然后`kubectl apply -f deploy/nginx_deployment.yaml`部署，可以看到结果：![](deployment.png)

你还可以查看minikube提供的Web Console进行可视化管理：

![alt text](<_media/CleanShot 2024-06-22 at 10.02.51.png>)

## 探针

探针分为存活探针,就绪探针和启动探针。

详细教程查看博客的对应章节：
- [存活探针 (livenessProb)](https://github.com/guangzhengli/k8s-tutorials?tab=readme-ov-file#存活探针-livenessprob)
- [就绪探针 (readiness)](https://github.com/guangzhengli/k8s-tutorials?tab=readme-ov-file#就绪探针-readiness)
- [使用启动探针保护慢启动容器](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-startup-probes)

### init container 和 sidecar container

讲到探针，就不得不涉及一下这部分了。`docker-compose`里有一个叫做`depends_on`的功能，可以设置服务启动的顺序，非常好用。但是Kubernetes里不提供这一功能，我查看讨论，大意是这不属于Kubernetes的scope，而是使用者的scope。而我练习自己部署ELK的服务时，最好先等待有状态的Elasticdearch启动完成后再启动Kibana。这一功能可以通过`init container`实现。

对应官网文档地址：
- [Init 容器](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/init-containers/#understanding-init-containers)
- [边车容器](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/sidecar-containers/)

## Service

Service 为 pod 提供了一个稳定的 **Endpoint**（可以理解为Service维护了pod列表，知道如何找到对应的pod） 和 **负载均衡** 功能。

### DNS

[DNS 服务](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#dns)是Service中非常重要的一个功能，可以帮助工程师写出更灵活的部署配置。

> 例如，如果你在 Kubernetes 命名空间 my-ns 中有一个名为 my-service 的 Service， 则控制平面和 DNS 服务共同为 my-service.my-ns 生成 DNS 记录。 名字空间 my-ns 中的 Pod 应该能够通过按名检索 my-service 来找到服务 （my-service.my-ns 也可以）。
> 
> 其他名字空间中的 Pod 必须将名称限定为 my-service.my-ns。 这些名称将解析为分配给 Service 的集群 IP。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-hellok8s-clusterip
spec:
  type: ClusterIP
  selector:
    # 这里声明哪些是被 selector 选中的 Pod
    app: angular-k8s
  ports:
  - port: 4000
    targetPort: 80
```

这样的话其它pod就能通过访问`http://service-hellok8s-clusterip:80`访问到这个服务。因为配置的是ClusterIP，所以只有K8s内部可以访问到。

如果想外部也可以访问到，就需要把`type`设置为`NodePort`。比如：
```yaml
apiVersion: v1
kind: Service
metadata:
  name: service-hellok8s-nodeport
spec:
  type: NodePort
  selector:
    app: hellok8s
  ports:
  - port: 3000
    nodePort: 30000
```

这样就可以通过`k8s cluster node IP + 30000`端口访问到该服务啦。不过使用的情况不是很多，一般情况下我会再写一个`Ingress`或者`Route`对象来暴露给外界。

### 调试 DNS 问题
如果发现找不到其它Service或者类似问题，就需要[调试 DNS 问题](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/dns-debugging-resolution/#check-the-local-dns-configuration-first)。官方文档给出了具体的方法。

题外话，DNS好像也是一个pod，有点一切皆pod的意思了。

## Ingress
Ingress 公开从集群外部到集群内服务的 HTTP 和 HTTPS 路由。 流量路由由 Ingress 资源上定义的规则控制。Ingress 可为 Service 提供外部可访问的 URL、负载均衡流量、 SSL/TLS，以及基于名称的虚拟托管。

Ingress 可以“简单理解”为服务的网关 Gateway，它是所有流量的入口，经过配置的路由规则，将流量重定向到后端的服务。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: angular-k8s-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
        # 和下面的 template 中的标签一致，使用选中的pod
      app: angular-k8s
  template:
    metadata:
      labels:
        app: angular-k8s
    spec:
      containers:
        - image: terry/k8s_angular_demo:v2
          name: angular-container-name

---

apiVersion: v1
kind: Service
metadata:
  name: service-hellok8s-clusterip
spec:
  type: ClusterIP
  selector:
    # 这里声明哪些是被 selector 选中的 Pod
    app: angular-k8s
  ports:
  - port: 4000
    targetPort: 80

---

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-ingress
  annotations:
    # We are defining this annotation to prevent nginx
    # from redirecting requests to `https` for now
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
    - http:
        paths:
          - path: /angular
            pathType: Prefix
            backend:
              service:
                name: service-hellok8s-clusterip
                port:
                  number: 4000
```

## 其它
除此之外，还有`Namespace`，`Configmap`，`Secret`，`Job`，`CronJob`等等其它的资源，这部分比较符合直觉，我也想对比较熟悉，将在接下里的`实践篇-从0开始搭建EAK集群`中进行实践和记录。

### Job
公司内的Jenkins打包，除了build image还有deploy image。deploy image 可以用来部署 build image，我觉得就是通过[Job](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/job/) 以及 [Source-To-Image (S2I)](https://github.com/openshift/source-to-image) 实现的部署完就停止运行，`oc apply`等等命令都在`deployer image`里，不用每次部署都需要我手动输入命令。

## Helm

> 经过前面的教程，想必你已经对 kubernetes 的使用有了一定的理解。但是不知道你是否想过这样一个问题，就是我们前面教程中提到的所有资源，包括用 pod, deployment, service, ingress, configmap,secret 所有资源来部署一套完整的 hellok8s 服务的话，难道需要一个一个的 kubectl apply -f 来创建吗？如果换一个 namespace，或者说换一套 kubernetes 集群部署的话，又要重复性的操作创建的过程吗？
> 
> 我们平常使用操作系统时，需要安装一个应用的话，可以直接使用 apt 或者 brew 来直接安装，而不需要关心这个应用需要哪些依赖，哪些配置。在使用 kubernetes 安装应用服务 hellok8s 时，我们自然也希望能够一个命令就安装完成，而提供这个能力的，就是 CNCF 的毕业项目 Helm。

教程作者的这段话基本可以概括Helm的优点，轻度使用下来和Openshift的template类似。
