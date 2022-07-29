# Kubernate基础概念学习

* Master节点：整个集群的中枢

1. Kube API Server: 集群的控制中枢，各个模块之间的交互都需要API Server,同时它也是集群管理，资源配置，整个集群的安全入口
2. Controller-Manager: 集群的状态管理器，保证pod和其他资源达到期望值，也需要和API Server进行通信，在需要的时候创建，更新或删除它所管理的资源
3. Scheduler: 集群的调度中心
4. Etcd: 键值数据库

* Node节点，负责维护每个Pod的状态

1. Kublet: 负责监听节点上Pod的状态，同时负责上报节点和节点Pod的状态，负责与Master节点通信，管理节点上的Pod
2. Kube Proxy: 负责Pod之间的通信和负载均衡

> - **Master 组件**
> - API Server。Kubernetes 如何接收请求，又是如何将结果返回至客户端。
> - [Etcd](https://etcd.io/docs)。了解 Etcd 主要功能机制。
> - Controller Manager。Kubernetes 控制器是其架构中最为核心的一环，我们需要了解控制器的原理，List-Watch 的基本原理，知道 Kubernetes 默认情况下大致包含哪些类型的控制器。
> - [Scheduler](https://kubernetes.io/docs/concepts/scheduling/kube-scheduler/)。熟悉 Kubernetes 的调度流程是怎样的，调度器在整个调度流程中的角色。
> - **Node 组件**
> - Kubelet。知道 Kubelet 是如何接受调度请求并启动容器的。
> - Kube-proxy。了解 Kube-proxy 的作用，提供的能力是什么。
> - Container Runtime。了解都有哪些 Container Runtime，主要了解 [Docker](https://docs.docker.com/) 一些基本操作与实现原理。

## Api Server

**Kubernetes API Server的核心功能是提供Kubernetes各类资源对象(如Pod、RC、Server等)的增、删、改、查及watch等HTTP REST接口，成为集群内各个功能模块之间数据交互和通信中心枢纽，是整个系统的数据总线和数据中心。除此之外，它还是集群管理的API入口，是资源配额控制的入口，提供了完备的集群安全机制。**

### Kubernates API Server核心组件

Kubernate API Server A通过kube-apiserver提供服务，该进程运行在Master上，在默认情况下，kube-apiserver进程在本机的8080端口，可以通过同时启动HTTPs端口来启动

```

curl localhost:8080/api/v1 #查看Kubernete API Server目前支持的资源对象种类
 
curl localhost:8080/api/v1/pods #查看集群中pod的列表
 
curl localhost:8080/api/v1/services #查看集群中Service列表
 
curl localhost:8080/api/v1/replicationcontrollers #查看RC列表
```

### 通过编程调用Kubernates API Server

运行在Pod用户进程调用Kubernates API Server,通常用分布式集群达到目标。Pod在启动过程中访问EndPoints API,找到副本构建集群

![](https://img-blog.csdnimg.cn/f3cee9608de94453910e50b794fb5b64.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5bGM5Lid5Lit5oiY5paX5py6,size_20,color_FFFFFF,t_70,g_se,x_16)

* API层：Rest方式提供API接口，有Kubernates资源对象的CURD
* 访问控制层：当客户端访问API接口时，访问控制层对用户鉴权，验明用户身份，核准Kubernates资源的访问权限，根据配置的各种访问资源许可逻辑，判断是否允许访问。
* 注册表层：Kubernates把所有资源对象都保存在注册表中，针对注册表定义了各种注册类型
* etcd数据库：用于持久化存储Kubernates资源的KV数据库



## etcd数据库

etcd是近几年比较火热的开源的，分布式的键值对数据存储系统，提供共享配置，服务的注册发现

### Raft算法

Raft算法属于Multi-Paxos算法，它是在Multi-Paxos的基础上，做了一些简化和限制，增加了日志必须是连续的，只支持领导者，跟随者和候选人三种状态

**Raft算法是通过一切以领导者为准，实现一系列的共识和节点一致**

1. 领导者选举

Raft算法支持：领导者，候选人和跟随者三种状态：

* 跟随者：接收和处理来自领导者的消息，当领导者心跳超时的时候，就主动站出来，自己当候选人
* 候选人：候选人向其他节点发送请求的RPC消息，通知其他节点投票，如果赢得了选票，就成为领导者
* 领导者：负责处理写请求，管理日志复制和不断的发送心跳信息，通知其他节点

选举领导者状态

1. 初始条件下，大家都是候选人

![](https://img-blog.csdnimg.cn/20210530200941313.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMzc4MDM0,size_16,color_FFFFFF,t_70#pic_center)

这个时间，每个节点设置随机超时时间，集群中没有领导者，节点A等待时间最少，发生超时

节点A增加自己的任期编号，推举自己为候选人，向其他节点发送RPC消息，请求投票

![](https://img-blog.csdnimg.cn/20210530200950353.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMzc4MDM0,size_16,color_FFFFFF,t_70#pic_center)

如果其他节点接收到候选人A的请求投票RPC消息，在编号为1的这届任期内，也还没有进行过投票，那么它将把选票投给节点A，并增加自己的任期编号

![](https://img-blog.csdnimg.cn/20210530201003244.jpeg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMzc4MDM0,size_16,color_FFFFFF,t_70#pic_center)

如果候选人赢得了选票，则会变成领导者

节点A当选领导者后，它将周期性地发送心跳消息，通知其他服务器我是领导者，阻止跟随者发起新的选举

Raft算法中每个任期由单调递增的数字（任期编号）标识，任期编号是随着选举的举行而变化的

1. 跟随者在等待领导者心跳信息超时后，推举自己为候选人时，会增加自己的任期编号，比如节点A的任期编号为0，那么在推举自己为候选人时，会将自己的任期编号增加为1
2. 如果一个服务器节点，发现自己的任期编号比其他节点小，那么它会更新自己的任期编号到较大的编号值，比如节点B的任期编号是0，当收到来自节点A的请求投票RPC消息时，因为消息中包含了节点A的任期编号，且编号为1，那么节点B将把自己的任期编号更新为1

3. 如果一个候选人或者领导者，发现自己的任期编号比其他节点小，那么它会立即恢复成跟随者状态。比如分区错误恢复后，任期编号为3的领导者节点B，收到来自新领导者的包含任期编号为4的心跳消息，那么节点B将立即恢复成跟随者状态
4. 如果一个节点接收到一个包含较小的任期编号值的请求，那么它会直接拒绝这个请求。比如节点C的任期编号为4，收到包含任期编号为3的请求投票RPC消息，那么它将拒绝这个消息

选票规则是：**先来先服务**

etcd具有以下特点：

- 完全复制：集群中的每个节点都可以使用完整的存档
- 高可用性：Etcd可用于避免硬件的单点故障或网络问题
- 一致性：每次读取都会返回跨多主机的最新写入
- 简单：包括一个定义良好、面向用户的API（gRPC）
- 安全：实现了带有可选的客户端证书身份验证的自动化TLS
- 快速：每秒10000次写入的基准速度
- 可靠：使用Raft算法实现了强一致、高可用的服务存储目录

![](https://img2020.cnblogs.com/blog/1739642/202112/1739642-20211211165008860-1978494429.png)

​	**配置中心**：

将一些配置信息放在etcd上集中管理

这类场景使用方式是这样：应用在启动etcd获取一次配置信息，在etcd注册一个Watcher并等待，每次配置有更新的时候，etcd都会实时通知订阅者。

**分布式锁**

因为etcd使用Raft实现了强一致性，操作存储到集群中的值必然是全局一致的，所以很容易实现分布式锁，锁服务有两种方式，一种是独占，一种是控制时序

- **保持独占即所有获取锁的用户最终只有一个可以得到**。etcd 为此提供了一套实现分布式锁原子操作 CAS（`CompareAndSwap`）的 API。通过设置`prevExist`值，可以保证在多个节点同时去创建某个目录时，只有一个成功。而创建成功的用户就可以认为是获得了锁。
- 控制时序，即所有想要获得锁的用户都会被安排执行，但是**获得锁的顺序也是全局唯一的，同时决定了执行顺序**。etcd 为此也提供了一套 API（自动创建有序键），对一个目录建值时指定为`POST`动作，这样 etcd 会自动在目录下生成一个当前最大的值为键，存储这个新的值（客户端编号）。同时还可以使用 API 按顺序列出所有当前目录下的键值。此时这些键的值就是客户端的时序，而这些键中存储的值可以是代表客户端的编号。

## 什么用 etcd 而不用ZooKeeper？

etcd 实现的这些功能，ZooKeeper都能实现。那么为什么要用 etcd 而非直接使用ZooKeeper呢？

### 为什么不选择ZooKeeper？

1. 部署维护复杂，其使用的`Paxos`强一致性算法复杂难懂。官方只提供了`Java`和`C`两种语言的接口。
2. 使用`Java`编写引入大量的依赖。运维人员维护起来比较麻烦。
3. 最近几年发展缓慢，不如`etcd`和`consul`等后起之秀。

### 为什么选择etcd？

1. 简单。使用 Go 语言编写部署简单；支持HTTP/JSON API,使用简单；使用 Raft 算法保证强一致性让用户易于理解。
2. etcd 默认数据一更新就进行持久化。
3. etcd 支持 SSL 客户端安全认证。

最后，etcd 作为一个年轻的项目，正在高速迭代和开发中，这既是一个优点，也是一个缺点。优点是它的未来具有无限的可能性，缺点是无法得到大项目长时间使用的检验。然而，目前 `CoreOS`、`Kubernetes`和`CloudFoundry`等知名项目均在生产环境中使用了`etcd`，所以总的来说，etcd值得你去尝试。

## Controller Manager

Controller Manager由kube-controller-manager和cloud-controller-manager组成，是Kubernates的大脑，他通过api-server监控集群的状态，并确保集群处于工作状态：

![](https://pic2.zhimg.com/80/v2-f6810d088d3c95d7359511f04de3a3d5_1440w.jpg)

- Certificate Controller
- ClusterRoleAggregation Controller
- Node Controller
- CronJob Controller
- Daemon Controller
- Deployment Controller
- StatefulSet Controller
- Endpoint Controller
- Endpointslice Controller
- Garbage Collector
- Namespace Controller
- Job Controller
- Pod AutoScaler
- PodGC Controller
- ReplicaSet Controller
- Service Controller
- ServiceAccount Controller
- Volume Controller
- Resource quota Controller
- Disruption Controller

在Kubernates中，控制器通过监控集群的公共状态，并致力于将当前状态转变为期望的状态：

一个控制器追踪一个Kubernates资源，这些对象有一个spec字段，该资源的控制器负责确保资源接近期望状态：

```
for {
	desired := getDesiredState(),
	current := getCurrentState()
	makeChanges(desired, current)
}
```

控制器有两个组件：informer/SharedInformer和Workqueue

![](https://pic4.zhimg.com/80/v2-9f3f0edea1dd5e1413a86aee3deb82bf_1440w.jpg)

Reflector 和 APIServer 建立长连接，并使用 ListAndWatch 方法获取并监听某一个资源的变化。List 方法将会获取某个资源的所有实例(如ReplicaSet、Deployment等)，Watch 方法则监听资源对象的创建、更新以及删除事件，获取到的事件称之为一个增量(Delta)，该增量会被放进一个称之为 Delta FIFO Queue，即增量先进先出队列中。

Informer会不断地从Delta FIFO Queue pop增量事件，根据新增的类型决定更改事件的缓存。

Informer的另一个职责是出发好事先注册的Event Handler,在回调函数中做一些简单的处理

![](https://pic4.zhimg.com/v2-41c95a02634f8b35b784487416b7479b_r.jpg)

## kubelet

一旦Pod被调用到对应宿主机后，后续要做的是管理这个pod，管理这个pod的生命周期，**包括pod的增删改查的操作**

对于一个Pod来说：里面会存有多个容器，每个容器可以关联不同的镜像，进而运行不同的程序：

1. 感知Pod被创建的命令，并知道这个Pod被创建需要哪些信息
2. kubelet在获取到这部分数据，会根据资源信息完成分配

![](https://img-blog.csdnimg.cn/img_convert/d59ebf079349f8718d55637aee7c84a2.png)

kubelet更新的核心，就是一个循环控制的syncLoop

> 1. Pod更新事件
> 2. Pod生命周期变化
> 3. kubelet本身设置的执行周期
> 4. 定期的清理事件

## Kubernates Scheduler

Scheduler是Kubernates的调度器，属于核心组件，主要任务十八定义的Pod分配到集群的节点上

* 公平：每个节点被分配资源
* 资源高效利用：集群资源最大化使用
* 效率：调度的性能更好
* 灵活：允许用户根据自己的需求调整控制调度的逻辑

![](https://img-blog.csdnimg.cn/img_convert/ccc6950c79e9271c48a9d860c0b22dc7.png)

根据预选规则和优选规则打分后，kube-scheduler会将pod调度到得分最高的Node上，如果存在多个得分最高的Pod* 

* 预选：K8s会遍历所有集群的Node,筛选出其中符合条件的Pod作为候选
* 优选：k8s对Node进行打分

## Kube-Proxy

ubernetes以Pod作为应用部署的最小单位。kubernetes会根据Pod的声明对其进行调度，包括创建、销毁、迁移、水平伸缩等，因此Pod IP地址不是固定的，不方便直接采用Pod IP对服务进行访问。

为解决该问题，Kubernetes提供了Service资源，Service对提供同一个服务的多个Pod进行聚合。一个Service提供一个虚拟的Cluster IP，后端对应一个或者多个提供服务的Pod。在集群中访问该Service时，采用Cluster IP即可，Kube-proxy负责将发送到Cluster IP的请求转发到后端的Pod上。

kube-proxy是一个运行在每个节点的go应用程序

为了避免增加内核和用户空间的数据拷贝操作，提高转发效率，Kube-proxy提供了iptables模式。在该模式下，Kube-proxy为service后端的每个Pod创建对应的iptables规则，直接将发向Cluster IP的请求重定向到一个Pod IP。该模式下Kube-proxy不承担四层代理的角色，只负责创建iptables规则。该模式的优点是较userspace模式效率更高，但不能提供灵活的LB策略，当后端Pod不可用时也无法进行重试。