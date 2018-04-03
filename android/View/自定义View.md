# 三个过程

measure、layout、draw

# invalidate

请求重绘View树，即draw过程

当View的appearance发生改变，比如状态改变（enable，focus），背景改变，隐显改变等，这些都属于appearance范畴，都会引起invalidate操作

所以当我们改变了View的appearance，需要更新界面显示，就可以直接调用invalidate方法

# requestLayout

只是对View树重新布局layout过程包括measure和layout过程，不会调用draw()过程

View执行requestLayout方法，会向上递归到顶级父View中，再执行这个顶级父View的requestLayout，所以其他View的onMeasure，onLayout也可能会被调用