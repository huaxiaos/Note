# 行间距

两列之间的间距

`android:horizontalSpacing`

两行之间的间距

`android:verticalSpacing`

# Example

```
<GridView
    android:id="@+id/music_gv"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:clipToPadding="false"
    android:gravity="center_horizontal"
    android:numColumns="4"
    android:paddingBottom="60dp"
    android:paddingEnd="16dp"
    android:paddingStart="16dp"
    android:paddingTop="15.5dp"
    android:scrollbars="none"
    android:verticalSpacing="17.5dp" />
```