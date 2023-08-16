# Service

​	`Service` 在 `Kubernetes` 中是一个[对象](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/#kubernetes-objects) （与 Pod 或 ConfigMap 类似的对象）。`Service`定义了这样一种抽象：Pod的逻辑分组，一种可以访问它们的策略--通常称为微服务。这一组Pod能够被Service访问到，通常是通过Label Selector实现的。

## VIP和Service代理

在`Kubernetes`集群中，每个Node运行一个 kube-proxy 进程。kube-proxy负责为Service实现了一种VIP（虚拟IP）的形式，而不是 ExternalName 的形式。

在`Kubenetes` v1.0版本，代理完全在userspace，Service是4蹭概念。

在`Kubenetes` v1.1版本，新增了iptables代理，但并不是默认的运行模式。新增了Ingress API，用来表示7层代理。

从 `Kubernetes` v1.2 起，默认就是 iptables 代理。

在 `Kubernetes` v1.8.0-beta.0 中，添加了 ipvs 代理。

### userspace代理模式

这种模式，kube-proxy 会监视 Kubernetes master 对 `Service` 对象和 `Endpoints` 对象的添加和移除。 对每个 `Service`，它会在本地 Node 上打开一个端口（随机选择）。任何连接到“代理端口”的请求，都会被代理到 `Service` 中的 某个backend `Pods` 上面。

![userspace 代理模式下 Service 概览图](https://lib.jimmysong.io/kubernetes-handbook/images/services-userspace-overview.jpg)

### iptables代理模式

这种模式，kube-proxy 会监视 Kubernetes master 对 `Service` 对象和 `Endpoints` 对象的添加和移除。 对每个 `Service`，它会安装 iptables 规则，从而捕获到达该 `Service` 的 `clusterIP`（虚拟 IP）和端口的请求，进而将请求重定向到 `Service` 的一组 backend 中的某个上面。对于每个 `Endpoints` 对象，它也会安装 iptables 规则，这个规则会选择一个 backend `Pod`。

![iptables 代理模式下 Service 概览图](https://lib.jimmysong.io/kubernetes-handbook/images/services-iptables-overview.jpg)

### ipvs代理模式

这种模式，kube-proxy 会监视 Kubernetes `Service`对象和`Endpoints`，调用`netlink`接口以相应地创建 ipvs 规则并定期与 Kubernetes `Service`对象和`Endpoints`对象同步 ipvs 规则，以确保 ipvs 状态与期望一致。访问服务时，流量将被重定向到其中一个后端 Pod。

ipvs 为负载均衡算法提供了更多选项，例如：

- `rr`：轮询调度
- `lc`：最小连接数
- `dh`：目标哈希
- `sh`：源哈希
- `sed`：最短期望延迟
- `nq`： 不排队调度

![ipvs 代理模式下 Service 概览图](https://lib.jimmysong.io/kubernetes-handbook/images/service-ipvs-overview.png)

## 发布服务（服务类型）

对一些应用（如 Frontend）的某些部分，可能希望通过外部（Kubernetes 集群外部）IP 地址暴露 Service。

Kubernetes `ServiceTypes` 允许指定一个需要的类型的 Service，默认是 `ClusterIP` 类型。

`Type` 的取值以及行为如下：

- `ClusterIP`：通过集群的内部 IP 暴露服务，选择该值，服务只能够在集群内部可以访问，这也是默认的 `ServiceType`。你可以使用 [Ingress](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress/) 或者 [Gateway API](https://gateway-api.sigs.k8s.io/) 向公众暴露服务。
- `NodePort`：通过每个 Node 上的 IP 和静态端口（`NodePort`）暴露服务。`NodePort` 服务会路由到 `ClusterIP` 服务，这个 `ClusterIP` 服务会自动创建。通过请求 `<NodeIP>:<NodePort>`，可以从集群的外部访问一个 `NodePort` 服务。
- `LoadBalancer`：使用云提供商的负载均衡器，可以向外部暴露服务。外部的负载均衡器可以路由到 `NodePort` 服务和 `ClusterIP` 服务。
- `ExternalName`：通过返回 `CNAME` 和它的值，可以将服务映射到 `externalName` 字段的内容（例如， `foo.bar.example.com`）。 没有任何类型代理被创建，这只有 Kubernetes 1.7 或更高版本的 `kube-dns` 才支持。

### ClusterIP

此默认服务类型从你的集群中有意预留的 IP 地址池中分配一个 IP 地址。

其他几种服务类型在 `ClusterIP` 类型的基础上进行构建。

如果你定义的服务将 `.spec.clusterIP` 设置为 `"None"`，则 Kubernetes 不会分配 IP 地址。

### NodePort

如果你将 `type` 字段设置为 `NodePort`，则 Kubernetes 控制平面将在 `--service-node-port-range` 标志指定的范围内分配端口（默认值：30000-32767）。 每个节点将那个端口（每个节点上的相同端口号）代理到你的服务中。 你的服务在其 `.spec.ports[*].nodePort` 字段中报告已分配的端口。

使用 NodePort 可以让你自由设置自己的负载均衡解决方案， 配置 Kubernetes 不完全支持的环境， 甚至直接暴露一个或多个节点的 IP 地址。

对于 NodePort 服务，Kubernetes 额外分配一个端口（TCP、UDP 或 SCTP 以匹配服务的协议）。 集群中的每个节点都将自己配置为监听分配的端口并将流量转发到与该服务关联的某个就绪端点。 通过使用适当的协议（例如 TCP）和适当的端口（分配给该服务）连接到所有节点， 你将能够从集群外部使用 `type: NodePort` 服务。

### LoadBalancer

在使用支持外部负载均衡器的云提供商的服务时，设置 `type` 的值为 `"LoadBalancer"`， 将为 Service 提供负载均衡器。 负载均衡器是异步创建的，关于被提供的负载均衡器的信息将会通过 Service 的 `status.loadBalancer` 字段发布出去。

### ExternalName

类型为 ExternalName 的服务将服务映射到 DNS 名称，而不是典型的选择算符，例如 `my-service` 或者 `cassandra`。 你可以使用 `spec.externalName` 参数指定这些服务。

