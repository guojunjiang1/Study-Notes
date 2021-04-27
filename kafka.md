#Kafka

#特性
* 高吞吐，低延迟；kafka每秒可以处理几十万条消息，它的延迟最低只有几毫秒，每个主题可以分为多个分区，消费组对分区进行消费操作
* 可扩展性：kafka支持热扩展
* 持久性、可靠性：消息被持久化到磁盘，并且支持数据备份防止数据丢失
* 容错性：允许集群中节点失败（若副本数量是n，则允许n-1个节点失败）
* 高并发：允许数千个客户端同时读写

#使用场景
* 日志收集:一个公司可以用kafka收集各种服务的log，通过kafka以统一接口服务的方式开放给各种consumer，例如	Hadoop，Hbase，Solr等
* 消息系统:
* 用户活动跟踪：kafka进程被用来记录用户的各种活动，如浏览页面、搜索、点击等活动，这些活动信息被各个服务器发布到kafka的topic中，然后消费者通过订阅这些topic来做实时的监控分析、或者装载到Hadoop、数据仓库中做离线分析和挖掘
* 运营指标：kafka也经常用来记录运营监控数据。包括收集各种分布式应用的数据，生产各种操作的集中反馈，比如报警和报告

#核心概念
* Producer：消息的生产者，该角色将消息发送到kafka的topic（主题）中。broker接收到生产者发送的消息后，broker将该消息追加到当前用于追加数据的segment文件中。生产者发送的消息，存储到一个partition（分区）中，生产者也可以指定数据存储的partition中
* Consumer：消息的消费者，消费者从broker当中读取数据，消费者可以消费多个topic中的数据
* Topic：主题，kafka可以包括多个topic，如果把kafka看做一个数据库，那么topic可以理解为数据库的一张表，topic名为表名
* Partition：分区，一个topic可以分割为一个或多个Partition，每个Partition的数据用多个segment文件存储，partition中的数据是有序的，但多个partition间的数据丢失了数据的顺序。
> 需要注意，如果topic有多个partition，那么消费消息时就不能保证消息的顺序了，在需要严格保证消息的消费顺序的场景下，需要将partition数目设为1

> 分区只能添加不能删除

* Partition offset：每条消息都有一个partition下唯一的64字节的offset，它指明了这条消息的起始位置
* Replicas of partition：副本，一个分区的副本，副本不会被消费者消费，副本只是用于防止数据丢失
> 即消费者不从为follower的partition中消费数据，而是从为leader的partition中读取数据，副本之间是一主多从的关系

* Broker：kafka集群包含一个或多个服务器，服务器节点称为Broker，broker存储topic数据。

如果某topic有N个partition，集群有N个Broker，那么每个broker存储该topic的一个partition

如果某topic有N个partition，集群有N+M个broker，那么N个broker存储该topic的partition，其他M个broker不存储该topic的partition数据。

如果某topic有N个partition，集群的broker数目少于N个，那么一个broker存储该topic的一个或多个partition，在实际生产环境中，尽量避免这种情况发生，这种情况容易导致kafka集群数据不均衡。

*  Consumer Group消费组
每个 Consumer 属于一个特定的 Consumer Group（可为每个 Consumer 指定Group Name，若不指定 Group Name 则属于默认的 Group）。

> 这是 Kafka 用来实现一个 Topic 消息的广播（发给所有的 Consumer ）和单播（发给任意一个 Consumer ）的手段。

一个Topic可以有多个 Consumer Group。Topic 的消息会复制（不是真的复制，是概念上的）到所有的 Consumer Group，但每个 Consumer Group 只会把消息发给该 Consumer Group 中的一个 Consumer。如果要实现广播，只要每个 Consumer 有一个独立的 Consumer Group 就可以了。如果要实现单播只要所有的 Consumer 在同一个 Consumer Group 。

