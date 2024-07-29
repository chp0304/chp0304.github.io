---
title: Chapter4-消费者
date: 2024-07-29 20:15:12 +0800
categories: [Kafka]
tags: [kafka]     # TAG names should always be lowercase
typora-root-url: ../../chp0304.github.io/assets/pictures
---


应用程序使用 KafkaConsumer 向 Kafka 订阅主题，并从订阅的主题上接收消息。

### Kafka消费者

#### 通过横向伸缩提升消费者的消费能力

Kafka 消费者从属于消费者群组。一个群组里的消费者订阅的是同一个主题，每个消费者接收主题一部分分区的消息。往群组里增加消费者是横向伸缩消费能力的主要方式。Kafka 消费者经常会做一些高延迟的操作，比如把数据写到数据库或 HDFS，或者使用数据进行比较耗时的计算。在这些情况下，单个消费者无法跟上数据生成的速度，所以可以增加更多的消费者，让它们分担负载，每个消费者只处理部分分区的消息，这就是横向伸缩的主要手段。我们有必要为主题创建大量的分区，在负载增长时可以加入更多的消费者。不过要注意，不要让消费者的数量超过主题分区的数量，多余的消费者只会被闲置。第 2 章介绍了如何为主题选择合适的分区数量。

#### 消费者群组和分区再均衡

- 一个新的消费者加入群组时，它读取的是原本由其他消费者读取的消息
- 当一个消费者被关闭或发生崩溃时，它就离开群组，原本由它读取的分区将由群组里的其他消费者来读取
- 主题发生变化时，比如管理员添加了新的分区，会发生分区重分配
- **再均衡**：分区的所有权从一个消费者转移到另一个消费者，这样的行为被称为再均衡。在再均衡期间，消费者无法读取消息，造成整个群组一小段时间的不可用。另外，当分区被重新分配给另一个消费者时，消费者当前的读取状态会丢失，它有可能还需要去刷新缓存，在它重新恢复状态之前会拖慢应用程序。

### 创建Kafka消费者

#### 消费者属性

- 必选属性

  - `bootstrap.servers`：指定broker的address，格式为host：post


  - `key.deserializer`：一个类用于反序列化key


  - `value.deserializer`：一个类用于反序列化value


  - `groun.id`:用于指定KafkaConsumer属于那个消费者群组

- 其他属性

  - ` fetch.min.bytes`:该属性指定了消费者从服务器获取记录的最小字节数。broker 在收到消费者的数据请求时，如果可用的数据量小于 fetch.min.bytes 指定的大小，那么它会等到有足够的可用数据时才把它返回给消费者.
  - `fetch.max.wait.ms`: feth.max.wait.ms 则 用 于 指 定 broker 的 等 待 时 间， 默 认 是 500ms。
  - `max.partition.fetch.bytes`:该属性指定了服务器从每个分区里返回给消费者的最大字节数。它的默认值是 1MB。
  - `session.timeout.ms`:该属性指定了消费者在被认为死亡之前可以与服务器断开连接的时间，默认是 3s。如果消费者没有在 session.timeout.ms 指定的时间内发送心跳给群组协调器，就被认为已经死亡，协调器就会触发再均衡，把它的分区分配给群组里的其他消费者。
  - `heartbeat.interval.ms `:heartbeat.interval.ms 指 定 了 poll() 方 法 向 协 调 器发送心跳的频率，session.timeout.ms 则指定了消费者可以多久不发送心跳。所以，一般需要同时修改这两个属性，heartbeat.interval.ms 必须比 session.timeout.ms 小，一般 是 session.timeout.ms 的 三 分 之 一。 如 果 session.timeout.ms 是 3s， 那 么 heartbeat.interval.ms 应该是 1s。
  - `auto.offset.reset`:该属性指定了消费者在读取一个没有偏移量的分区或者偏移量无效的情况下（因消费者长时间失效，包含偏移量的记录已经过时并被删除）该作何处理。它的默认值是 latest，意思是说，在偏移量无效的情况下，消费者将从最新的记录开始读取数据（在消费者启动之后生成的记录）。另一个值是 earliest，意思是说，在偏移量无效的情况下，消费者将从起始位置读取分区的记录。
  - `enable.auto.commit`:该属性指定了消费者是否自动提交偏移量，默认值是 true。
  - `partition.assignment.strategy`:根据给定的消费者和主题，决定哪些分区应该被分配给哪个消费者。有两个分配策略：`Range`和`RoundRobin`。
  - `client.id`:broker 用它来标识从客户端发送过来的消息。
  - `max.poll.records`:该属性用于控制单次调用 call() 方法能够返回的记录数量，可以帮你控制在轮询里需要处理的数据量。
  - `receive.buffer.bytes 和 send.buffer.bytes`:socket 在读写数据时用到的 TCP 缓冲区也可以设置大小。

#### 读取消息

消费者使用`poll`方法读取消息。每次调用 poll() 方法，它总是返回由生产者写入 Kafka 但还没有被消费者读取过的记录，我们因此可以追踪到哪些记录是被群组里的哪个消费者读取的。

- **提交**：我们把更新分区当前位置的操作叫作提交。
- 提交的几种方式
  - 自动提交：如果 `enable.auto.commit` 被设为 true，那么每过 5s，消费者会自动把从 poll() 方法接收到的最大偏移量提交上去。
  - 同步提交：使用 `commitSync()`提交偏移量最简单也最可靠。
  - 异步提交：使用 `commitASync()`提交偏移量
  - 同步和异步组合提交
  - 提交特定的偏移量：以上几种方法都是提交`poll`方法放回数据的最后一个偏移量。

#### 再均衡监听器

消费者在退出和进行分区再均衡之前，会做一些清理工作。你会在消费者失去对一个分区的所有权之前提交最后一个已处理记录的偏移量。如果消费者准备了一个缓冲区用于处理偶发的事件，那么在失去分区所有权之前，需要处理在缓冲区累积下来的记录。你可能还需要关闭文件句柄、数据库连接等。在 为 消 费 者 分 配 新 分 区 或 移 除 旧 分 区 时， 可 以 通 过 消 费 者 API 执 行 一 些 应 用 程 序 代码， 在 调 用 subscribe() 方 法 时 传 进 去 一 个 `ConsumerRebalanceListener` 实 例 就 可 以 了。



`ConsumerRebalanceListener`需要实现两个方法：

- ` public void onPartitionsRevoked(Collection<TopicPartition> partitions) `方法会在再均衡开始之前和消费者停止读取消息之后被调用。如果在这里提交偏移量，下一个接管分区的消费者就知道该从哪里开始读取了。
- `public void onPartitionsAssigned(Collection<TopicPartition> partitions)` 方法会在重新分配分区之后和消费者开始读取消息之前被调用。

#### 退出读取消息

要记住，consumer.wakeup() 是消费者唯一一个可以从其他线程里安全调用的方法。调用 consumer.wakeup() 可以退出 poll()，并抛出 WakeupException 异常，或者如果调用consumer.wakeup() 时线程没有等待轮询，那么异常将在下一轮调用 poll() 时抛出。
