# 类加载器划分

- 启动类加载器(Bootstrap ClassLoader)
- 扩展类加载器(Extendsion ClassLoader)
- 应用程序类加载器(Application ClassLoader)

```
public class TestClassLoader {

    public static void main(String[] args) {
        ClassLoader loader = TestClassLoader.class.getClassLoader();
        System.out.println(loader.toString());
        System.out.println(loader.getParent().toString());
        System.out.println(loader.getParent().getParent());
    }
}

// log
sun.misc.Launcher$AppClassLoader@500c05c2
sun.misc.Launcher$ExtClassLoader@454e2c9c
null

```

# 双亲委派机制

双亲委派模型是一种组织类加载器之间关系的一种规范,他的工作原理是:如果一个类加载器收到了类加载的请求,它不会自己去尝试加载这个类,而是把这个请求委派给父类加载器去完成,这样层层递进,最终所有的加载请求都被传到最顶层的启动类加载器中,只有当父类加载器无法完成这个加载请求(它的搜索范围内没有找到所需的类)时,才会交给子类加载器去尝试加载

优点:java类随着它的类加载器一起具备了带有优先级的层次关系.这是十分必要的,比如java.langObject,它存放在\jre\lib\rt.jar中,它是所有java类的父类,因此无论哪个类加载都要加载这个类,最终所有的加载请求都汇总到顶层的启动类加载器中,因此Object类会由启动类加载器来加载,所以加载的都是同一个类,如果不使用双亲委派模型,由各个类加载器自行去加载的话,系统中就会出现不止一个Object类,应用程序就会全乱了

# 类加载耗时

## 背景

A方法里面初始化B类的单例，然后在A里面打日志查到的执行B类的初始化是用了100ms,但是在B类里面的构造方法里面打日志看到的只是用了20ms；

## 问题

中间相差的80ms用在了哪个地方呢？

## 答案

在第一次调用的时候B类还没有被加载的类的加载器里面，然后中间相关的80ms就是用于在加载这个类文件，加载完类文件之后才会去调用B类构造方法；通过 Class.forName 这个提前加载那个类：80ms，然后再调用构造方法,20ms，时间就对得上了

# 参考资料

- [https://juejin.im/post/5ae6613e6fb9a07aaa110802](https://juejin.im/post/5ae6613e6fb9a07aaa110802)