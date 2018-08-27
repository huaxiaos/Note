# 简介

Android多媒体文件管理简单来说，有三个部分

- 扫描：MediaScannerService
- 存储：MediaProvider
- 查询：MediaStore

# MediaScannerService

> 值得一提的是，MediaScannerService这类文件，AndroidStudio中不能直接search到，需要去官网查询其源码，这里推荐一个在线查看Android源码的链接[http://androidxref.com/](http://androidxref.com/)

```
public class MediaScannerService extends Service implements Runnable

// 核心方法
private void scan(String[] directories, String volumeName) {
    ...

    try {
        ...

        try {
            ...

            MediaScanner scanner = createMediaScanner();
            scanner.scanDirectories(directories, volumeName);
        } catch (Exception e) {
            ...
        }

        ...

    } finally {
        ...
    }
}

```
应用级Service，用于媒体文件扫描

scan是其核心方法，从源码中可以看出，最终是调用了MediaScanner的scanDirectories方法

# MediaScannerReceiver

与MediaScannerService配套的BroastReceiver

```
@Override
public void onReceive(Context context, Intent intent) {
    final String action = intent.getAction();
    final Uri uri = intent.getData();
    if (Intent.ACTION_BOOT_COMPLETED.equals(action)) {
        // Scan both internal and external storage
        scan(context, MediaProvider.INTERNAL_VOLUME);
        scan(context, MediaProvider.EXTERNAL_VOLUME);

    } else {
        if (uri.getScheme().equals("file")) {
        	// 省略...
            if (Intent.ACTION_MEDIA_MOUNTED.equals(action)) {
                // scan whenever any volume is mounted
                scan(context, MediaProvider.EXTERNAL_VOLUME);
            } else if (Intent.ACTION_MEDIA_SCANNER_SCAN_FILE.equals(action) &&
                    path != null && path.startsWith(externalStoragePath + "/")) {
                scanFile(context, path);
            }
        }
    }
}
```

从onReceive方法中可以看出，MediaScannerReceiver执行scan的时机有三种

- 启动完毕，扫描内部存储和外部存储
- sdcard挂载完毕，扫描外部存储
- 扫描单个文件

# MediaScanner

具体执行scan动作的类

# MediaScannerProvider

ContentProvider用于保存查询结果，结果由数据库保存

以外部存储为例，数据库的路径为/data/data/com.android.providers.media/databases/external.db

# MediaStore

MediaStore是面向应用的入口，作用了是为了便于用户查询已经扫描过的数据

MediaStore中的资源有四类

- MediaStore.Files
- MediaStore.Audio
- MediaStore.Images
- MediaStore.Video

# 媒体库查询

## 核心方法

```
public final @Nullable Cursor query(@RequiresPermission.Read 
		@NonNull Uri uri, 
		@Nullable String[] projection, 
		@Nullable String selection, 
		@Nullable String[] selectionArgs, 
		@Nullable String sortOrder)
```

## Uri uri

```
Uri uri = MediaStore.Video.Media.INTERNAL_CONTENT_URI;
Uri uri = MediaStore.Video.Media.EXTERNAL_CONTENT_URI;
```
Uri总共有两种，分别对应内部存储和外部存储

## String[] projection

```
String[] mediaColumns = {
	        MediaStore.Video.Media.DATA,
	        MediaStore.Video.Media.SIZE,
	        MediaStore.Video.Media.DATE_MODIFIED,
	        MediaStore.Video.Media.DURATION};
```

用于指定查询后，返回给用户的媒体信息

## String selection & String[] selectionArgs

这两个是可选项，用户可以通过这两个参数，定制化查询条件

```
String selection = MediaStore.Video.Media.MIME_TYPE + "=? or "
            + MediaStore.Video.Media.MIME_TYPE + "=? or "
            + MediaStore.Video.Media.MIME_TYPE + "=? or "
            + MediaStore.Video.Media.MIME_TYPE + "=? or "
            + MediaStore.Video.Media.MIME_TYPE + "=?"

String[] selectionArgs = new String[]{"video/mp4", "video/avi", "video/quicktime", "video/webm", "video/x-ms-wmv"}
```

需要注意的是，selection中的条件和selectionArgs数组中的类型，需要是一一对应的，否则会发生异常

## String sortOrder

查询的排序方式

## .nomedia

文件夹中新建一个.nomedia的空文件，会屏蔽掉系统默认的媒体库扫描。遇到带有该文件的文件夹，只能通过文件遍历的方式进行扫描

## MIME_TYPE

需要注意的是MIMETYPE不一定和文件的扩展名一致，比如mov格式的文件，MIMETYPE为video/quicktime等等

# 新增视频资源并通知手机相册

## 基本方法

### 方法一：发送广播，通知媒体库刷新

```
Intent intent = new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE);
intent.setData(Uri.fromFile(new File(filePath)));
context.sendBroadcast(intent);
```

### 方法二：插入数据库

也可以使用直接插入媒体库数据库的方法来实现（但这两种方法不能同时使用，会导致媒体库中出现重复的数据）

```
ContentValues values = new ContentValues(4);
values.put(MediaStore.Video.Media.TITLE, "");
values.put(MediaStore.Video.Media.MIME_TYPE, minetype);
values.put(MediaStore.Video.Media.DATA, path);
values.put(MediaStore.Video.Media.DURATION, duration_int);
context.getContentResolver().insert(MediaStore.Video.Media.EXTERNAL_CONTENT_URI, values);
```

插入数据库的这种方法，这边博文中有详细介绍[blog](http://blog.csdn.net/chendong_/article/details/52290329)

### 方法三：MediaScannerConnection.scanFile

```
public static void insert(Context context, String[] paths, String[] types) {
    MediaScannerConnection.scanFile(
            context,
            paths,
            types,
            new MediaScannerConnection.OnScanCompletedListener() {
                @Override
                public void onScanCompleted(String path, Uri uri) {
                    LogUtils.i(TAG, "insert onScanCompleted path " + path + " uri " + uri);
                }
            });
}
```

## 机型适配

部分机型上述两种方法都不能实现实时刷新（只能重启手机才能看到效果），考虑可能是第三方ROM的改动导致的，将目录设置为`/sdcard/DCIM/Camera`可以解决大部分机型的问题，但vivo和魅族的手机仍旧不能刷新，vivo的地址应该为`/sdcard/相机`，魅族的地址应该为`/sdcard/DCIM/Video`

下面给出一个获取输出地址的方法

```
public static String getVideoOutputDir() {
    String sdcardPath = Environment.getExternalStorageDirectory().getPath() + File.separator;
    String brand = SystemUtil.getDeviceBrand();

    if (TextUtils.isEmpty(brand)) {
        return sdcardPath + "DCIM" + File.separator + "Camera";
    }

    brand = brand.toLowerCase();
    if (brand.contains("vivo")) {
        return sdcardPath + "相机";
    } else if (brand.contains("meizu")) {
        return sdcardPath + "DCIM" + File.separator + "Video";
    }

    return sdcardPath + "DCIM" + File.separator + "Camera";
}

public static String getDeviceBrand() {
    return android.os.Build.BRAND;
}
```

> 值得一提的是，如果想删除相册中的某个资源文件，OPPO的某些机型（ColorOS 3.1系统），会弹窗提示：删除图库中的照片和视频权限申请，需要用户手动去确认删除。这个没有特别好的办法

# 参考资料

- [https://my.oschina.net/youranhongcha/blog/787223](https://my.oschina.net/youranhongcha/blog/787223)
- [https://droidyue.com/blog/2014/07/12/scan-media-files-in-android-chinese-edition/](https://droidyue.com/blog/2014/07/12/scan-media-files-in-android-chinese-edition/)
