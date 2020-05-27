## Dubbo

### 架构

#### 服务提供者

​	暴露服务的服务提供方，服务提供者在启动时向注册中心注册自己提供的服务。

#### 服务消费者

​	调用远程服务的服务消费方，服务消费者在启动时，向注册中心订阅自己所需的服务，服务消费者从提供者地址列表中基于软负载均衡算法，选一台提供者进行调用。

#### 注册中心

​	注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。

#### 监控中心

​	服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心

### 通信协议

​	dubbo、hessian、rmi、http、webservice、thrift、memcached、redis。

​	dubbo官方推荐使用dubbo协议。默认端口号20880。

### 序列化协议

​	支持hessian、java二进制序列化、json、SOAP文本序列化多种序列化协议。

​	hessian是默认的序列化协议。

### 通信过程

1. 是基于什么协议实现的？

   Dubbo 协议（通信使用 netty 协议）

2. 怎么发起请求？

   请求内容封装在 RpcInvocation 对象，利用 netty 客户端发请求

3. 怎么将请求转化为符合协议的格式的？

   对象序列化（其实还包含编码及解码过程）

4. 使用什么传输协议传输？

   Netty（本质是 NIO 及 socket）

5. 响应端基于什么机制来接收请求？

   Netty 的响应机制

6. 怎么将流还原为传输格式的？

   反序列化

7. 处理完毕后怎么回应？

   回应内容封装为 RpcResult 对象中，序列化后通过 netty 传给客户端

### 源码分析

#### Dubbo SPI

​		SPI 全称为 `Service Provider Interface`，是一种服务发现机制。SPI 的本质是将接口实现类的全限定名配置在文件中，并由服务加载器读取配置文件，加载实现类。这样可以在运行时，动态为接口替换实现类。正因此特性，我们可以很容易的通过 SPI 机制为我们的程序提供拓展功能。SPI 机制在第三方框架中也有所应用，比如 Dubbo 就是通过 SPI 机制加载所有的组件。不过，Dubbo 并未使用 Java 原生的 SPI 机制，而是对其进行了增强，使其能够更好的满足需求。在 Dubbo 中，SPI 是一个非常重要的模块。基于 SPI，我们可以很容易的对 Dubbo 进行拓展。

##### 使用方法

​	Dubbo SPI 的相关逻辑被封装在了` ExtensionLoader` 类中，通过 `ExtensionLoader`，我们可以加载指定的实现类。

1. 接口上加上@SPI注解

2. Dubbo SPI 所需的配置文件需放置在 META-INF/dubbo 路径下，文件名为接口的全限定名。例如：

   ```java
   optimusPrime = org.apache.spi.OptimusPrime
   bumblebee = org.apache.spi.Bumblebee
   ```

#### Filter

​	结合拓展点功能，使用装饰模式来进行服务的过滤。

```java
private static <T> Invoker<T> buildInvokerChain(final Invoker<T> invoker, String key, String group) {
        Invoker<T> last = invoker;
        List<Filter> filters = ExtensionLoader.getExtensionLoader(Filter.class).getActivateExtension(invoker.getUrl(), key, group);

        if (!filters.isEmpty()) {
            for (int i = filters.size() - 1; i >= 0; i--) {
                final Filter filter = filters.get(i);
                final Invoker<T> next = last;
                last = new Invoker<T>() {

                    @Override
                    public Class<T> getInterface() {
                        return invoker.getInterface();
                    }

                    @Override
                    public URL getUrl() {
                        return invoker.getUrl();
                    }

                    @Override
                    public boolean isAvailable() {
                        return invoker.isAvailable();
                    }

                    @Override
                    public Result invoke(Invocation invocation) throws RpcException {
                        Result asyncResult;
                        try {
                            asyncResult = filter.invoke(next, invocation);
                        } catch (Exception e) {
                            // onError callback
                            if (filter instanceof ListenableFilter) {
                                Filter.Listener listener = ((ListenableFilter) filter).listener();
                                if (listener != null) {
                                    listener.onError(e, invoker, invocation);
                                }
                            }
                            throw e;
                        }
                        return asyncResult;
                    }

                    @Override
                    public void destroy() {
                        invoker.destroy();
                    }

                    @Override
                    public String toString() {
                        return invoker.toString();
                    }
                };
            }
        }

        return new CallbackRegistrationInvoker<>(last, filters);
    }
```

#### 负载均衡

##### RandomLoadBalance

​	加权随机算法

​	算法思想：

​		假设有一组服务器 servers = [A, B, C]，他们对应的权重为 weights = [5, 3, 2]，权重总和为10。现在把这些权重值平铺在一维坐标值上，[0, 5) 区间属于服务器 A，[5, 8) 区间属于服务器 B，[8, 10) 区间属于服务器 C。

##### LeastActiveLoadBalance

​	最小活跃数负载均衡。活跃调用数越小，表明该服务提供者效率越高，单位时间内可处理更多的请求。

​	算法思想：

​		每个服务提供者对应一个活跃数 active。初始情况下，所有服务提供者活跃数均为0。每收到一个请求，活跃数加1，完成请求后则将活跃数减1。在服务运行一段时间后，性能好的服务提供者处理请求的速度更快，因此活跃数下降的也越快，此时这样的服务提供者能够优先获取到新的服务请求、这就是最小活跃数负载均衡算法的基本思想。除了最小活跃数，LeastActiveLoadBalance 在实现上还引入了权重值。某一时刻它们的活跃数相同，此时 Dubbo 会根据它们的权重去分配请求，权重越大，获取到新请求的概率就越大。如果两个服务提供者权重相同，此时随机选择一个即可。

##### ConsistentHashLoadBalance

​	一致性hash算法。

​	算法思想：

​		所有Invoker构成一个圆环，相同颜色的节点均属于同一个服务提供者，比如 Invoker1-1，Invoker1-2，……, Invoker1-160。这样做的目的是通过引入虚拟节点，让 Invoker 在圆环上分散开来，避免数据倾斜问题。所谓数据倾斜是指，由于节点不够分散，导致大量请求落到了同一个节点上，而其他节点只会接收到了少量请求的情况。

##### RoundRobinLoadBalance

​	加权轮训负载均衡算法。

​	

### 直连方式

​	不使用注册中心，服务者和消费者配置中的`registry` = 'N/A'

### 最佳实践

1. 分包
   服务接口和服务模型放在公共包中

2. 粒度

   * 服务接口尽可能大粒度，每个服务方法应代表一个功能，而不是某功能的一个步骤
   * 服务接口应以业务场景为单位划分，并对相近业务作抽象，防止接口数量爆炸
   * 不要使用过于抽象的通用接口

3. 版本

   每个接口都应定义版本号，区分同一接口的不同实现

### 优点

1. 性能高

   序列化：序列化的方案很多：xml、json、二进制流...，其中效率最高的就是二进制流

   网络通信：不同于 ``HTTP`需要进行7步走（三次握手和四次挥手），Dubbo采用`Socket`通信机制，提升了效率，并且可以建立长连接，不用反复连接，直接传输数据。

