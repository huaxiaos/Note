# Bitmap复用

Glide内部有一个BitmapPool，被memoryCache中不用的Bitmap会先放在BitmapPool中

用户可以自定义BitmapPool，通过GlideModule设置

# 参考资料

https://muzhi1991.gitbooks.io/android-glide-wiki/content/chapter8.html

https://juejin.im/entry/59004a0c61ff4b0066819a15