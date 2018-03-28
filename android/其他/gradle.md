# APT&annotationProcessor&android-apt&Provided

## APT

APT(Annotation Processing Tool)是一种处理注解的工具,它对源代码文件进行检测找出其中的Annotation，根据注解自动生成代码。 Annotation处理器在处理Annotation时可以根据源文件中的Annotation生成额外的源文件和其它的文件(文件具体内容由Annotation处理器的编写者决定),APT还会编译生成的源文件和原来的源文件，将它们一起生成class文件

##annotationProcessor

annotationProcessor是APT工具中的一种，他是google开发的内置框架，不需要引入，可以直接在build.gradle文件中使用

## android-apt

android-apt是由一位开发者自己开发的apt框架，随着Android Gradle 插件 2.2 版本的发布，Android Gradle 插件提供了名为 annotationProcessor 的功能来完全代替 android-apt

##Provided

Provided 虽然也是编译时执行，最终不会打包到apk中，Provided是间接的得到了依赖的Library，运行的时候必须要保证这个Library的存在，否则就会崩溃，起到了避免依赖重复资源的作用

[你必须知道的APT、annotationProcessor、android-apt、Provided、自定义注解](https://blog.csdn.net/xx326664162/article/details/68490059)



# 参考资料

- [全面理解Gradle - 执行时序](http://blog.csdn.net/singwhatiwanna/article/details/78797506)