# 背景透明

```
@Nullable
@Override
public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, Bundle savedInstanceState) {
    if (getDialog().getWindow() != null) {
        getDialog().getWindow().setBackgroundDrawable(new ColorDrawable(0));
    }
    return inflater.inflate(R.layout.dialog_fragment_low_version, container, false);
}
```

# 全屏

```
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setStyle(DialogFragment.STYLE_NORMAL, android.R.style.Theme_Black_NoTitleBar_Fullscreen);
}
```