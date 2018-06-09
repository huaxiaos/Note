# 概述

1. 内部采用LinkedHashMap以强引用的方式存储外界缓存对象
2. 从Map中移除最近最少使用的对象，但对象的内存需要用户自己清除

# 原理

LinkedHashMap，内部有排序功能，数据在被get的时候就会排序，把最近访问的数据放到集合的最前面，后面的数据就是最近最少使用的，删除的时候从后面删除

例如 A B C D 四个数据，执行get B

- 先remove B
- 然后将B 放在表头，变成 B A C D
- 删除的时候，删除最后一个item

```
/**
 * Returns the value for {@code key} if it exists in the cache or can be
 * created by {@code #create}. If a value was returned, it is moved to the
 * head of the queue. This returns null if a value is not cached and cannot
 * be created.
 */
public final V get(K key) {
    
}

/**
 * Caches {@code value} for {@code key}. The value is moved to the head of
 * the queue.
 *
 * @return the previous value mapped by {@code key}.
 */
public final V put(K key, V value) {

}
```

LruCache可以设置最大值