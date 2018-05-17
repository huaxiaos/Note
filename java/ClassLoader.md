# 类加载过程

类加载的过程包括了加载、验证、准备、解析、初始化五个阶段

## 加载

主要完成下面这三件事情

1. 通过一个类的`全限定名`来获取其定义的二进制字节流。
2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
3. 在Java堆中生成一个代表这个类的java.lang.Class对象，作为对方法区中这些数据的访问入口

相对于类加载的其他阶段而言，加载阶段（准确地说，是加载阶段获取类的二进制字节流的动作）是可控性最强的阶段，因为开发人员既可以使用系统提供的类加载器来完成加载，也可以自定义自己的类加载器来完成加载

> 全限定名：在常量池中， 一个类型的名字并不是我们在源文件中看到的那样， 也不是我们在源文件中使用的包名加类名的形式。 源文件中的全限定名和class文件中的全限定名不是相同的概念。 源文件中的全新定名是包名加类名， 包名的各个部分之间，包名和类名之间， 使用点号分割。 如Object类， 在源文件中的全限定名是java.lang.Object 。 而class文件中的全限定名是将点号替换成“/” 。 例如， Object类在class文件中的全限定名是 java/lang/Object

## 验证

验证的目的是为了确保Class文件中的字节流包含的信息符合当前虚拟机的要求，而且不会危害虚拟机自身的安全

- 文件格式的验证
- 元数据验证
- 字节码验证
- 符号引用验证

## 准备

准备阶段是为类的静态变量分配内存并将其初始化为默认值，这些内存都将在方法区中进行分配

- 这时候进行内存分配的仅包括类变量（static），而不包括实例变量，实例变量会在对象实例化时随着对象一块分配在Java堆中
- 这里所设置的初始值通常情况下是数据类型默认的零值（如0、0L、null、false等），而不是被在Java代码中被显式地赋予的值。
	- public static int value=123
	- 在准备阶段value初始值为0，在初始化阶段才会变为123

## 解析

解析阶段是虚拟机将常量池中的符号引用转化为直接引用的过程，解析动作主要针对类或接口、字段、类方法、接口方法四类符号引用进行

- 符号引用
	- 符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可。
	- 符号引用与虚拟机实现的内存布局无关
	- 引用的目标并不一定已经加载到内存中。
- 直接引用
	- 直接引用可以是直接指向目标的指针、相对偏移量或是一个能间接定位到目标的句柄。
	- 直接引用是与虚拟机实现的内存布局相关的
	- 如果有了直接引用，那么引用的目标必定已经在内存中存在。

## 初始化

真正开始执行类中定义的Java程序代码的阶段，变量被赋值的阶段

何时开始类的初始化？

- 创建类的实例（隐式）
- 访问类的静态变量、静态方法（隐式）
- 反射（显示）

初始化类时，如果父类没有初始化，则初始化父类

什么情况不会初始化？

- 子类调用父类的静态变量，子类不会被初始化。只有父类被初始化
- 通过数组定义来引用类，不会初始化
- 访问类的常量（static final），不会初始化类

Demo

```
class SingleTon {
    private static SingleTon singleTon = new SingleTon();
    public static int count1;
    public static int count2 = 0;
 
    private SingleTon() {
        count1++;
        count2++;
    }
 
    public static SingleTon getInstance() {
        return singleTon;
    }
}
 
public class Test {
    public static void main(String[] args) {
        SingleTon singleTon = SingleTon.getInstance();
        System.out.println("count1=" + singleTon.count1);
        System.out.println("count2=" + singleTon.count2);
    }
}
```

Answer

count1=1
count2=0

[https://blog.csdn.net/ns_code/article/details/17881581](https://blog.csdn.net/ns_code/article/details/17881581)

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