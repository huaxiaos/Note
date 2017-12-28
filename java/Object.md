# equals & == 的区别

- == 比较的是两个对象的内存地址是否相等
- 如果不重写equals方法，那么和==是一样的，因为equals的源码默认返回的就是==的比较结果

# equals & hashCode

- 每个java对象都有一个hashCode，主要用于查找的快捷性，比如HashMap中就用到了hashCode
- 如果两个对象equals比较相等，那么这两个对象的hashCode肯定相等
- 反过来，如果两个对象的hashCode相等，但equals可能不相等，比如，HashMap中hashCode相等的值，会放入相应索引的链表中