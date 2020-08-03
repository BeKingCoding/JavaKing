

> 开足码力，码动人生，本文首发公众号【 **程序员大帝** 】，关注这个一言不合就开车的的代码界老司机

> 本文 **GitHub**上已经收录 [https://github.com/BeKingCoding/JavaKing](https://github.com/BeKingCoding/JavaKing) ， 一线大厂面试核心知识点、我的联系方式和技术交流群，欢迎Star和完善

## 前言

HashMap 作为 Java 语言中经典的数据结构之一，在面试中出现的频率几乎是最多的。它本身有太多的点可以深入探讨，这里不得不佩服大神 Doug Lea 的水平，每一次读源码都感叹它的优美。



如果大家仔细看过源码的话，应该知道 HashMap 大小的初始值为16，如果要指定的话，必须是2的指数幂。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200730194405583.png)




但是你有没有想过为什么要这样设计呢？其实背后的思路很巧妙，本篇文章我们就来探索一下。

## 正文

我们知道 HashMap 是一个 K-V 存储的数据结构，在 JDK8 之后升级为数组 + 链表 + 红黑树的形式。 



当需要对KV对放入 HashMap 中时，会按照下面的步骤来完成：

（1）首先对 Key 做哈希运算，得到它的哈希值；



（2）根据哈希值值，按照 HashMap 的寻址算法得到 index 值，也就是对应应该放入数组的位置；

```java
index = hashcode & （length -1）
```

（3）如果发生哈希冲突，则会通过拉链法来处理；



而 HashMap 为什么容量必须是 2 的指数幂就是因为由上述的寻址算法要求的。



当 HashMap 的容量为 2 指数幂时，那么length -1 后，十进制换算成二进制你会发现每一位都是1。



比如我们假设容量 length=32，则 length-1后为31，转换为二进制是 11111。这样当 hashcode 和它进行与运算（两个同时为1，结果为1，否则为0）的时候，可以得到（ 0 ~ 31 ）范围内的每一个值。



但如果容量不为 2 的指数幂，比如 length=50，那么 length -1=49，换算为二进制是110001。



那么无论一个 hashcode是什么， 和 49 （即110001）与运算后的结果只能为以下几种，因为中间的三位肯定是零：

000000  ： 0

000001  ：1

010000  ：16

010001  ：17

100000  ：32

100001  ：33

110000  ：48

110001  ：49





## 04 絮叨

最近无忌的工作有点忙，写技术文章的频率有所下降，之前答应大家日更，果然被打脸了.



最近各大互联网公司的秋招都陆陆续续开始了，还在找工作的小伙伴可以后台回复「**进群**」，进入秋招抢跑群，我给大家整理了各大公司的内推通道、简历模板还有历年的笔试题，大家要好好准备哦。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200730194542749.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200730194547486.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70)





**欢迎更多有需要对简历进行修改，包括工作建议的同学，直接后台联系我，看完简历后我会和你语音进行沟通。**


-----

>《**Offer收割机**》系列持续更新，也会定期分享互联网常用技术栈相关的文章，**GitHub** 上已经收录 [https://github.com/BeKingCoding/JavaKing](https://github.com/BeKingCoding/JavaKing) ，讲解一线大厂面试要求的核心知识点、并有对标阿里P7级别的成长体系脑图，欢迎加入技术交流群，我们一起有点东西。


<br>

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200723161809991.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70#pic_center)

<br>

我是无忌，创作不易，各位的支持和认可，就是我创作的最大动力，我们下篇文章见！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200723153618185.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70)
