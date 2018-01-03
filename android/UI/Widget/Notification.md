# setFullScreenIntent自动执行PendingIntent的问题

`mBuilder.setFullScreenIntent(pendingIntent, true);`

上述代码，当通知来临时，会自动执行pendingIntent

stackoverflow上的解决方案：

`mBuilder.setFullScreenIntent(null, true);`

但是这样会导致悬浮通知不显示

# 点击后消失

## 方法一

Notification对象，设置一个flags，notification.flags |= Notification.FLAG_AUTO_CANCEL;(一定要加这个"|"，不然没效果)

## 方法二

builder.setAutoCancel(true);

# 创建悬浮通知

Android 5.0 以上，使用`setFullScreenIntent`

```
Intent intent = new Intent(context, MainActivity.class);
PendingIntent pendingIntent = PendingIntent.getActivity(
        context, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT);
mBuilder.setContentIntent(pendingIntent);

mBuilder.setFullScreenIntent(pendingIntent, true);
```

# 基本使用

```
// 第一步：创建Builder
NotificationCompat.Builder mBuilder = new NotificationCompat.Builder(context)
        .setSmallIcon(R.drawable.ic_launcher_app)
        .setContentTitle("My notification")
        .setContentText("Hello World!");

// 第二步：创建PendingIntent（该步骤非必须）
Intent intent = new Intent(context, MainActivity.class);
PendingIntent pendingIntent = PendingIntent.getActivity(
        context, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT);
mBuilder.setContentIntent(pendingIntent);

// 第三步：创建Notification
Notification notification = mBuilder.build();

// 第四步：发送通知
int mNotificationId = 100;
NotificationManager notificationManager = (NotificationManager) context.getSystemService(NOTIFICATION_SERVICE);
if (notificationManager == null) return;
notificationManager.notify(mNotificationId, notification);
```

# 参考资料

- [全面了解Android Notification](https://www.jianshu.com/p/22e27a639787)
- [Android悬浮通知无效的问题](http://blog.csdn.net/firedancer0089/article/details/72866589)