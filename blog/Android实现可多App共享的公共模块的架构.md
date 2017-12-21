---
title: Android实现可多App共享的公共模块的架构
date: 2016-06-25 22:07:46
tags: [Android,架构]
---

> 版权声明：本文为博主原创文章，未经博主允许不得转载。

## 背景

随着公司业务的不断发展，公司多个App之间或多或少会有一些公共模块，这些公共模块并不适合独立成App来实现，那么如果能App之间共享，一处更新多App同步更新，那么无疑会减少非常多的开发成本。

## 公共模块的拆分

需要根据业务逻辑进行拆分，并且需要将业务无关的模块独立出来，如将一个项目拆分为基本库、登录注册库、业务库A、业务库B等等。

- 基本库中主要包含一些框架类和工具类的模块，如网络请求框架、数据库框架、自定义View、工具类等等。基本库完全与业务逻辑无关，不依赖于任何其他库，且可以被所有的其他库所依赖。

- 业务库是根据业务逻辑拆分出来的，是否需要依赖其他库需要根据需要实际处理，理论上，业务库是需要依赖基本库的。

## 项目文件组织结构

根据上面的分析和拆分，我们基本上可以整理出一个项目的文件组织结构了。

![](http://7xuvhf.com1.z0.glb.clouddn.com/Q20160625211909.png)

下面就是如何实现了。

## 如何将一个Library实现App间共享

这是一个难点，Android Studio没有直接生成一个可以APP间共享的Library，默认生成的Library只能供一个App独立调用。

不过我们可以通过修改Gradle的配置来实现共享，步骤如下：

1. 按照默认方式新建一个Library，如Lib_Basic
2. 将Lib_Basic移动到上面整理的文件结构中的Libs目录下
3. 修改Gradle配置：将App_A中根目录下的`setting.gradle`文件做如下修改。

``` gradle
include ':app' , ':lib_basic', 
project(':lib_basic').projectDir = new File(settingsDir, '../Libs/lib_basic')
```

4. 按照同样的方式增加其他Lib，并在其他App中也做类似的修改。

这样，我们就实现了Lib的共享：通过修改`setting.gradle`中对Library的引用路径来实现。

## 后记

看到这里，各位可能会有这样的一个疑问：这样实现App间共享的公共模块后，确实是一处修改其他App同步更新，如果我们并不想让其他App更新呢？

这个问题我们就可以通过Git来解决，将App和各个公共库的Git分离，即App是一个Git repo，其他的库各自是一个Git repo，那么我们就可以通过切换公共库中Git的branch来实现自主控制。

简而言之，App_A可以使用Lib_A的1.0版本和Lib_B的2.0版本，而App_B可以使用Lib_A的2.0版本和Lib_B的3.0版本，等等。

实现方式有多种，可以直接用Git命令行、SourceTree等等，不过，现在有一款叫grgit（[GitHub地址](https://github.com/ajoberstar/grgit)）的Gradle库可以便捷的通过Android Studio的Terminal命令行方式实现。

> 后续我会更新一个关于grgit使用的博文