* Leader：每个partition有多个副本，其中有且只有一个Leader，Leader是当前负责数据的读写的partition
* Follower：每个partition的多个副本中，除了Learder外的其他副本统称为Follower，所有读写请求都通过Leader路由，数据变更会广播给所有Follower，Follower与leader保持数据同步

> 如果Leader失效，控制器就会从Follower中选举出一个新的Leader，当Follower与Leader挂掉、卡主或者同步太慢，leader会把这个Follower从“in sync replicas”（ISR）列表中删除，重新创建一个Follower


* AR（Assigned Replicas）：分区中所有的副本统称为AR
* ISR（In Sync Replicas）：所有与Leader部分保持同步的副本(包括Leader在内)统称为ISR
* OSR(Out-of Sync Replicas)：与Leader副本同步滞后过多的副本（除了ISR的副本，其他剩下的就是OSR副本）
* HW（High Watermark）：高水位，标识了一个特定的offset，消费者只能拉取到这个offset之前的消息
* LEO（Log End Offset）：日志末端位移，记录了该副本底层日志（log）中下一条消息的位移值，如果LEO=10，那么表示该副本保存了10条消息，位移值是【0,9】
* 重平衡：Rebalance：消费组内某个消费者实例挂掉后，其他消费者实例自动重新分配订阅主题分区的过程，Rebalance是kafka消费端实现高可用的重要手段

# kafka的启动
windows下启动：先启动zookeeper，再打开kafka->F:\kafka\kafka_2.12-2.7.0：open cmd here
输入：.\bin\windows\kafka-server-start.bat .\config\server.properties即可启动成功。

zookeeper端口：2181，kafka端口：9092

# **生产者**

# 发送消息的流程
一：生产者创建出ProducerRecord对象，必须指定topic(主题)，value(要发送的消息)，任意指定partition(分区)，key(键)

二：根据value序列化器，将发送的数据封装成字节数组进行发送。

三：如果指定了partition则将消息发送到对应的topic下的partition去，如果没有指定partition那么则根据指定的key来选择一个partition进行发送。

四：发送成功则可以获取到partition的一些信息，以及当前消息在partition中的offset

# 消息的发送类型
## 同步发送
默认调用send方法就是同步发送，返回一个Future对象，根据Future对象可以获取到kafka的一些信息

## 异步发送
调用send方法，传入一个callback对象，重写onCompletion方法，异步获取到kafka的一些信息

## 区别
同步发送，A向kafka发送消息，等待消息发送完成后获取发送的结果，再执行接下来的业务。

异步发送，A向kafka异步的发送消息，继续执行接下来的业务，当kafka接收到消息后将结果异步回调给A。

对于实时性要求强的场景最好用同步，想提高效率则用异步。

# 序列化器（Serializer）
序列化器的作用：将要发送的信息序列化为可在网络中传输的字符串或字节数组。

kafka提供的默认序列化器有：StringSerializer（字符串），IntegerSerializer（整形），BytesSerializer（字节），这些序列化器都实现了Serializer接口，基本能满足大部分场景的使用了。

自定义序列化器：
当发送的消息是我们的自定义对象时，就需要设置一个自定义序列化器，来传输消息。
>自定义序列化器应实现Serializer接口并重写它的三个方法。

自定义反序列化器：用于将传输的字节数组反序列化为自定义对象，并将接收到的消息反序列化为自定义对象。
>自定义反序列化器应实现Deserializer接口并重写它的三个方法。

# 分区器（Partitioner）
本身kafka有自己的分区策略，如果发送消息时没指定分区，那么就会根据key使用kafka默认的分区器或者自定义的分区器来计算出分区，hash(key)%partitionsNum。

默认分区器：DefaultPartitioner 

自定义分区器：需要实现Partitioner接口，重写方法，并在消息发送方的properties中设置自定义分区器

# 拦截器（Interceptor）
Producer拦截器和Consumer拦截器是kafka0.10版本被引入的，主要用于实现clients端的定制化控制逻辑。

