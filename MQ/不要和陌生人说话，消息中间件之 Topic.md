

> 开足码力，码动人生，本文首发公众号【 **程序员大帝** 】，关注这个一言不合就开车的的代码界老司机

> 本文 **GitHub**上已经收录 [https://github.com/BeKingCoding/JavaKing](https://github.com/BeKingCoding/JavaKing) ， 一线大厂面试核心知识点、我的联系方式和技术交流群，欢迎Star和完善

## 前言
通过之前的两篇文章，向大家介绍了消息中间件 MQ ，大家可以看：

[《十分钟入门消息中间件》](https://mp.weixin.qq.com/s/vemmwC5EMcK9SrgoIJmJzA)

[《还在纠结秒杀？看看 MQ 如何搞定》](https://mp.weixin.qq.com/s/vemmwC5EMcK9SrgoIJmJzA)

[《消息中间件的正确打开方式》](https://mp.weixin.qq.com/s/rHHyNcLcoOIh9vNhN-d-Yw)

本篇文章以目前比较流行的 RocketMQ 为例，讲解一下相关的技术，帮助大家更好地理解消息中间件。


## 正文
## 01 什么是Topic？
Topic 中文含义大家肯定不陌生，直接翻译过来是话题。而在 MQ 里，无论是 RocketMQ 还是 Kafka，都用 Topic 这个名词来代表一种数据的集合。


比如说，现在需要往 MQ 中发送订单的消息，那么我们就可以将这一种类的消息归为一个 Topic，给它取名为 order_info_topic，也就是一个包含了订单信息的数据集合。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200806192703768.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70)


接下来物流系统可以去这个 order_info_topic 中获取订单信息进行发货。简单的总结一下，Topic 并不具有真正的属性，它只是一类数据的集合，不同类型的数据我们应该放到不同的 Topic 中。



也就是说当有商品数据时，这时应该新建一个Topic，假设取名为 product_info_topic，代表这里面放的商品信息。获取商品数据时，消费者应该从这个 Topic 中获取，不会与之前的 order_info_topic 混淆。



当我们真正使用 MQ 时，第一步应该总是先创建一些 Topic，作为数据集合存放不同类型的消息，其实本质上来讲和使用数据库时总是先创建表结构是一样的。

## 02 Topic 是怎么在 Broker 集群里存储的?
我们知道 RocketMQ 肯定是分布式、集群化部署的，所以才能实现诸如读写分离、主从备份的功能。而 Topic 的存储方式，也分布式存储的一种体现。



设想一下，我们的 APP 非常受欢迎用户量很大，每天都会产生几百万条订单数据。所以系统不停会把数据投放至订单信息 order_info_topic 中，然后订单数据作为非常重要的一环，一般都会在 MQ 集群上再多保留一段时间，最终可能会有几千万的数据量堆积在这个 Topic 中。



而整个系统肯定有多个 Topic ，并且每个都有大量数据，加起来的总和也许是一个惊人的数字，这么大的量级不可能存放在一台机器上的，所以必然是分布式进行存储的。



我的观点是，其实分布式的本质就是三个臭皮匠顶的上一个诸葛亮。如果一台机器能力不够放不下，那就多叫几个帮手，一起放。



Topic 在创建的时候就可以指定让它把数据散落存储到多台 Broker 服务器上，比如一个 Topic 里有 3000 万条数据，此时有 3 台 Broker，那么每台 Broker 上都放 1000 万条数据。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200806192740128.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70)





## 03 生产者怎么发送消息到 Broker 的 Topic 中？
在讨论这个问题之前，我先简单介绍一个组件 NameServer ，目前大家可以简单理解它为整个 RockerMQ 的管理员，它可以看到集群里其他每个的组件的一举一动，下篇文章我再来详细介绍它。



在生产者发送消息之前，它会和 NameServer 建立一个TCP长连接，然后从 NameServer 那里拉取到最新的路由信息，包括集群里有哪些 Broker，集群里有哪些 Topic，每个 Topic 都存储在哪些 Broker 上。



然后生产者根据这些信息，就可以把自己生产的消息投递到对应的 Topic 内。由于一个 Topic 是分布在多台 Broker 上，可以根据负载均衡算法，无论是傻瓜式 round robine 轮询还是 hash 都可以，从里面选择一台 Broker 机器进行投递就行。



具体的 Broker 选择算法我后面会来细说，反正本质说白了就是生产者选择一台 Broker 之后，建立一个 TCP 长连接发送消息即可。



这里注意的一点，就是生产者一定是投递到 Master Broker 上的，然后 Master Broker 再同步数据给它的 Slave Brokers，实现主从备份，不记得的同学可以回顾一下之前的文章[《消息中间件的正确打开方式》](https://mp.weixin.qq.com/s/rHHyNcLcoOIh9vNhN-d-Yw)。

## 04 总结
本篇文章介绍了消息中间件 MQ 的一个最基础组件 Topic ，简单总结一下：

（1）Topic 是一类数据的集合；

（2）Topic 会分布式的进行存储；



## 文末福利



最近各大互联网公司的秋招都陆陆续续开始了，还在找工作的小伙伴可以后台**回复关键字进入对应的秋招/内推/面试群，我给大家整理了各大公司的内推通道、简历模板还有历年的笔试题**，大家要好好准备哦。还可以帮助大家**免费修改简历、模拟面试哦~**


![在这里插入图片描述](https://img-blog.csdnimg.cn/20200803164240831.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70)




![在这里插入图片描述](https://img-blog.csdnimg.cn/20200803164246398.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70)


**关注公众号「程序员大帝」，《Offer收割机》系列持续更新~**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200803165001620.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70)

创作不易，各位的支持和认可，就是我创作的最大动力，我们下篇文章见！




