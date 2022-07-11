# Ribbon

## 概念

Spring Cloud Ribbon是基于Netflix Ribbon实现的一套客户端负载均衡工具。

Ribbon是Netflix发布的开源项目，主要功能是提供客户端的软件负载均衡算法和服务调用。Ribbon客户端组件提供一系列完善的配置项如连接超时、重试等。简单的说，就是在配置文件中列出Load Balancer（简称LB）后面所有的机器，Ribbon会自动的帮助你基于某种规则（如轮询，随机等）去连接这些机器。

Ribbon = 负载均衡 + RestTemplate。

## 负载均衡

### 进程内LB（本地负载均衡）

将LB逻辑集成到消费方，消费方从服务注册中心获取服务列表，然后根据负载均衡算法选出合适的一个服务进行调用。

Ribbon属于进程内LB，它是一个类库，集成于消费方进程。

### 集中式LB（服务端负载均衡）

在服务的消费方和提供方之间使用独立的LB设施（可以是硬件，如F5，也可以是软件，如Nginx），由该设施负责把访问请求通过某种策略转发至服务器的提供方。

## 负载均衡算法

### RoundRobinRule

简单轮询

### RandomRule

随机

### RetryRule

重试策略。先按照轮询策略获取，如果服务失败则在指定时间内进行重试。

### WeightedResponseTimeRule

响应时间加权策略。对轮询的扩展，响应速度越快的实例选择权重越大，越容易被选择。

### BestAvailableRule

最低并发策略。会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务。

### AvailabilityFilteringRule

可用过滤策略。过滤掉故障实例，选择并发量较小的实例。

### ZoneAvoidanceRule

区域权衡策略。默认规则，复合判断server所在区域的性能和server的可用性选择服务器。

