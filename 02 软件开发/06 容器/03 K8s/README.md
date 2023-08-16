# k8s

## 架构

`Kubernetes `主要由以下几个核心组件组成：

- `etcd` 保存了整个集群的状态。
- `apiserver ` 提供了资源操作的唯一入口，并提供认证、授权、访问控制、API注册和发现等机制。
- `controller manager` 负责维护集群的状态，比如故障检测、自动扩展、滚动更新等。
- `scheduler` 负责资源的调度，按照预定的调度策略将Pod调度到相应的机器上。
- `kubelet` 负责维护容器的生命周期，同时也负责Volume（CSI）和网络（CNI）的管理。
- `Container runtime` 负责镜像管理以及Pod和容器的真正运行（CRI）。
- `kube-proxy` 负责为Service提供cluster内部的服务发现和负载均衡。

几种主要插件：

- `CoreDNS` 负责为整个集群提供DNS服务。
- `Ingress Controller` 为服务提供外网入口。
- `Prometheus` 提供资源监控。
- `Dashboard` 提供GUI。
- `Federation` 提供跨可用区的集群。

## 设计理念

### API设计原则

1. 所有API应该都是声明式的。
2. API 对象是彼此互补而且可组合的。
3. 高层 API 以操作意图为基础设计。
4. 低层 API 根据高层 API 的控制需要设计。
5. 尽量避免简单封装，不要有在外部 API 无法显式知道的内部隐藏的机制。
6. API 操作复杂度与对象数量成正比。
7. API 对象状态不能依赖于网络连接状态。
8. 尽量避免让操作机制依赖于全局状态，因为在分布式系统中要保证全局状态的同步是非常困难的。

### 控制机制设计原则

1. 控制逻辑应该只依赖于当前状态。
2. 假设任何错误的可能，并做容错处理。
3. 尽量避免复杂状态机，控制逻辑不要依赖无法监控的内部状态。
4. 假设任何操作都可能被任何操作对象拒绝，甚至被错误解析。
5. 每个模块都可以在出错后自动恢复。
6. 每个模块都可以在必要时优雅地降级服务。

## 核心概念

### API对象

API 对象是 `Kubernetes` 集群中的管理操作单元。`Kubernetes` 集群系统每支持一项新功能，引入一项新技术，一定会新引入对应的 API 对象，支持对该功能的管理操作。

三大属性

- 元数据metadata
- 规范spec
- 状态status

### Pod

Pod 是在 `Kubernetes` 集群中运行部署应用或服务的最小单元，它是可以支持多容器的。Pod 的设计理念是支持多个容器在一个 Pod 中共享网络地址和文件系统，可以通过进程间通信和文件共享这种简单高效的方式组合完成服务。Pod 对多容器的支持是 K8 最基础的设计理念。

### 副本控制器（Replication Controller，RC）

RC 是 `Kubernetes` 集群中最早的保证 Pod 高可用的 API 对象。通过监控运行中的 Pod 来保证集群中运行指定数目的 Pod 副本。

### 副本集（Replica Set，RS）

RS 是新一代 RC，提供同样的高可用能力，区别主要在于 RS 后来居上，能支持更多种类的匹配模式。副本集对象一般不单独使用，而是作为 Deployment 的理想状态参数使用。

### 部署（Deployment）

部署表示用户对 `Kubernetes` 集群的一次更新操作。部署是一个比 RS 应用模式更广的 API 对象，可以是创建一个新的服务，更新一个新的服务，也可以是滚动升级一个服务。

### 服务（Service）

在 K8 集群中，客户端需要访问的服务就是 Service 对象。每个 Service 会对应一个集群内部有效的虚拟 IP，集群内部通过虚拟 IP 访问一个服务。在 `Kubernetes` 集群中微服务的负载均衡是由 `Kube-proxy` 实现的。`Kube-proxy` 是 `Kubernetes` 集群内部的负载均衡器。它是一个分布式代理服务器，在 `Kubernetes` 的每个节点上都有一个；这一设计体现了它的伸缩性优势，需要访问服务的节点越多，提供负载均衡能力的 `Kube-proxy` 就越多，高可用节点也随之增多。

### 任务（Job）

Job 是 `Kubernetes` 用来控制批处理型任务的 API 对象。批处理业务与长期伺服业务的主要区别是批处理业务的运行有头有尾，而长期伺服业务在用户不停止的情况下永远运行。

### 后台支撑服务集（DaemonSet）

长期伺服型和批处理型服务的核心在业务应用，可能有些节点运行多个同类业务的 Pod，有些节点上又没有这类 Pod 运行；而后台支撑型服务的核心关注点在 `Kubernetes` 集群中的节点（物理机或虚拟机），要保证每个节点上都有一个此类 Pod 运行。节点可能是所有集群节点也可能是通过 nodeSelector 选定的一些特定节点。典型的后台支撑型服务包括，存储，日志和监控等在每个节点上支持 `Kubernetes` 集群运行的服务。

### 有状态服务集（StatefulSet）

适合于 StatefulSet 的业务包括数据库服务 MySQL 和 PostgreSQL，集群化管理服务 ZooKeeper、etcd 等有状态服务。StatefulSet 的另一种典型应用场景是作为一种比普通容器更稳定可靠的模拟虚拟机的机制。传统的虚拟机正是一种有状态的宠物，运维人员需要不断地维护它，容器刚开始流行时，我们用容器来模拟虚拟机使用，所有状态都保存在容器里，而这已被证明是非常不安全、不可靠的。使用 StatefulSet，Pod 仍然可以通过漂移到不同节点提供高可用，而存储也可以通过外挂的存储来提供高可靠性，StatefulSet 做的只是将确定的 Pod 与确定的存储关联起来保证状态的连续性。

