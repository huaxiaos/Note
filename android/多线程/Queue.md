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