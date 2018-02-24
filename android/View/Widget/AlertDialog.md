# 自定义AlertDialog去掉默认背景

```
mAlertDialog = new AlertDialog.Builder(this).create();
mAlertDialog.setCancelable(false);
mAlertDialog.show();

Window window = mAlertDialog.getWindow();
if (window == null) return;
window.setContentView(R.layout.pause_alert);
window.setBackgroundDrawable(new ColorDrawable(android.graphics.Color.TRANSPARENT));
window.setGravity(Gravity.CENTER);
```

# AlertDialog和Handler

在子线程中使用AlertDialog，会报如下错误

`java.lang.RuntimeException: Can't create handler inside thread that has not called Looper.prepare()`

说明AlertDialog中使用了Handler，通过查看源码找到Handler

```
public AlertDialog show() {
    final AlertDialog dialog = create();
    dialog.show();
    return dialog;
}
```

```
public void show() {
    ...
    sendShowMessage();
}
```

```
private void sendShowMessage() {
    if (mShowMessage != null) {
        // Obtain a new message so this dialog can be re-used
        Message.obtain(mShowMessage).sendToTarget();
    }
}
```

```
/**
 * Sends this Message to the Handler specified by {@link #getTarget}.
 * Throws a null pointer exception if this field has not been set.
 */
public void sendToTarget() {
    target.sendMessage(this);
}
```

这里的target就是Handler

# 基本使用

```
new AlertDialog.Builder(mContext)
    .setTitle("通知")
    .setMessage(String.format("该视频%sMB，您正在使用非WIFI网络，下载将产生流量费用。是否继续下载？", length))
    .setPositiveButton("继续下载", new DialogInterface.OnClickListener() {
        @Override
        public void onClick(DialogInterface dialog, int which) {
            dialog.dismiss();
        }
    })
    .setNegativeButton("取消", null)
    .show();
```