Producer生产者拦截器的功能（消息发送到kafka之前，拦截到消息对消息进行业务处理）：

* 按照某个规则过滤掉不符合要求的消息
* 修改消息的内容
* 统计类需求

自定义发送方拦截器：需实现ProducerInterceptor接口

# 其它生产者参数

## acks
这个参数用于指定分区中有多少个副本要收到这条消息，之后生产者才会认为这条消息写入成功。acks是生产者客户端中非常重要的一个参数，它涉及到消息的可靠性和吞吐量之间的权衡。

* ack=0：生产者在成功写入消息之前不会等待任何来自服务器的响应，如果出现问题生产者是感知不到的，消息直接丢失，不过由于他不需要等待服务器响应从而它会达到很高的**吞吐量**。

* ack=1：默认不配置时ack=1，只要副本中的leader副本能接收到这条消息，生产者就会收到一个来自服务器的成功响应。但会出现消息丢失情况。
> eg：发送消息时leader副本宕机，新的leader没有选举出来，此时就会出现消息丢失，不过生产者还会有消息重试机制，但如果leader成功接收到消息并返回给生产者接收成功通知，此时如果leader给follower节点同步消息时leader节点宕机，那么消息就丢失掉了。


* ack=-1：只有当参与到的所有副本都成功接收到消息时，才会向生产者发出一个消息发送成功的消息，所以它是最**安全**的。

ack=0吞吐最大但最不安全，ack=-1最安全但吞吐最小，ack=1介于两者之间

> 注意：acks参数配置是字符串类型，不是整数类型。


## retries
这个参数就是重试了，可以指定消息的重试次数，当发送失败后，会重试重新向kafka进行发送，默认情况下消息每次重试之间等待100ms，可以通过retry.backoff.ms参数来修改这个时间

## batch.size
当有多条消息需要被发送到同一分区时，生产者会把这些消息放到同一批次里，该参数指定了一个批次可以使用的内存大小，按照字节数计算而不是按照消息个数计算。
> 不过生产者也不是必须等批次被填满才会发送，半满的批次甚至只有一条消息的批次也可以直接发送。所以batch.size设的很大也不会造成延迟，只会占用更多内存而已，但是如果设置的太小，生产者会因为频繁的发送消息而造成一些额外的开销

## max.request.size
该参数可以指定单条消息的最大值，也可以指定单个请求里所有消息的总大小。
> broker对可接收的消息最大值也有自己的限制（message.max.size），因此两边的配置最好一样，避免出现生产者发送的消息过大broker接收不了。

#消费者

##消费者和消费组
消费消息时，要指定一个group，就是消费组，消费组下可以有多个消费者。
> 一个消费组内的消费者订阅的是同一个topic
> 多个消费组可以消费同一topic下的消息

## 常见参数设置
* key，vlaue反序列化器
* kafka的集群地址
* 消费组名称
* 消费者客户端名称（自定义名称用于标识消费者）

## 订阅主题
通过consumer.subscribe(Arrays.asList(topic,topic1));可订阅一个或多个主题

也可通过consumer.subscribe(Pattern.compile("heima*"));正则表达式形式来订阅以heima开头的topic主题

## 指定分区
consumer.assign(Arrays.asList(new TopicPartition(topic,0)));可指定一个或多个分区（接收来自指定topic下指定分区的消息）

> 代码指定分区和主题时，两者指定其中一个就可以了，不然会发生冲突
> 如果不指定分区那么默认就是监听topic下的所有分区

## 位移提交
对于kafka中topic内分区而言，它的每条消息都有唯一的offset，用来表示消息在分区中的位置。

当我们调用poll()时，该方法会返回我们没有消费的消息。当消息从broker发送给消费者时，broker并不跟踪这些消息是否被消费者接收到，kafka让消费者自身来管理这些消息的位移，然后消费者通过接口将当前消费的消息的位移更新到kafka，这种更新位移方式成为提交
> 就是消费者消费完消息后手动将当前消息的offset同步到kafka。

