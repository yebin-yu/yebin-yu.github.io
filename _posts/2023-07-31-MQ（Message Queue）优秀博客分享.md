---
layout:     post
title:         MQ（Message Queue）优秀博客分享
subtitle:   找到一组很详细的MQ讲解
date:       2023-07-31
author:     yebin-yu
header-img: img/post-bg-debug.png
catalog: false
tags:
    - MQ
---

## MQ（Message Queue）优秀博客分享

[MQ系列1：消息中间件执行原理](https://www.cnblogs.com/wzh2010/p/15888498.html)
[MQ系列2：消息中间件的技术选型 ](https://www.cnblogs.com/wzh2010/p/15311174.html)
[MQ系列3：RocketMQ 架构分析](https://www.cnblogs.com/wzh2010/p/16556570.html)
[MQ系列4：NameServer 原理解析](https://www.cnblogs.com/wzh2010/p/16607258.html)
[MQ系列5：RocketMQ消息的发送模式](https://www.cnblogs.com/wzh2010/p/16629876.html)
[MQ系列6：消息的消费](https://www.cnblogs.com/wzh2010/p/16631097.html)
[MQ系列7：消息通信，追求极致性能](https://www.cnblogs.com/wzh2010/p/16631103.html)
[MQ系列8：数据存储，消息队列的高可用保障](https://www.cnblogs.com/wzh2010/p/16631107.html)
[MQ系列9：高可用架构分析](https://www.cnblogs.com/wzh2010/p/15888521.html)
[MQ系列10：如何保证消息幂等性消费](https://www.cnblogs.com/wzh2010/p/15888523.html)
[MQ系列11：如何保证消息可靠性传输](https://www.cnblogs.com/wzh2010/p/15888525.html)
[MQ系列12：如何保证消息顺序性](https://www.cnblogs.com/wzh2010/p/15888528.html)
[MQ系列13：消息大量堆积如何为解决 ](https://www.cnblogs.com/wzh2010/p/15888534.html)



## MQ选型

> 参考 [MQ系列2：消息中间件的技术选型 ](https://www.cnblogs.com/wzh2010/p/15311174.html)

中小型系统建议选用RabbitMQ，数据量相对较小，选型应首选功能比较完备的，所以kafka排除。RocketMQ是阿里出品，如果阿里放弃维护，中小型公司一般很难投入人力进行RocketMQ的定制化开发，因此不推荐。
根据具体使用规模在RocketMQ和kafka之间二选一。

大型业务系统：有实际的业务体量需求，比如足够大规模的分布式环境，以及足够大的数据量。这时候 RocketMQ  和 kafka 都是10w+的吞吐量，都可以在考虑范围内。

如果你有业务定制需求，可以优先选用RocketMQ，毕竟是开源的，大的业务系统也愿意花精力去优化JAVA源码的。至于kafka，根据业务方向选择，类似日志采集功能，首选kafka，因为他在日志上报、监控数据采集方面有着大规模的实践经验，这也是他们主打的应用场景。

具体该选哪个，看使用场景。引入MQ之后，也会有一定的弊端，必然一定程度上降低系统可用性，增加复杂性。



比如我开发过的一个问题定位相关的工具，选型就用RabbitMQ，原因是：

1. 消息数量很少很少，一秒最多几十个。
2. 因为消息数量少，所以不需要分布式，主从模式就行。
3. 因为消息数量少，可以一直开着rabbitmq_tracing来消息跟踪
4. 任务有优先级，需要优先级队列。
5. 需要根据consumer的情况来消费，所以需要”push-pull“模式
6. 有的消息push上去后，还不能马上执行（待执行机器需要等待启动等），需要延迟队列



但rabbitmq不支持幂等性，好在producer功能简单，而且消息少，用主从模式（主ok的时候从不启用），就可以在producer端控制幂等性。

