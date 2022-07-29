# Kubernate

## Kubernate解决什么问题

### 1. 什么是云计算

PaaS是（Platform as a Service）的缩写，是指[平台即服务](https://baike.baidu.com/item/平台即服务/4329761)。 把[服务器](https://baike.baidu.com/item/服务器/100571)平台作为一种服务提供的商业模式，通过网络进行程序提供的服务称之为SaaS(Software as a Service)，是云计算三种服务模式之一，而云计算时代相应的服务器平台或者开发环境作为服务进行提供就成为了PaaS(Platform as a Service)。

所谓PaaS实际上是指将[软件](https://baike.baidu.com/item/软件)研发的平台作为一种服务，以[SaaS](https://baike.baidu.com/item/SaaS)的模式提交给用户。因此，PaaS也是[SaaS](https://baike.baidu.com/item/SaaS)模式的一种应用。但是，PaaS的出现可以加快SaaS的发展，尤其是加快SaaS应用的开发速度。在2007年国内外SaaS厂商先后推出自己的[PAAS平台](https://baike.baidu.com/item/PAAS平台/123870)。

**云计算，将计算任务分发给无数的小机器，然后进行合并，又称为网格计算**

1. 基础设施即服务（IaaS）
2. 平台即服务(PaaS)
3. 软件即服务(SaaS)

### 2. Docker

在Linux中，实现容器的边界，主要用两种技术，主要实现namespace将容器隔离起来

Linux Namespace

 Linux 的 Namespace 机制。比如，上述将 PID 变成 1 的方法就是通过PID Namespace。在 Linux 中创建线程的方法是 clone, 在其中指定 CLONE_NEWPID 参数，这样新创建的进程，就会看到一个全新的进程空间。而此时这个新的进程，也就变成了 PID=1 的进程。

```
int pid = clone(main_function, stack_size, CLONE_NEWPID | SIGCHLD, NULL); 
```

**容器的限制：Cgroups**

通过namespace技术，实现了容器和容器，容器和宿主之间的隔离，虽然容器相互隔离，但在宿主间的角度看，两个容器就是特殊的进程，进程之间存在着竞争的关系。

Cgroup是Linux为进程设置资源的技术

我们知道，systemd 在 Linux 中的功能就是管理系统的资源。而为了管理的方便，衍生出了一个叫 Unit 的概念，比如一个 unit 可以有比较宽泛的定义，比如可以表示抽象的服务，网络的资源，设备，挂载的文件系统等。为了更好的区分，Linux 将 Unit 的类型主要分为 12 种。

![img](https://pic1.zhimg.com/80/v2-beff4575844a6f8089973d514b1db97c_1440w.jpg)

在Cgroup中，使用的主要是service,slice和scope概念

**容器的文件系统**

容器技术的核心通过namespace 限制了视野，通过CGroup限制了访问的资源

Mount space

chroot改变根目录到指定的目录，在容器内，应该看到完全独立的文件系统，这个文件系统叫做容器镜像，这个独立文件系统，叫做rootfs

在linux中文件和内核是分开存放的，操作系统只有在开启启动时才加载相应的内核，所有的宿主机都共享操作系统的内核

**docker引入了层的概念，每次针对容器的修改，都只是增量修改**

> 想象这个场景，你通过 rootfs 打包了一个包含 java 环境的 centos 镜像，别人需要在容器内跑一个 apache 的服务，那么他是否需要从头开始搭建 java 环境呢？docker 在解决这个问题时，引入了一个叫层的概念，每次针对 rootfs 的修改，都只保存增量的内容，而不是 fork 一个新镜像。

**解决这个问题的方法是每个容器有多个层，每个层关联多个镜像，每个镜像只存储元数据，真正的数据存在layer中**

```text
image/
└── overlay2
    ├── distribution
    ├── imagedb
    │   ├── content
    │   └── metadata
    ├── layerdb
    │   ├── mounts
    │   ├── sha256
    │   └── tmp
    └── repositories.json
```



在 Linux 的主机上，OverlayFS 一般有两个目录，但在显示时具体会显示为一个目录。这两个目录被称为层，联合在一起的过程称为 union mount. 在其下层的目录称为 lowerdir, 上层的目录称为 upperdir. 两者联合后，暴露出来的视图称为 view. 听起来有点抽象，先看下整体结构：

![img](https://pic3.zhimg.com/80/v2-f5bda71e25eab49e3e798a2925137a0e_1440w.jpg)

可以看到，lowerdir 其实对应的就是镜像层，upperdir 对应的就是容器器。而 merged 对应的就是两者联合挂载之后的内容。而且我们发现，当镜像层和容器层拥有相同的文件时，会以容器层的文件为准（最上层的文件为准）。通常来说，overlay2 支持最多 128 lower 层。

### 3. kubernate用处

- **服务发现和负载均衡**

  Kubernetes 可以使用 DNS 名称或自己的 IP 地址来曝露容器。 如果进入容器的流量很大， Kubernetes 可以负载均衡并分配网络流量，从而使部署稳定。

- **存储编排**

  Kubernetes 允许你自动挂载你选择的存储系统，例如本地存储、公共云提供商等。

- **自动部署和回滚**

  你可以使用 Kubernetes 描述已部署容器的所需状态， 它可以以受控的速率将实际状态更改为期望状态。 例如，你可以自动化 Kubernetes 来为你的部署创建新容器， 删除现有容器并将它们的所有资源用于新容器。

- **自动完成装箱计算**

  Kubernetes 允许你指定每个容器所需 CPU 和内存（RAM）。 当容器指定了资源请求时，Kubernetes 可以做出更好的决策来为容器分配资源。

- **自我修复**

  Kubernetes 将重新启动失败的容器、替换容器、杀死不响应用户定义的运行状况检查的容器， 并且在准备好服务之前不将其通告给客户端。

- **密钥与配置管理**

  Kubernetes 允许你存储和管理敏感信息，例如密码、OAuth 令牌和 ssh 密钥。 你可以在不重建容器镜像的情况下部署和更新密钥和应用程序配置，也无需在堆栈配置中暴露密钥。

### 4. MiniKube

```
Minikube是一个快速搭建单节点Kubenetes集群的工具，大家可以把它和docker Machine进行类比。
```

Minikube是一种轻量化的Kubernetes集群，是Kubernetes社区为了帮助开发者和学习者能够更好学习和体验k8s功能而推出的，借助个人PC的虚拟化环境就可以实现Kubernetes的快速构建启动。目前已支持在macOS、Linux、Windows平台上利用各类本地虚拟化环境作为驱动运行。

### Minikube和Kubernates的区别

通常情况下，一套完整的Kubernetes集群至少需要包括`master`节点和`node`节点，下图是常规k8s的集群架构，`master`节点一般是独立的，用于协调调试其它节点之用，而容器实际运行都是在node节点上，`kubectl`位于 `master`节点。

![](https://upload-images.jianshu.io/upload_images/16254840-44f55035f12879c9.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

Minikube master节点与其他节点融为一体，整体通过kubectl进行管理，可以更加节省资源。

![](https://upload-images.jianshu.io/upload_images/16254840-4bec1f9098962451.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