### 存储卷（Volume）

`Kubernetes` 集群中的存储卷跟 `Docker` 的存储卷有些类似，只不过 `Docker` 的存储卷作用范围为一个容器，而 `Kubernetes` 的存储卷的生命周期和作用范围是一个 Pod。每个 Pod 中声明的存储卷由 Pod 中的所有容器共享。

### 持久存储卷（Persistent Volume，PV）和持久存储卷声明（Persistent Volume Claim，PVC）

PV 和 PVC 使得 `Kubernetes` 集群具备了存储的逻辑抽象能力，使得在配置 Pod 的逻辑里可以忽略对实际后台存储技术的配置，而把这项配置的工作交给 PV 的配置者，即集群的管理者。存储的 PV 和 PVC 的这种关系，跟计算的 Node 和 Pod 的关系是非常类似的；PV 和 Node 是资源的提供者，根据集群的基础设施变化而变化，由 `Kubernetes` 集群管理员配置；而 PVC 和 Pod 是资源的使用者，根据业务服务的需求变化而变化，有 `Kubernetes` 集群的使用者即服务的管理员来配置。

### 节点（Node）

`Kubernetes` 集群中的计算能力由 Node 提供，最初 Node 称为服务节点 Minion，后来改名为 Node。`Kubernetes` 集群中的 Node 也就等同于 Mesos 集群中的 Slave 节点，是所有 Pod 运行所在的工作主机，可以是物理机也可以是虚拟机。不论是物理机还是虚拟机，工作主机的统一特征由上面要运行 kubelet 管理节点上运行的容器。

### 命名空间（Namespace）

命名空间为 `Kubernetes` 集群提供虚拟的隔离作用，`Kubernetes` 集群初始有两个命名空间，分别是默认命名空间 default 和系统命名空间 kube-system，除此以外，管理员可以可以创建新的命名空间满足需要。

## 资源对象

在 `Kubernetes` 系统中，`*Kubernetes` 对象是持久化的条目。`Kubernetes` 使用这些条目去表示整个集群的状态。特别地，它们描述了如下信息：

- 什么容器化应用在运行（以及在哪个 Node 上）
- 可以被应用使用的资源
- 关于应用如何表现的策略，比如重启策略、升级策略，以及容错策略

`Kubernetes` 对象是 “目标性记录” —— 一旦创建对象，`Kubernetes` 系统将持续工作以确保对象存在。通过创建对象，可以有效地告知 `Kubernetes` 系统，所需要的集群工作负载看起来是什么样子的，这就是 `Kubernetes` 集群的 **期望状态**。

| 类别     | 名称                                                         |
| :------- | ------------------------------------------------------------ |
| 资源对象 | Pod、ReplicaSet、ReplicationController、Deployment、StatefulSet、DaemonSet、Job、CronJob、HorizontalPodAutoscaling、Node、Namespace、Service、Ingress、Label、CustomResourceDefinition |
| 存储对象 | Volume、PersistentVolume、Secret、ConfigMap                  |
| 策略对象 | SecurityContext、ResourceQuota、LimitRange                   |
| 身份对象 | ServiceAccount、Role、ClusterRole                            |

## 控制器

### RC、RS

`ReplicationController` 用来确保容器应用的副本数始终保持在用户定义的副本数，即如果有容器异常退出，会自动创建新的 Pod 来替代；而如果异常多出来的容器也会自动回收。

在新版本的 `Kubernetes` 中建议使用 `ReplicaSet` 来取代 `ReplicationController`。`ReplicaSet` 跟 `ReplicationController` 没有本质的不同，只是名字不一样，并且 `ReplicaSet` 支持集合式的 selector。

### Deployment

`Deployment`为 Pod 和 ReplicaSet 提供了一个声明式定义（declarative）方法，用来替代以前的 ReplicationController 来方便的管理应用。典型的应用场景包括：

- 定义 Deployment 来创建 Pod 和ReplicaSet
- 滚动升级和回滚应用
- 扩容和缩容
- 暂停和继续 Deployment

### StatefulSet

`StatefulSet`是为了解决有状态服务的问题（对应`Deployments`和`ReplicaSet`是为无状态服务而设计），其应用场景包括：

- 稳定的持久化存储，即Pod重新调度后还是能访问到相同的持久化数据，基于PVC来实现
- 稳定的网络标志，即Pod重新调度后其PodName和HostName不变，基于Headless Service（即没有Cluster IP的Service）来实现
- 有序部署，有序扩展，即Pod是有顺序的，在部署或者扩展的时候要依据定义的顺序依次依次进行（即从 0 到 N-1，在下一个 Pod 运行之前所有之前的 Pod 必须都是 Running 和 Ready 状态），基于 init containers 来实现
- 有序收缩，有序删除（即从 N-1 到 0）

### DaemonSet

`DaemonSet`确保全部（或者一些）Node上运行一个 Pod 的副本。当有Node加入集群时，也会为他们新增一个Pod。当有Node从集群移除时，这些Pod也会被回收。删除`DaemonSet`将会删除它创建的所有Pod。

使用`DaemonSert`的一些典型用法：

- 运行集群存储daemon，例如在每个Node上运行glusterd、ceph。
- 在每个Node上运行日志收集daemon，例如fluentd、logstash。
- 在每个Node上运行监控daemon，例如Promethus Node Exporter、collectd、Datadog代理、New Relic代理，或Ganglia gmond。

### Job

Job负责批处理任务，即仅执行一次的任务，它保证批处理任务的一个或多个Pod成功结束。

### CronJob

Cron Job管理基于时间的Job，即：

- 在给定时间点只运行一次
- 周期性地在给定时间点运行

一个`CronJob`对象类似于crontab文件中的一行。它根据指定的预定计划周期性地运行一个Job。