# OpenFeigh

## 概念

Feigh是一个声明式WebService客户端。使用Feigh能让编写WebService客户端更加简单。

使用方法是定义一个服务接口然后在上面添加注解。Feigh也支持可插拔式的编码器和解码器。Spring Cloud对Feigh进行了封装，使其支持Spring MVC标准注解和HttpMessageConverters。Feigh可以与Eureka和Ribbon组合使用以支持负载均衡。

## 特点

旨在使编写Java Http客户端变得更容易。

Ribbon + RestTemplate方式需要自己去封装请求调用。而OpenFeigh只用在接口上添加注解就可完成对服务提供方的接口绑定，简化了使用Ribbon时封装服务调用客户端的开发量。

Feigh里面集成了Ribbon，可以利用Ribbon的负载均衡。

## 日志

Fiegh提供了日志打印功能，我们可以通过配置来调整日志级别，从而了解Feigh中Http请求的细节。

- NONE

  默认的，不显示任何日志。

- BASIC

  仅记录请求方法、URL、响应状态码及执行时间。

- HEADERS

  除了BASIC中定义的信息外，还有请求和响应的头信息。

- FULL

  除了HEADERS中定义的信息外，还有请求和响应的正文及元数据。