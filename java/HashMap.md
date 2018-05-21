# HashMap 数据结构

数组 + 链表 + 红黑树

- Node[] table，即哈希桶数组
- 默认情况下，链表长度大于8时，转换为红黑树

> 什么是红黑树？
> 
> 先说下二叉查找树BST，它的左子节点的值比父节点的值要小，右节点的值要比父节点的值大。它的高度决定了它的查找效率
> 
> BST存在的主要问题是，数在插入的时候会导致树倾斜，不同的插入顺序会导致树的高度不一样，而树的高度直接的影响了树的查找效率。理想的高度是logN，最坏的情况是所有的节点都在一条斜线上，这样的树的高度为N
> 
> 红黑树在插入和删除的时候，会通过旋转操作将高度保持在logN
> 
> 1. 任何一个节点都有颜色，黑色或者红色
> 2. 根节点是黑色的
> 3. 父子节点之间不能出现两个连续的红节点
> 4. 任何一个节点向下遍历到其子孙的叶子节点，所经过的黑节点个数必须相等
> 5. 空节点被认为是黑色的

# Hash冲突解决办法

链地址法

优缺点

- 开放地址法
	- 容易产生堆积问题；
	- 不适于大规模的数据存储；
	- 插入时可能会出现多次冲突的现象
	- 结点规模很大时会浪费很多空间；

- 链地址法
	- 无堆积现象
	- 链表中的结点是动态申请的，适合构造表不能确定长度的情况
	- 相对而言，拉链法的指针域可以忽略不计，因此较开放地址法更加节省空间

# 影响HashMap性能的关键参数

- length，Node[] table的初始化长度（默认16）
- Load factor，负载因子（默认0.75）

# Resize

- resize时，HashMap使用新数组代替旧数组，对原有的元素根据hash值重新计算索引位置，重新安放所有对象（transfer方法）
- resize是耗时的操作
- 扩大至原来的2倍

# HashMap容量为什么是2次幂？

主要是为了在取模（hash%length）和扩容时做优化

# HashTable

## HashTable为什么线程安全？

Hashtable的源码里，put,get,contains等绝大多数方法都加了 synchronized 关键字

## HashMap & HashTable 区别

- 线程同步
- null
	- HashTable不允许
	- HashMap允许
- 迭代器不同
- hash值的使用不同
	- HashTable直接使用hash值
	- HashMap取模
- 初始容量、扩容不同

## 弃用？

ConcurrentHashMap代替HashTable

# ConcurrentHashMap

## 相对于HashTable的优化

主要体现在锁机制不同

- Hashtable中采用的锁机制是一次锁住整个hash表，从而同一时刻只能由一个线程对其进行操作
- ConcurrentHashMap中则是一次锁住一个桶(segment，一个segment，守护多个数组)。ConcurrentHashMap默认将hash表分为16个桶，只锁当前需要用到的桶。同时可以有多个线程进行操作

## put

如果需要插入一个新节点到链表中时 , 会在链表头部插入这个新节点。此时，链表中的原有节点的链接并没有被修改。也就是说：插入新健 / 值对到链表中的操作不会影响读线程正常遍历这个链表

## remove

在执行 remove 操作时，原始链表并没有被修改。删除节点 C 之后的所有节点原样保留到新链表中，但是会被反转

# Fail-Fast & Fail-Safe

## Fail-Fast

Fail-fast 机制是 Java 集合中的一种错误机制。 当多个线程对同一个集合的内容进行操作时，就可能会产生 Fail-fast 事件

HashMap、HashTable、ArrayList、Vector都会产生Fail-Fas

原理：Iterator 时，如果 expectedModCount 与 modCount 不相等，则会报异常

## Fail-Safe

ConcurrentHashMap、CopyOnWriterArrayList

原理：在原集合的 copy 上遍历

缺点

- 创建 copy 需要额外的空间和时间上的开销
- 不能保证遍历的是最新的内容

# 参考资料

- [https://tech.meituan.com/java-hashmap.html](https://tech.meituan.com/java-hashmap.html)