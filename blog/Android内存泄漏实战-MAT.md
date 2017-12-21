---
title: Android内存泄漏实战-MAT
date: 2016-06-01 11:03:28
tags: [Android,Android Studio,MAT,内存泄漏]
categories: Android
---

> 本文是实际项目中的真实案例，部分截图中马赛克了包名信息

# 起因
日常coding的过程中，一般很少会特意的去做内存泄漏的检查。项目上线后，出现了一个bug，经过调查发现，某个Activity中的成员变量被重置成了初始值，检查完所有代码后得出结论，是由于某种情况下的内存回收导致的重置。但内存回收的原因一直没有查明，之前一直认为是低配置的手机的问题，直到高配的手机也出现这个问题后，我考虑到可能是由于内存泄漏引起的Activity回收。

# 工具选择
有以下三种工具供我们选择，排查代码中出现的内存泄漏问题。
- **MAT** ：Memory Analyzer Tool，一个基于Eclipse的内存分析工具，是一个快速、功能丰富的JAVA heap分析工具。有独立使用的版本（[下载地址](http://www.eclipse.org/mat/)）。
- **Android Studio** ：Android Studio 1.1及更高的版本，内置了内存泄漏检测的工具。
- **LeakCanary** ：Square开源的内存泄漏检查库，可以直接集成到代码中。（[Github地址](https://github.com/square/leakcanary)）

如果不知道那个页面出现内存泄漏，可以直接集成LeakCanary，将App跑一遍，LeakCanary会自动通知你哪里出现了内存泄漏，非常方便。如果确定了某个页面出现问题，想详细查询内存泄漏的相关信息的话，还是建议使用MAT工具来分析，MAT能给出非常全面且直观的数据。
最终，我决定使用Android Studio和MAT来分析内存泄漏问题。

# 使用Android Studio导出MAT所能识别的hprof文件

- 打开Android Monitor中的Monitors标签，我们可以实时监控到App的Memory、CPU、NetWork、GPU使用状态。
- 打开App中出现内存泄漏问题的页面（比如我的项目中是KnowledgePointActivity），退出该页面，然后在Memory中点击“Initiate GC”，手动执行GC操作。
> **提示：**这里的手动GC操作不能省，否则会导致有错误数据误导你，后面会有实例展示这个“坑”。

- 反复执行上述操作几遍。
- 点击“Dump Java Heap”，Android Studio（下称AS）会自动生成hprof文件，并自带的内存泄漏分析。顺便讲一下，AS自带的内存泄漏分析也非常方便，能自动分析哪个页面出现内存泄漏问题。只需要点击“Analyzer Tasks”页签中的“Perform Analysis”按钮即可。
- AS自动生成的hprof并不能被MAT识别，需要做一次转换。在“Captures”页签中可以找到我们生成的hprof文件，右键该文件，点击“Export to standard .hprof”，就可以生成MAT能识别的文件了。

# 使用MAT分析内存泄漏

第一次打开hprof文件会有一些初始化操作，只要按照默认配置下一步即可。
生成报告后，打开OverView界面，并点击“Histogram”（列出内存中的对象，对象的个数以及大小），从这些对象中寻找我们想找到的内存泄漏路径。
有如下两种方法进行定位：

## 方法一：使用OQL执行语句查询
- 点击“OQL”图标。
- 在窗口输入`select * from instanceof android.app.Activity`，然后点击“!”按钮。

## 方法二：Histgram中进行逐步筛选
- 打开Histgram页面，第一行我们可以通过正则表达式来查到我们想定位的类（如输入“Activity”，则可以筛选出所有的Activity）
- 找到我们的指定页面（本文中是KnowledgePointActivity），右键选择List objects → with incoming reference

通过上述两种方法，我们就能找到该页面下所有没有被回收的对象。如下图所示：

![](http://7xuvhf.com1.z0.glb.clouddn.com/knowledgepointactivity_list.png)

选择其中任意一条记录，右键选择Path to GC Roots → exclude weak/soft references，新页面中展示的是列出所有强引用的GC Roots结点，这里就能很直观的发现是哪里强引用了Activity对象，导致无法回收。

如下图所示，mBtnList对象强持有了Activity的引用。
![](http://7xuvhf.com1.z0.glb.clouddn.com/mBtnList.png)


# 回归代码中发现问题

那么上面我们找到的mBtnList具体为什么会强持有Activity的引用呢？
检查代码后发现，该对象的相关代码如下：
``` java
private static ArrayList<HashMap<String, Object>> mBtnList;

HashMap<String, Object> map = new HashMap<>();
map.put("type", "string");
map.put("view", (View) child);
mBtnList.add(map);
```
我这里错误的将持有Activity引用的View对象加入了一个static的对象中，这样就直接导致了在Activity销毁时无法完成对该对象的回收。

这里将mBtnList改为非静态的对象即可解决问题，修改完后再重新执行上述步骤，检查内存泄漏，发现问题已经成功解决了。

![](http://7xuvhf.com1.z0.glb.clouddn.com/knowledgepointactivity_success.png)

> **提示：**静态有风险，使用需谨慎。

# 内存泄漏检查过程中可能出现的“坑”

- 在生成hprof文件时，如果没有手动执行GC操作，会产生一些“错误”的数据误导你，如下图：

![](http://7xuvhf.com1.z0.glb.clouddn.com/mOwningView.png)

理论上，运行10遍Activity，如果不执行GC操作，就会有10个上图中的对象出现，这些对象（虚引用的对象？）在GC时会被销毁，不属于内存泄漏的范畴。
