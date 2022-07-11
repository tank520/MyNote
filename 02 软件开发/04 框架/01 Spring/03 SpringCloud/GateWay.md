# GateWay

## 概念

GateWay是在Spring生态系统之上构建的API网关服务，基于Spring5、SpringBoot 2和Project Reactor等技术。

旨在提供一种简单而有效的方式来对API进行路由，以及提供一些强大的过滤功能，例如：安全、监控/指标、限流等。

GateWay性能好，是因为底层使用了WebFlux，而WebFlux底层使用的是Netty通信。

### 路由

路由是构建网关的基本模块，它是由ID、目标URI、一系列断言和过滤器组成。

### 断言

请求地址匹配

### 过滤器

对请求过滤

## 原理

![](.\图片\gateway_flow.png)

1. 客户端向Gateway发出请求，然后再Gateway Handler Mapping中找到与请求相匹配的路由，将其发送到Gateway Web Handler。
2. Handler再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑。

## 过滤器

### 过滤器类型

#### 全局过滤器

所有请求过滤。

#### 局部过滤器

某个/某些路由特定的过滤器。

### 作用位置

#### PRE

请求之前。

#### POST

请求之后。

