# BottomSheetDialog

## 全屏显示

BottomSheetDialog继承自AppCompatDialog，需要重写BottomSheetDialog，并做特殊设置，需要注意的是，下面代码的执行顺序不能改变

```
public class PCMsgDeleteDialog extends BottomSheetDialog {
    public PCMsgDeleteDialog(@NonNull Context context) {
        super(context);
    }

    public PCMsgDeleteDialog(@NonNull Context context, int theme) {
        super(context, theme);
    }

    protected PCMsgDeleteDialog(@NonNull Context context, boolean cancelable, OnCancelListener cancelListener) {
        super(context, cancelable, cancelListener);
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Window window = getWindow();
        if (window == null) return;

        requestWindowFeature(Window.FEATURE_NO_TITLE);

        int flag = WindowManager.LayoutParams.FLAG_FULLSCREEN;
        window.setFlags(flag, flag);
        
        setContentView(R.layout.dialog_fragment_pc_delete);

        window.setLayout(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT);
    }
    
}
```

## 背景透明

### 方法一

```
bottomSheetDialog.getWindow().findViewById(R.id.design_bottom_sheet).setBackgroundResource(android.R.color.transparent);
```

### 方法二

自定义style

```
<style name="BottomSheetDialogStyle" parent="Theme.Design.BottomSheetDialog">
    <item name="android:colorBackground">@android:color/transparent</item>
</style>
```

```
BottomSheetDialog dialog = new BottomSheetDialog(mContext, R.style. BottomSheetDialogStyle);
```

## 基本使用

```
BottomSheetDialog dialog = new BottomSheetDialog(mContext);
dialog.setContentView(view);
dialog.show();
```

# BottomSheet

# BottomSheetFragment