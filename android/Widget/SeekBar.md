# 自定义SeekBar

## 自定义颜色渐变进度条

在`layer-list`中，相应的属性，增加`gradient`即可

```
<item android:id="@android:id/progress">
    <clip>
        <shape>
            <corners android:radius="1dp" />

            <gradient
                android:angle="0"
                android:endColor="#3968FF"
                android:startColor="#5289FF" />
        </shape>
    </clip>
</item>
```

## 自定义进度条

需要设置以下三个属性：

- `android:id/background`：进度条的整体背景颜色
- `android:id/secondaryProgress`：二级进度条的颜色
- `android:id/progress`：一级进度条的颜色，即进度条当前已经滑过面积的颜色

对应xml里面的`android:progressDrawable`属性，新建`layer-list`，将其设置给该属性即可

需要注意的是，SeekBar的宽高，需要设置`android:maxHeight`属性，直接在layer-list中设置，不会生效

```
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@android:id/background">
        <shape>
            <corners android:radius="1dp" />
            <solid android:color="#383838" />
        </shape>
    </item>

    <item android:id="@android:id/secondaryProgress">
        <clip>
            <shape>
                <corners android:radius="1dp" />
                <solid android:color="#383838" />
            </shape>
        </clip>
    </item>

    <item android:id="@android:id/progress">
        <clip>
            <shape>
                <corners android:radius="1dp" />
                <solid android:color="#d7c60e" />
            </shape>
        </clip>
    </item>
</layer-list>
```

## 自定义滑块

对应`android:thumb`属性，将drawable设置给该属性即可

```
android:thumb="@drawable/speed_thumb"
```

# 基本使用

```
<SeekBar
	android:id="@+id/sb_volume"
	android:layout_width="match_parent"
	android:layout_height="55dp"
	android:progress="100" />
```