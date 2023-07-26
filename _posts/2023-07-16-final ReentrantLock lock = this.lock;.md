---
layout:     post
title:      final ReentrantLock lock = this.lock;
subtitle:   为啥要这么写？
date:       2023-07-16
author:     yebin-yu
header-img: img/tag-bg.jpg
catalog: false
tags:
    - Java Tips
---

#### final ReentrantLock lock = this.lock; 为啥要这么写？

最近看JDK源码的时候都看到过类似这样的语句

```java
    public E pollFirst() {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            return unlinkFirst();
        } finally {
            lock.unlock();
        }
    }
```

这应该是原作者的解释

> It's ultimately due to the fundamental mismatch between memory models and OOP
>
> Just about every method in all of j.u.c adopts the policy of reading fields as locals whenever a value is used more than once. This way you are sure which value applies when. This is not often pretty, but is easier to visually verify.
>
> This surprising case is doing this even for "final" fields. This is because JVMs are not always smart enough to exploit the fine points of the JMM and not reload read final values, as they would otherwise need to do across the volatile accesses entailed in locking. Some JVMs are smarter than they used to be about this, but still not always smart enough.

1、this.lock是类的成员变量，一般都是存到堆上，访问堆上的变量会涉及内存同步的操作(这个建议通过编译后的bytecode进行观察)，而将其copy到栈上，然后访问就不存在这个问题了；

2、在访问堆上的this.lock时，对于多个CPU，可能会存在cache命中的问题，这样必然会导致内存重新load，而copy到栈上，则直接是线程相关的，就不存在这个问题了。

#### 参考

https://toutiao.io/posts/v8pfwy/preview

https://blog.csdn.net/zqz_zqz/article/details/79438502