位移提交可能造成的现象：

* 重复消费：有两个消费者来消费消息时，有消息AB，如果消费者1消费了A消息后**没来得及位移提交**给kafka，此时消费者2来消费消息又会重复消费一次A消息。
* 消息丢失：消费者拉取下来消息后，还没消费就**已经位移提交**给了kafka，此时消费者宕机了，之后再去拉取消息就拉取不到丢失的消息了。

### 位移提交的几种方式
#### 自动提交
默认情况下采用的就是自动提交方式，消费者在poll方法调用后每隔5s提交一次位移。
>会出现重复消费的情况，因此如果要采用这种方式那么就需要做幂等性校验

#### 同步提交
自己手动提交offset，需要设置一个参数：关闭自动位移提交（ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG），在消费完消息后手动提交位移提交（consumer.commitSync()）。
>同步提交会阻塞当前的应用，当然我们可以减少同步提交的频率，但那样的话也会出现和自动提交一样的问题就是重复消费了。

### 异步提交
自己手动提交offset，需要设置一个参数：关闭自动位移提交（ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG），在消费完消息后手动提交位移提交（consumer.commitAsync(new OffsetCommitCallback())）。
>异步提交不会阻塞当前的应用，但如果提交过程中服务器发生异常，异步提交是不会重试的（异步提交没重试是因为，如果同时存在多个异步提交，A提交的位移是200，A提交失败了就不断进行重试，此时B提交的位移是300然后提交成功了，A不断重试最终也提交成功了，那么位移就从300回滚到了200）

### 总结：
* 默认情况下是自动提交，会出现重复消费情况。
* 同步提交会阻塞当前业务
* 异步提交一旦提交失败不会重试提交
>解决消息丢失，重复消费可以采用再均衡监听器的方式（业务常用的方式就是异步提交消息，然后设置一个再均衡监听器，一旦出现消费者变更，那么再均衡监听器监听到再同步提交，即时向kafka更新消费到消息的位置避免消息丢失及重复消费）

## 再均衡监听器
当某个topic下的分区下的消费者变更后（从消费者A变成消费者B时或者消费者A宕机），一旦发生消费者变更就有可能出现消息丢失以及消息重复消费的情况。

再均衡为消费组提供了高可用性和伸缩性的保障。使得我们既方便又安全地删除消费组内的消费者又或者往消费组内添加消费者。ConsumerRebalanceListener
>不过再均衡期间，消费者是无法拉取消息的。

## 消费回溯
所谓消息回溯就是可以追踪以前消费过的消息，以及指定消费位置的offset开始消费消息

通过seek（）方法指定offset，拉取指定offset后的所有消息

## 消费者拦截器
在消费消息之前，对消息进行拦截进行一些定制化的操作，可以过滤掉不想要的消息

拦截器需实现ConsumerInterceptor接口并重写方法。

然后在消费端添加拦截器的配置

# 其他消费者参数
* fetch.min.bytes：这个参数允许消费者指定从kafka读取消息时最小的数据量。当消费者从kafka读取消息时，如果数据量小于这个阈值，kafka会等待直到有足够的数据，然后才返回给消费者。
* fetch.max.wait.ms：这个参数指定了消费者读取时最长	等待时间，从而避免长时间阻塞，默认是500ms。
* max.partition.fetch.bytes：这个参数指定了每个分区返回的最多字节数，默认是1M。
> 在实际开发中，如果一个topic下有20个分区，有2个消费者，那么意味着这两个消费者，每个消费者需要10M的空间来处理消息，但实际情况下往往这个空间需要更大，这样当某一消费者宕机后，其他消费者可以承担更多的分区
* max.poll.records:这个参数控制poll（）调用后返回的记录数。

