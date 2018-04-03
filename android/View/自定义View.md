# 三个过程

measure、layout、draw

# invalidate

请求重绘View树，即draw过程，假如视图发生大小没有变化就不会调用layout过程

# requestLayout

只是对View树重新布局layout过程包括measure和layout过程，不会调用draw()过程