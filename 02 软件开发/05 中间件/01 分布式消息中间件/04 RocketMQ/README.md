# RocketMQ

## 架构

### NameServer

1. Broker管理者，每个NS持有所有的Broker信息，并且和NS保持心跳连接。
2. Routing管理者，为Consumer/Provider提供Broker中的queue信息
3. 各个NS之间无连接

### Broker

Broker保证消息的存储、传递和HA等。

​	几个重要的子模块：

1. Remoting Module：处理客户端的请求
2. Client Manager：管理生产者 / 消费者，维护消费者的订阅
3. Store Service：提供API从物理磁盘来存储或查询消息
4. HA Service：主从Broker之间进行数据的同步
5. Index Service：根据特殊的key构建索引，提供快速查询

### Provider

消息生产者，用于提供消息

### Consumer

消息消费者，用于消费消息

## 存储

### Commit Log

消息存放的最终物理文件，被本机Broke的所有Queue所共享。

顺序写、随机读

### Consume Queue

存储路径格式：/${topicName}/${queueId}/${fileName}

格式：

![img](https://upload-images.jianshu.io/upload_images/175724-7212acc81b91c086.png?imageMogr2/auto-orient/strip%7CimageView2/2)

1. CommitLog Offset是指这条消息在Commit Log文件中的实际偏移量
2. Size存储中消息的大小
3. Message Tag HashCode存储消息的Tag的哈希值：主要用于订阅时消息过滤（订阅时如果指定了Tag，会根据HashCode来快速查找到订阅的消息）

