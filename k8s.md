# Kubernates

# 三大概念
iaas：硬件即服务（阿里云），提供一个硬件平台供用户使用

paas：平台即服务（docker），提供一个硬件平台并提供一些开发环境

saas：软件即服务（saas-ihrm），提供一个软件供用户使用，用户无需安装该软件，在网站直接访问即可

# k8s的特点
* 轻量级：消耗资源小
* 开源
* 弹性伸缩，想要扩展很容易
* 内部实现了负载均衡

# k8s的组成
全称kubernetes，由谷歌开源，它是基于容器的集群管理平台，k8s可以很方便的对docker进行管理

一个k8s系统，通常称为一个k8s集群。k8s高可用集群中副本数量最好是>=3的奇数个数

集群一般分为两块内容：一个master（主节点），一群Node（计算节点）

* master节点主要负责管理和控制
* node节点是工作负载节点，里面是具体的容器

## master
master节点主要包括：Api Server、Scheduler、Controller manager、etcd

* API Server：是整个系统的对外接口，供客户端和其他组件调用，相当于“营业厅”
* Scheduler：负责对集群内部的资源进行调度，相当于“调度室”
* Controller manager：负责管理控制器，相当于“大总管”
* etcd：一个可信赖的分布式键值存储系统，负责k8s中各种数据的存储

## node
node节点包括：pod、Docker、Kubelet、Kube-proxy、Fluentd

* pod：是Kubernetes最基本的操作单元，一个Pod代表着集群中运行的一个进程，它内部封装了一个或多个紧密相关的容器。（一个node可包括1个或多个pod）
* Docker：创建容器的
* Kubelet：负责监视它所在Node上的Pod，包括创建、修改、监控、删除
* Kube-proxy：主要负责为Pod对象提供代理，为Pod进行负载均衡
* Fluentd：负责日志收集、存储与查询

## 其他组件
* Flannel：由CoreOS团队针对Kubernates设计的一个网络规划服务，它可以让不同机器节点上的pod进行通信

### pod
一个pod中包括多个容器，同一pod内的多个容器共享同一网络(pause)和存储。

pod分为静态pod和普通pod

* 静态pod，它并不存放在etcd存储里，而是存放在某个具体的Node的一个文件中，并且只在这个Node上运行。
* 普通pod，创建之后就会被存储在etcd中，随后被Master调度到某个Node上进行绑定，被Node上的Kubelet进程实例化成一组相关的Docker容器并启动，默认情况下，Pod某个容器停止时，k8s会自动检测到并重启此pod，如果pod所在的Node宕机，k8s会将该Node上的pod重新分配给其他Node。

# jenkins
Jenkins相当于一个自动化的，提供友好操作界面的持续集成(CI)工具，主要用于持续、自动的构建/测试软件项目、监控外部任务的运行。Jenkins用Java语言编写，可在Tomcat等流行的servlet容器中运行，也可独立运行。通常与版本管理工具、构建工具结合使用。常用的版本控制工具有SVN、GIT，构建工具有Maven、Ant、Gradle。

## CI/CD
* 持续集成（CI）：将代码提交、合并，构建，部署，测试都在一起，不断地执行这个过程，并对结果进行反馈。
* 持续交付（CD）：任意部署到测试环境，预生产环境，生产环境。
* 持续部署（CD）：将最终产品部署到生产环境，给用户使用。

## 项目使用了Jenkins+docker后的工作流程
* 程序员提交代码到gitlab
* jenkins新建一个任务，指定git地址，maven命令（清除，编译，打包），以及shell脚本（用于将代码封装成镜像并推送到HARBOR仓库）
* 任务中再执行一个脚本，这个脚本用于生产环境服务器自动到HARBOR仓库拉取镜像并运行成容器，对外提供访问
* jenkins自动到gitlab上拉取代码
* jenkins自动调用maven进行编译，打包，并将打包后的文件封装成docker镜像（通过dockerfile），将docker镜像提交到HARBOR镜像仓库（通过build.sh）
* docker拉取镜像，将其变为容器并运行

## 项目使用了Jenkins+k8s后的工作流程
* 程序员提交代码到gitlab
* jenkins新建一个任务，指定git地址，maven命令（清除，编译，打包），以及shell脚本（用于将代码封装成镜像并推送到HARBOR仓库）
* 任务中再执行一个脚本，这个脚本用于生产环境服务器自动到HARBOR仓库拉取镜像并运行成Pod，对外提供访问
* jenkins自动到gitlab上拉取代码
* jenkins调用maven进行打包，并将打包后的文件封装成docker镜像，将docker镜像提交到HARBOR镜像仓库
* k8s测试环境拉取镜像封装成pod进行测试
* 测试通过，k8s生产环境拉取镜像封装成pod，然后发布Service暴露pod供外部访问

（Jenkins也是部署在k8s当中的）
## pipelline
pipellline是jenkins中的一个组件，使用pipellline构建项目比传统构建项目要多很多好处，例如：项目发布可视化、明确阶段、方便处理问题，一个jenkinsFile文件管理整个项目生命周期，jenkindsFile可以放到项目代码中版本管理。
