# 颜色渐变

使用gradient，例如实现一个颜色渐变的圆角背景图

```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">

    <corners android:radius="4dp" />

    <gradient
        android:angle="0"
        android:endColor="#5E52FE"
        android:startColor="#5289FF" />

</shape>
```

# 圆

```
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">

   <solid 
       android:color="#666666"/>

   <size 
       android:width="120dp"
        android:height="120dp"/>
</shape>
```