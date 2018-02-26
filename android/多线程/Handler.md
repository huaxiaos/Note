# MessageQueue

完全自定义的Queue，没有继承自Queue（类似ActivityThread，并没有继承自Thread）

内部采用链表结构实现，这样保证了消息的时序性（可以参照sendMessageAtTime的实现）

消息队列的处理，是线程安全的，使用了对类的同步，synchronized (this)