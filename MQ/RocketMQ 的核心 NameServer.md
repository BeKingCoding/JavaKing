

> 开足码力，码动人生，本文首发公众号【 **程序员大帝** 】，关注这个一言不合就开车的的代码界老司机

> 本文 **GitHub**上已经收录 [https://github.com/BeKingCoding/JavaKing](https://github.com/BeKingCoding/JavaKing) ， 一线大厂面试核心知识点、我的联系方式和技术交流群，欢迎Star和完善

## 前言

本文属于《从零开始消息中间件》的系列文章，接着上篇文章《[不要和陌生人说话，消息中间件之 Topic](https://mp.weixin.qq.com/s/W9L8IqyPwHgr1mlBh4H58A)》，今天来介绍一下 RocketMQ 的核心组件 NameServer。


这个东西很重要，它要管理集群里所有 Broker 的信息，让使用 MQ 的上下游系统可以通过它感知到集群的情况。


开始消息中间件学习的时候，最好有一个切入点，从而搞清楚它的架构设计细节，然后就可以申请一些机器开始落地部署了。



而 NameServer 非常适合我们入手，因为没有 NameServer 一切都无从谈起，可以说这是 RocketMQ 运行的起点。

## 正文

## 01 什么是 NameServer？
NameServer 也称之为路由中心，它的角色主要是为了感知集群里所有的节点与组件，然后配合生产者和消费者，使其能够和 MQ 系统进行通信。



针对目前流行的三种消息中间件 Kafka、 RabbitMQ 和 RocketMQ ，它们对路由中心的实现均有所不同。



Kafka 的路由中心实现相对复杂、混乱，它由 ZooKeeper 以及某个作为 Controller 的 Broker 共同完成的。RabbitMQ 的话是集群每个节点同时也会扮演了路由中心的角色。



而 RocketMQ 是把路由中心抽离出来作为一个独立的 NameServer 角色运行的，因此可以说在路由中心这块，它的架构设计是最清晰明了的。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200810165900175.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70)





## 02 NameServer 如何进行部署？
一个 RocketMQ 集群必须部署 NameServer，那么有两个问题：

（1）NameServer 需要部署几台？是一台机器还是部署多台？



（2）如果部署多台，它们之间是怎么协同工作？



分析这个问题我们可以想一下，NameServer 是整个集群的路由中心，如果没有了它，生产者往哪个 Broker 投递消息都不知道，没有了它，会很麻烦！



为了保证高可用性，NameServer 必然是需要支持多台部署的。如果 NameServer 就部署一台机器的话，一旦它宕机了会导致 RocketMQ 集体出现故障。



所以多机器部署保证了任何一台 NameServer 宕机，其他机器上的 NameServer 可以继续对外提供服务。

## 03 Broker 注册到哪个 NameServer 上去?
假如有 10 台 Broker 机器，两台 NameServer 机器，是不是其中 5 台 Broker 会把自己的信息注册到第一台 NameServer 上去，另外 5 台 Broker 把自己的信息注册到第二台 NameServer 上去？


答案是不对的。如果任何一台 NameServer 宕机了，不就导致 5 个 Broker 的信息就没了。每个 Broker 启动时都得向所有的 NameServer 进行注册。也就是说，每台 NameServer 都会有一份集群中所有 Broker 的信息。

## 04 生产者和消费者如何获取 Broker 信息?
生产者和消费者的系统必须从 NameServer 获取到集群的 Broker 信息然后进行发送与拉取，这个过程有两种办法:

（1）NameServer 每隔一会儿推送 Broker 信息给所有的系统。



（2）生产者和消费者自己每隔一段时间，定时发送请求到 NameServer 去拉取最新的集群 Broker 信息。



相比较而言，肯定是第二种方法更靠谱的。因为如果是 NameServer 主动推送，它怎么知道要推送给哪些系统呢？如果推送失败怎么办呢？

## 05 如果 Broker 宕机了怎么办？
如果生产者和消费者向一台已经挂了的 Broker 发送或者拉取消息必然是徒劳的，那如何保证 Broker 挂了之后，能够迅速的通知到整个系统的各个组件和上下游呢？


要解决这个问题，靠的就是 Broker 和 NameServer 之间的心跳机制。



每隔 30 秒，Broker 就会向集群里所有的 NameServer 发送心跳连接，告诉它们自己的最新状态，NameServer 接收到之后会更新这个 Broker 的最近一次心跳时间。



然后 NameServer 会每隔 10 秒去检查各个 Broker 的最近一次心跳时间，如果某个 Broker 超过 120 秒都没发送心跳了， 那么就认为这个 Broker 已经挂掉了，NameServer 就知道现在集群中的 Broker 已经少了一台。


对于生产者而言，可以考虑不发送消息到那台 Broker，改成发到其他Broker上去。对于消费者而言，每个 Broker 都有 Slave 节点进行备份，可以继续从 Slave 上去拉取信息从而继续使用。



## 总结
今天对于RocketMQ 的核心组件 NameServer 的技术分享就到这里了，总结一下，大家最主要需要知道它的：

（1）集群化部署

（2）Broker 全量注册所有 NameServer 

（3）30 秒心跳机制和 120 秒故障检查机制

（4）生产者和消费者的容错机制

## 文末福利



最近各大互联网公司的秋招都陆陆续续开始了，还在找工作的小伙伴可以后台**回复关键字进入对应的秋招/内推/面试群，我给大家整理了各大公司的内推通道、简历模板还有历年的笔试题**，大家要好好准备哦。还可以帮助大家**免费修改简历、模拟面试哦~**


![在这里插入图片描述](https://img-blog.csdnimg.cn/20200803164240831.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70)




![在这里插入图片描述](https://img-blog.csdnimg.cn/20200803164246398.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70)


**关注公众号「程序员大帝」，《Offer收割机》系列持续更新~**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200803165001620.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70)

创作不易，各位的支持和认可，就是我创作的最大动力，我们下篇文章见！




