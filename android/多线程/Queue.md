# LinkedBlockingQueue和ArrayBlockingQueue的区别

除内部容器不同（一个是数组，一个是链表），链表的添加删除需要额外创建和删除Node对象外，主要的区别是以下几点：

1. 队列大小有所不同，ArrayBlockingQueue是有界的初始化必须指定大小，而LinkedBlockingQueue可以是有界的也可以是无界的(Integer.MAX_VALUE)，对于后者而言，当添加速度大于移除速度时，在无界的情况下，可能会造成内存溢出等问题
2. 两者的实现队列添加或移除的锁不一样，ArrayBlockingQueue实现的队列中的锁是没有分离的，即添加操作和移除操作采用的同一个ReenterLock锁，而LinkedBlockingQueue实现的队列中的锁是分离的，其添加采用的是putLock，移除采用的则是takeLock，这样能大大提高队列的吞吐量，也意味着在高并发的情况下生产者和消费者可以并行地操作队列中的数据，以此来提高整个队列的并发性能

# BlockingQueue

BlockingQueue是一个接口类，可以用来线程安全的存取数据。典型的应用是生产者/消费者模式

## 基本操作

- put(Object)
- take()

## 实现类

- ArrayBlockingQueue


- DelayQueue
- LinkedBlockingQueue
- PriorityBlockingQueue
- SynchronousQueue

# BlockingDeque

和BlockingQueue类似，但支持两头入队/出队