# kafkaAdminClient
kafka中添加分区之类的操作，一般是命令行操作的，但我们也可以使用JAVA客户端对其进行操作，因此就有了KafkaAdminClient

# 存储机制
kafka中存储消息是通过文件来存储的

生产方发送消息到kafka下的topic下的某一分区partition当中，每个partition下用三个segment文件来存储消息

## segment文件
segment文件默认大小为1G。

segment文件由两大部分组成，分别是index和data file，后缀为.index的为segment索引文件，后缀为.log的是segment数据文件。

# kafka存储为什么高效
kafka的Message存储采用了分区(partition)，分段（logSegment）和稀疏索引这几个手段来达到高效

# kafka的强大功能
## 生产者幂等性
当生产者向kafka发送消息出现了网络波动后，不知道是否发送成功，此时会进行消息重试，也就意味着可能多条相同的消息会发送到kafka。

为了解决这种问题，kafka有一种生产者幂等性，只需要在生产者端配置一个
pro.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG,"true")即可。
> 只能在单个分区内实现幂等性，跨分区是实现不了的

## 事务
kafka中的事务可以保证对多个分区写入操作的原子性，要么多条消息都写入成功，要么都写入失败。

### 使用方法
消息发送方需指定一个全局唯一的事务Id，并且开启幂等性（幂等性默认就是开启的）

## 数据管道Connect
kafka是一个使用越来越广的消息系统，它可以集成很多系统，Connect的出现的就是为了简化和其他系统的集成并实现各种connector（File,JDBC,HDFS等）。

Connect中一些概念

* 连接器：实现了connect API，决定需要运行多少个任务，按照任务来进行数据复制，从work进程获取任务数量并将其传递下去。
* 任务：负责将数据移入或移除kafka。
* work进程：相当于connect和任务的容器，用于负责管理连接器的配置、启动连接器和连接器任务，提供RESTAPI。
* 转换器：kafka connect和其他存储系统直接发送或者接收数据之间转换数据。
### 实例
将a.txt文件中的内容通过source连接器写入kafka主题中，然后kafka将内容写出到b.txt中（通过kafka做了中介）
> 通过命令行方式实现

* worker进程用到的配置文件：config/connect-standalone.properties
* source连接器用到的配置文件：config/connect-file-source.properties
* sink连接器用到的配置文件：config/connect-file-sink.properties



## 控制器
在kafka集群中有一个或多个broker，其中只有一个broker会被选举为控制器，他负责管理整个集群中所有分区和副本的状态，当某个分区的leader副本出现问题时，就是由控制器来选举新的leader副本
> 控制器选举的工作依赖于zookeeper，成功竞选为控制器的broker会在zookeeper中创建/controller这个临时节点

## 可靠性消息保证
* kafka的保证：在同一分区下的消息是有序的（但无法保证同一主题下的消息有序），只要有一个副本是活跃的那么已提交的消息就不会丢失

## kafka如何保证消息不丢失
* 消息发送方：设置ack=-1，同步发送消息时只有当参与到的所有副本都接收到消息后才返回发送成功的标识。
* kafka：首先对于某一分区下会有多个副本做消息的存储来保证消息不丢失，其次就算出现宕机重新选举leader导致消息丢失的情况，kafka也推出了epoch+offset的机制来保证消息不丢失（epoch记录leader的版本号，offset来记录已消费消息的位移）。
* 消息接收方：每次成功消费完消息后，同步提交位移。

## kafka如何保证消息不重复
* 消息发送方：开启生产者幂等性，设置ack=0，可能会导致丢失消息，但能保证消息不重复，适用于吞吐量指标高于消息不丢失的场景，例如：日志收集。
* 消息接收方：根本原因在于消息消费完没有及时提交offset到broker，每次消费完消息后同步提交位移即可。或者做一下接收方的幂等性校验，对于重复消息直接丢弃即可。



