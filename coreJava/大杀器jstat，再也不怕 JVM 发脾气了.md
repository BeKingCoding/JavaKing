

> 本文首发于公众号【 **程序员大帝** 】，关注第一时间获取

> 一起码动人生，成为Coding King！！！

> GitHub上已经开源 [https://github.com/BeKingCoding/JavaKing](https://github.com/BeKingCoding/JavaKing) ， 一线大厂面试核心知识点、我的联系方式和技术交流群，欢迎Star和完善

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200718224809998.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70)
## 前言

上一篇文章《[重大事故！线上系统频繁卡死，凶手竟然是 Full GC ？](https://mp.weixin.qq.com/s/XYwkZPCrSsWaFKl2xJs4yg)》，咱们通过一个现实的故障讲解了如果 JVM 出现问题，如何快速进行定位与排查。



其中提到了 jstat 这样一个工具，它可以很好地帮助我们了解 JVM 运行情况，轻易地实时显示 JVM 内的 Eden、Survivor、老年代的内存使用情况，还有 Young GC 和 Full GC 的执行次数以及耗时。



通过这些指标，我们可以按照之前的分析过程，判断系统的压力和内存分配是否合理。很多小伙伴看完上篇文章，都要求好好讲讲 jstat，接下来我们就来看看如何使用这个小而强大的工具。

## 正文
## 01 jstat -gc [PID]
默认大家的服务器都是Linux，可以通过 jps 命令先获取 Java 进程号，然后通过这个命令就可以显示 JVM 的内存和 GC 情况了。







命令运行后会告诉我们很多重要的指标，图片比较小，我们就把每一列挨个来解释下：


|  |  |
|--|--|
|S0C	|From Survivor 区的大小|
|S1C	|To Survivor 区的大小
|SOU	|From Survivor 区已使用的内存大小
|S1U|To Survivor 区已使用的内存大小
EG	|Eden 区的大小
EU	|Eden 区已使用的内存大小
OC	|老年代的大小
OU	|老年代已使用的内存大小
MC	|方法区（元空间）的大小
MU	|方法区（元空间）已使用的内存大小
YGG|	系统运行迄今为止的 Young GG 次数
YGGT|	Young GC 的耗时
FGC|	系统运行迄今为止的 Full GC 次数
FGCT|	Full GC 的耗时
GCT|	所有 GC 的总耗时


## 02 如何利用 jstat
其实 jstat -gc 命令基本上可以满足对 JVM 的初步分析了，在得到这么多指标之后，我们还需要对信息进一步加工。



为大家讲述一下，其实在了解 JVM 进程时，我们最想要知道的信息有哪些呢？



（1）Eden区、老年代对象的增大速率



（2）Young GC、Full GC 触发的频率和每次的耗时



（3）每次 Young GC 后有多少对象可以存活下来，又有多少进入了老年代



我们可以遵循一个最基本的原则，那就是合理分配内存空间，尽可能让对象留在年轻代不进入老年代，避免发生频繁的FullGC，这就是对 JVM 好的性能优化了！



## 03 总结
本文主要是对上几篇内容的补充，希望可以让大家学会 jstat 命令的使用，对我们定位和排查 JVM 相关问题有很大的帮助。



大家完全可以灵活运行 jstat 这个实用的工具，轻而易举的掌控到线上 JVM 运行的详细情况，然后针对 JVM 的具体运行情况去针对性地优化。



有的同学可能来自于比较成熟的大公司，已经有可视化的监控工具，会说无忌，为什么我还要背命令去记这么多参数呢？



针对这个问题，我也没说不可以使用可视化工具，但是告诉大家，一个优秀、合格的工程师，他一定是可以非常灵活的运用各种命令行工具，在命令行就搞定 一切的。



所以 jstat 作为一个最简单易用、高效实用的命令行 JVM 监控工具，绝对值得大家掌握。因为每个人的公司情况不一样，万一你的公司不提供各种可视化工具呢？



那么我们就必须从最 low 最原始的命令行工具开始，快速上手使用，定位问题。在理解了本文思想之后，你用任何其他工具，都能轻松的把线上 JVM 的运行情况通过工具提供的数据分析清楚。


## 日常求赞
好了各位，以上就是这篇文章的全部内容了，能看到这里的人呀，都是人才。

我后面会持续更新《Offer收割机》系列和互联网常用技术栈相关的文章，非常感谢各位老板能看到这里，如果这个文章写得还不错，觉得我有点东西的话，求点赞👍 求关注❤️ 求分享👥 对我来说真的非常有用！！！

创作不易，各位的支持和认可，就是我创作的最大动力，我们下篇文章见！

Craig无忌 | 文 【原创】【转载请联系本人】 如果本篇博客有任何错误，请批评指教，不胜感激 ！

------

>《Offer收割机》系列会持续更新，可以关注我的公众号「 程序员大帝 」第一时间阅读和催更（公众号比博客早一到两篇哟），本文GitHub上已经收录 [https://github.com/BeKingCoding/JavaKing](https://github.com/BeKingCoding/JavaKing) ，会对一线大厂面试要求的核心知识点持续讲解、并有对标阿里P7级别的成长体系脑图，欢迎加入技术交流群，我们一起有点东西。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200715124857432.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2tpbmdjb2Rpbmc=,size_16,color_FFFFFF,t_70#pic_center)