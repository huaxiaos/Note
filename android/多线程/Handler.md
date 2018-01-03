# Bitmap设置Background变小的问题

代码片段

```
BitmapDrawable bd=new BitmapDrawable(bitmap);
llLeft.setBackgroundDrawable(bd);
```

这样会导致实际看到的背景远小于Bitmap实际的大小

原因是没有指定系统分辨率，需要setTargetDensity

```
BitmapDrawable bd=new BitmapDrawable(bitmap[0]);

DisplayMetrics dm = new DisplayMetrics();
getWindowManager().getDefaultDisplay().getMetrics(dm);
bd.setTargetDensity(dm);

llLeft.setBackgroundDrawable(bd);
```