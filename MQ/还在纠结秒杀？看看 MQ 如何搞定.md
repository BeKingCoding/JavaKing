

> 本文首发于公众号【 **程序员大帝** 】，关注第一时间获取

> 一起码动人生，成为Coding King！！！

> GitHub上已经开源 [https://github.com/BeKingCoding/JavaKing](https://github.com/BeKingCoding/JavaKing) ， 一线大厂面试核心知识点、我的联系方式和技术交流群，欢迎Star和完善


![在这里插入图片描述](https://img-blog.csdnimg.cn/20200708132205796.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70)

## 前言

相信即使你是第一次听说消息中间件的概念，通过上一篇文章《[十分钟入门消息中间件](https://blog.csdn.net/kingcoding/article/details/107171107)》的介绍，现在应该知道什么是 MQ ，为什么要使用 MQ 了。



我们先简单回顾一下，订单系统是整个电商交易平台的核心，在它与内部模块、外部第三方系统打交道的过程中，需要完成很多额外的步骤：

- 为用户积分

- 发放红包卡券

- 库存扣减

- 通知物流系统

- 发送短信通知

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200708184229161.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70)

在引入 MQ 后，我们可以让订单系统仅仅完成最核心的功能，然后将发送消息到 MQ。比如需要进行减库存，就发送一个消息到库存消息队列中，然后库存系统从这个 MQ 里获取消息再进行处理就可以，把这些很耗时的步骤慢慢执行，从而也实现了系统之间的解耦。

但 MQ 还有一个更加强大的功能：缓冲流量，削峰填谷，诸如双11这样的大促活动，瞬时间涌入的大量下单请求有可能直接压垮服务器，导致整个系统的瘫痪，通过使用 MQ 我们可以更好地解决流量突刺的问题。



本文就以秒杀场景为基础，看完后会有更清晰的认识。本文将会从以下几个方面来讲述相关知识，相信大家耐心看了之后肯定有收获，码字不易，别忘了「在看」，「转发」哦。

- 经典的秒杀场景
- 流量洪峰带来的难点
- 使用 MQ 削峰填谷
- 升级整体架构

## 正文 

## 01 经典的秒杀场景

秒杀场景一般出现在类电商的 APP 中，双十一、618 这种打折大促已经屡见不鲜，各种节日都能成为剁手的理由。



就连饥饿营销也被各大公司玩的越来越6，像我自己喜欢球鞋，喜欢买AJ和椰子，有过抢鞋经历的同学一定知道有多痛苦，尤其是每次的结果都是这张图。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200708184240261.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70)



每年的双 11 活动在零点之后会开启一个特别大的折扣优惠，比如前五分钟下单买五件可以享受三折优惠。



全中国无数的男生女生，在双 11 之前几天就会在购物车精挑细选大量的商品，零点一到，疯狂点击下单。



每个接口对应的业务复杂度不同，有的接口一个请求可能要执行五六次数据库操作。即使我们用高配置的 16 核 32G 以及 SSD 固态硬盘的机器，当流量洪峰到来时，CPU、磁盘、IO 等负载都会瞬间飙升，一段时间后，系统是无论如何扛不住持续到达的请求。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200708184245635.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70)






## 02 流量洪峰带来的难点

经典的秒杀场景主要有两个特点：

（1）秒杀时大量用户会在同一时间同时进行抢购，网站瞬时访问流量激增；



（2）秒杀一般是访问请求量远远大于库存数量，只有少部分用户能够秒杀成功。



随之而来，对我们的系统就带来一系列要求：

（1）短时间内并发量激增，对系统负载压力大，容易造成崩溃；



（2）真正能秒杀成功的还是少数，天然的读多写少场景，如何对流量进行分配；



（3）分布式的环境下，如何做到一致性的处理；



（4）竞争资源有限，如何做到精准的控制，准时准点，不能多买，不能少卖，不能重复买。



而在这么多难点中，我们首要任务就是去解决并发量激增的问题，如果到达的请求太多直接压垮了服务器，那其他的功能根本无从谈起。

## 03 使用 MQ 削峰填谷

MQ 除了可以使用异步的方式实现系统间的解耦，更可以在双 11 这样的秒杀活动中，通过削峰填谷的方式，处理瞬时间涌入的大量请求。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200708184250887.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70)


什么是削峰填谷？



削峰填谷本身是电力行业的概念，电力企业通过必要的技术和管理手段，降低电网的高峰负荷，提高低谷负荷，平滑负荷曲线，提高负荷率，保证电网的稳定运行。



假设一个应用，它能够每秒处理 1000 个请求。如果在第一秒接收到 2000 个请求，而接下来的两秒都没有请求到达。



![在这里插入图片描述](https://img-blog.csdnimg.cn/20200708184255733.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70)


整个应用必然面临两个问题：

（1）在第一秒被 2000 个请求直接压垮；



（2）假设第一秒没有被压垮，它在这一秒时间内只能处理 1000 请求，第二第三秒却完全空闲，浪费了系统资源。



所以，我们可以通过 MQ 把请求突刺均摊到一段时间内，让系统负载保持在请求处理水位之内，同时尽可能地处理更多请求，从而起到“削峰填谷”的效果。



红色的部分是超出系统处理能力的部分，可以把红色的那部分消息平摊到后面空闲时去处理，这样既可以保证系统负载处在一个稳定的水位，又可以尽可能地处理更多消息。通过配置流控规则，可以达到消息匀速处理的效果。

## 04 升级系统架构

随着使用应用的用户越来越多，系统面临的压力也会越来越大，无论是并发量还是数据量，你会发现整个系统各个模块都需要进行优化。



一个高并发、大数据量的系统架构，需要不断的迭代和进化，涉及到大量的技术方案、架构重构。



针对秒杀的场景，上游发起高并发的下单操作，由于下游处理能力有限，两端速度不匹配。此时我们引入 MQ 可以对流量进行缓冲，并实现削峰填谷。


![在这里插入图片描述](https://img-blog.csdnimg.cn/20200708184305649.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70)


上游速度很快，每秒发起五万个请求也没关系，它只管往 MQ 中发。下游系统虽然每秒只能处理 1000 个请求，但它完全可以 follow 自己的节奏，每隔一段时间，主动拉取若干条信息，实施限流的效果，保护自身。



而在这个过程中，只需要引入 MQ 组件，对上下游的业务代码并不用有太多的修改。



在接下来的文章我会更加详细介绍，使用现在主流的消息中间件 RocketMQ 进行实现，敬请期待~



## 日常求赞
好了各位，以上就是这篇文章的全部内容了，能看到这里的人呀，都是人才。

我后面会持续更新《Offer收割机》系列和互联网常用技术栈相关的文章，非常感谢各位老板能看到这里，如果这个文章写得还不错，觉得我有点东西的话，求点赞👍 求关注❤️ 求分享👥 对我来说真的非常有用！！！

创作不易，各位的支持和认可，就是我创作的最大动力，我们下篇文章见！

Craig无忌 | 文 【原创】【转载请联系本人】 如果本篇博客有任何错误，请批评指教，不胜感激 ！

------

>《Offer收割机》系列会持续更新，可以关注我的公众号「 程序员大帝 」第一时间阅读和催更（公众号比博客早一到两篇哟），本文GitHub上已经收录 [https://github.com/BeKingCoding/JavaKing](https://github.com/BeKingCoding/JavaKing) ，会对一线大厂面试要求的核心知识点持续讲解、并有对标阿里P7级别的成长体系脑图，欢迎加入技术交流群，我们一起有点东西。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200715124857432.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70#pic_center)
