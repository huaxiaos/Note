# GridView引发的性能优化

## 背景

TabLayout+ViewPager+Fragment+GridView实现照片墙，ImageView里面的图片全部是异步加载，滑动卡顿

## 解决过程

首先排除了itemView过度绘制的问题，因为整体UI设计比较简单。之后，定位在Adapter中getView方法耗时过长的问题，优化了之后，发现仍旧卡顿。线索一度中断，最后在不断的log分析后发现，getView在position为0的这个位置上，多次执行（一次初始化会执行10+次）

如果列表项中有一个图片是后加载的，图片加载完后，ImageView会执行requestLayout()。这将会导致GridView绘制第一项。如果总共有10项。最坏的情况，首项将会被绘制10次甚至更多。

## 优化方案

可以考虑重写GridView，对子item的view的绘制做大小的固定限制，但这种方法可能会引起其他问题。

最好的方案是将GridView换成RecycleView来实现。