# Eureka

## 架构

![](.\图片\eureka_arc.png)

- Eureka Server

  提供服务注册和发现，多个Eureka Server之间会同步数据，做到状态一致（最终一致性）。采用AP模式。

- Service Provider

  服务提供者，将自身服务注册到Eureka Server，使消费者能够获取。

- Service Consumer

  服务消费者，从Eureka Server获取服务注册列表，进行服务消费。

Eureka分为Server和Client。

- Eureka Server

  各个微服务节点通过配置启动后，会在EurekaServer中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息。

- Eureka Client（Service Provider + Service Consumer）

  Java客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的、使用轮询负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳（默认周期30s）。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，Eureka Server将会从服务注册表中把这个服务节点移除（默认90s）。

## 自我保护机制

假设在某种特定的情况下（如网络故障）, Eureka Client和Eureka Server无法进行通信，此时Eureka Client无法向Eureka Server发起注册和续约请求，Eureka Server中就可能因注册表中的服务实例租约出现大量过期而面临被剔除的危险，然而此时的Eureka Client可能是处于健康状态的（可接受服务访问），如果直接将注册表中大量过期的服务实例租约剔除显然是不合理的，自我保护机制提高了eureka的服务可用性。

### 配置项

```yaml
eureka.server.enable-self-preservation: false
```

## 集群

### 部署方式

eureka Server之间互相注册