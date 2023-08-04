---
layout:     post
title:      一致性哈希算法(Consistent Hashing)
subtitle:   简单讲了一下背景，使用场景和原理
date:       2023-08-04
author:     yebin-yu
header-img: img/post-bg-debug.png
catalog: true
tags:
    - 算法
    - 分布式
---

# 产生背景及原理

**先给结论：一致性哈希是一种分布式哈希方案，使服务器的伸缩不会给整个系统带来大问题。**

**一致性哈希只是一类算法的统称，细分还有很多具体的算法。**

**一致性hash算法是分布式系统负载均衡的首选算法。**



### 一致性哈希是为了解决什么问题？

设想一下，你有n台cache machine，肯定需要做**负载均衡(Load Balance)**，那么最直观的方法就是将将对象`o` 放在第 `hash(o) mod n` 台machine上，这样每台machine都独享同样大的数字空间（如果hash值有sum位，那么每台machine都能对应上其中的 `sum/n` 位）。

正常运行时，大家都相安无事。

但如果需要新增一台machine，或者可能性更大的，有一台machine down了，这个时候咋办？

按照之前的方式，n发生变化之后，同一个object会被hash到不同的machine机器上。新来的cache可以用新的规则去hash到对应的machine上，那原来已经存入machine的cache数据呢？这会导致在这时所有的cache都是不可用的，因为hash映射machine的算法已经变了，新的算法会导致原来的对象被hash到不同的设备上去，这就是重新哈希问题。

而这就是一致性哈希所希望解决的**一致性consistent—— 尽可能一致地将同一对象映射到同一台缓存机**。

> This is why you should care - consistent hashing is needed to avoid swamping your servers!



### **一致性哈希的目标**

1997年，Karger等人发布了论文[ 《Consistent Hashing and Random Trees: Distributed Caching Protocols for Relieving Hot Spots on the WWW》 ](https://www.akamai.com/us/en/multimedia/documents/technical-publication/consistent-hashing-and-random-trees-distributed-caching-protocols-for-relieving-hot-spots-on-the-world-wide-web-technical-publication.pdf)，并提出了一致性哈希

在论文中还对一致性哈希的算法好坏定义给出了4个评判指标:

**评判指标**

- **平衡性（Balance）**不同key的哈希结果分布均衡，尽可能的均衡地分布到各节点上。平衡性跟哈希函数关系密切，目前许多哈希算法都有较好的平衡性。

- **单调性（Monotonicity）**当有新的节点上线后，系统中原有的key要么还是映射到原来的节点上，映射到新加入的节点上，不会出现从一个老节点重新映射到另一个老节点。即表示：当可用存储桶的集合发生更改时，只有在必要时才移动项目以保持均匀的分布。

- **分散性（Spread）**由于客户端可能看不到后端的所有服务，这种情况下对于固定的key，在两个客户端上可能被分散到不同的后端服务，从而降低后端存储的效率，所以算法应该尽量降低分散性。

- **服务器负载均衡（Load）**负载主要是从服务器的角度来看，指各服务器的负载应该尽量均衡



### 原理

[![img](https://tom-e-white.com/assets/2007-11-27-image-0000.png)](https://tom-e-white.com/assets/2007-11-27-image-0003.png)

以java为例，hash的范围是int的范围，也就是[2^31-1, -2^31]，我们把这个范围展开成一个线段，将线段收尾相接，就成了一个环，如上图。

ABC表示三台不同的machine，一致性哈希算法的原理就是：当输入一个对象的hash的时候（图中表示为123），让他顺时针(逆时针也可以)顺着圆环走，走到的第一个machine就是目标machine。

例如图中的1，往后走会对应到A。同理，2对应B，3对应C，4对应A。

这样，当machine有新增或减少的时候，只会影响这个machine在环中对应的下一个machine。

##### 问题与优化1 - 如何保证machine在圆环上映射的均匀呢？新增或者删除如何保证影响的内容平均分摊给其他machine呢？

这里引入了一个概念，**虚拟节点**

我们为每台machine分配多个虚拟节点，每台machine对应的虚拟节点的数量（权重）取决于machine的处理性能。例如，如果某台machine的处理性能是其他machine的两倍时，它可以分配其他machine两倍的虚拟节点。

> 图片来源 [Consistent hashing, why are Vnodes a thing?](https://stackoverflow.com/questions/69841546/consistent-hashing-why-are-vnodes-a-thing)

![](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRhezms6erQn9Wk6FVkkE5XmG-_9O0AWJytwLiF618PN1HYgc-errXIojQCSHcYUV_rNs4&usqp=CAU)

如图显示，圆环上仅有四个物理节点，但却又十几个虚拟节点，一个物理节点对应多个虚拟节点。通过交叉分配这些节点的位置，可以做到数据倾斜的防护。

例如这里删掉了Node 0，那么图中四个Node 0节点的下一个节点都会受到影响，这时候就不会只有一个物理节点来全盘接受Node 0 的所有内容了。




# REFERENCES

[Consistent Hashing](https://tom-e-white.com/2007/11/consistent-hashing.html)

