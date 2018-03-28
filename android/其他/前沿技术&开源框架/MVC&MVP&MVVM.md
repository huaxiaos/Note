# MVC

最经典的组织架构

# MVP

问题

1. Presente层与View层是通过接口进行交互的，接口粒度不好控制。粒度太小，就会存在大量接口的情况，使代码太过碎版化;粒度太大，解耦效果不好。因为View定义的方法并不一定全部要用到，可能只是后面要用到先定义出来（后面要不要删也未知），而且如果后面有些方法要删改，Presenter和Activity都要删改，比较麻烦
2. 复杂的业务同时也可能会导致P层太大，代码臃肿的问题依然不能解决，这已经不是接口粒度把控的问题了，一旦业务逻辑越来越多，View定义的方法越来越多，会造成Activity和Fragment实现的方法越来越多，依然臃肿
3. V层与P层还是有一定的耦合度。一旦V层某个UI元素更改，那么对应的接口就必须得改，数据如何映射到UI上、事件监听接口这些都需要转变，牵一发而动全身。如果这一层也能解耦就更好了

# MVVM

## 优势

View—ViewModel—Model

1. 双向绑定，数据可以主导更新UI
2. Activity和Fragment、Fragment和Fragment之间的通信更加便捷
3. 如果Activity销毁重建，可以立即得到一个相同的`MyViewModel`实例，它是由之前销毁的Activity创建的。当宿主Activity最终销毁后，系统会调用ViewModel的`onCleared()`方法来释放资源

## Android Architecture Componets 组件

### ViewModel

在架构方案中，用于实现ViewModel层，同时默认绑定Lifecycle，生命周期可以使用Lifcycle中的生命回调来控制，可以脱离Activity或Fragment，在ViewModel中处理响应UI生命周期函数

### LiveData

使用观察者模式，用于观察数据返回，用过RxJava的都会觉得眼熟，可以替代部分RxJava的功能

### Lifecycle

有点像RxLifecycler的处理方式，监听生命周期中的事件

### Room

官方 Sqlite ORM库

# 参考资料

https://tech.meituan.com/android_mvvm.html