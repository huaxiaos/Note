---
title: Android Studio 多渠道打包中针对不同渠道不同应用名称的处理方法
date: 2016-05-04 19:48:39
tags: [Android,Android Studio,AndroidManifest,多渠道打包]
categories: Android
---

>版权声明：本文为博主原创文章，未经博主允许不得转载。

Android Studio 多渠道打包的文章相信大家看到的应该很多了。推荐阅读小单的这篇，非常不错，很齐全：http://blog.csdn.net/catoop/article/details/50435431


但是很多公司可能会有这种更细微的需求：不同的渠道号要对应不同的应用名字。
比如渠道号是小米，应用的名字叫“小米渠道”，渠道号是华为，应用的名字叫“华为渠道”等等。

网上有很多对这个问题的处理方法：
- **方法一**用shell脚本修改AndroidManifest
- **方法二**增加N个manifest.xml文件，文件中声明应用的名字。

但这些方法用起来都比较麻烦，还容易出错。

-------------------

现在有一种简便的方法处理这个问题，方法如下：

1. 对AndroidManifest中`<application>`标签下的`android:label`进行赋值：android:label="${APP_NAME}"

2. 这一步很关键，只做了第一步后，在编译的时候会报错，这里需要将android:label声明为可以修改的，也就是在`<application>`标签中增加：`tools:replace="android:label"`（这里用到了Manifest Merge，详情请参见[官方文档](http://tools.android.com/tech-docs/new-build-system/user-guide/manifest-merger)）
3. 最后根据网上的教程，完成后续build.gradle文件的配置即可。

-------------------

最后附上核心代码片段，祝各位以后更加安心的一键打包。


``` xml
<application  
    android:name="XXXApp"  
    android:allowBackup="true"  
    android:icon="@mipmap/ic_launcher"  
    android:label="${APP_NAME}"  
    android:theme="@style/AppTheme"  
    tools:replace="android:label">
```

``` gradle
android {

    ……

    // 渠道
    productFlavors {
        huawei {
            manifestPlaceholders = [APP_NAME: "华为Name",
                                    APP_CHANNEL: "huawei"]
        }

        xiaomi {
            manifestPlaceholders = [APP_NAME: "小米Name",
                                    APP_CHANNEL: "xiaomi"]
        }
    }
}
```
