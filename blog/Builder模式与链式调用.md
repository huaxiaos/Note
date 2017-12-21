---
title: Builder模式与链式调用
date: 2017-07-17 20:20:20
tags: [Android,Builder模式] 
categories: Android
---

> 版权声明：本文为作者原创，转载请注明出处。

# 前言

众所周知，Builder模式在Android中有着广泛的应用，是一个非常重要的设计模式。然而在实际开发过程中，我一直有这样一个疑问：链式调用貌似也能实现Builder模式所完成的功能，那么为何不简单的使用链式调用来实现呢？

> 为尽快切入正题，概念相关的介绍在本文的最后部分

# AlertDialog

AlertDialog是Android源码中使用Builder模式的经典案例，本文将从AlertDialog入手来进行分析。

AlertDialog的常规使用如下
```
AlertDialog alertDialog = new AlertDialog.Builder(this)
    .setTitle(title)
    .setMessage(msg)
    .setCancelable(false)
    .setPositiveButton(btn,
            new DialogInterface.OnClickListener() {
                @Override
                public void onClick(final DialogInterface dialog, int which) {
                    dialog.dismiss();
                }
            }).show();
```

很明显是一个标准的链式调用，那么问题来了，我能否去掉Builder，而直接new对象来实现呢？类似如下代码
```
AlertDialog alertDialog = new AlertDialog(this)
    .setTitle(title)
    .setMessage(msg)
    .setCancelable(false)
    .setPositiveButton(btn,
            new DialogInterface.OnClickListener() {
                @Override
                public void onClick(final DialogInterface dialog, int which) {
                    dialog.dismiss();
                }
            }).show();
```

请继续看下面的分析

# AlertDialog源码

首先看一下show方法

```
/**
 * Creates an {@link AlertDialog} with the arguments supplied to this
 * builder and immediately displays the dialog.
 * <p>
 * Calling this method is functionally identical to:
 * <pre>
 *     AlertDialog dialog = builder.create();
 *     dialog.show();
 * </pre>
 */
public AlertDialog show() {
    final AlertDialog dialog = create();
    dialog.show();
    return dialog;
}
```

可以看出，AlertDialog对象是通过create()方法来生成的。我们再来看下create()方法

```
/**
 * Creates an {@link AlertDialog} with the arguments supplied to this
 * builder.
 * <p>
 * Calling this method does not display the dialog. If no additional
 * processing is needed, {@link #show()} may be called instead to both
 * create and display the dialog.
 */
public AlertDialog create() {
    // Context has already been wrapped with the appropriate theme.
    final AlertDialog dialog = new AlertDialog(P.mContext, 0, false);
    P.apply(dialog.mAlert);
    dialog.setCancelable(P.mCancelable);
    if (P.mCancelable) {
        dialog.setCanceledOnTouchOutside(true);
    }
    dialog.setOnCancelListener(P.mOnCancelListener);
    dialog.setOnDismissListener(P.mOnDismissListener);
    if (P.mOnKeyListener != null) {
        dialog.setOnKeyListener(P.mOnKeyListener);
    }
    return dialog;
}
```
从中可以看出，在构建AlertDialog时，create()方法中做了大量的工作，如设置默认值、设置监听等等。create()方法控制着AlertDialog对象的产出结果。

show()和create()均是AlertDialog.Builder类中的方法，源码如下

```
public static class Builder {
    // 省略了注释，省略了诸多set方法
    private final AlertController.AlertParams P;

    public Builder(Context context) {
        this(context, resolveDialogTheme(context, 0));
    }

    public Builder(Context context, int themeResId) {
        P = new AlertController.AlertParams(new ContextThemeWrapper(
                context, resolveDialogTheme(context, themeResId)));
    }

    public Context getContext() {
        return P.mContext;
    }

    public Builder setTitle(@StringRes int titleId) {
        P.mTitle = P.mContext.getText(titleId);
        return this;
    }

    public Builder setTitle(CharSequence title) {
        P.mTitle = title;
        return this;
    }

    public AlertDialog create() {
        // Context has already been wrapped with the appropriate theme.
        final AlertDialog dialog = new AlertDialog(P.mContext, 0, false);
        P.apply(dialog.mAlert);
        dialog.setCancelable(P.mCancelable);
        if (P.mCancelable) {
            dialog.setCanceledOnTouchOutside(true);
        }
        dialog.setOnCancelListener(P.mOnCancelListener);
        dialog.setOnDismissListener(P.mOnDismissListener);
        if (P.mOnKeyListener != null) {
            dialog.setOnKeyListener(P.mOnKeyListener);
        }
        return dialog;
    }

    public AlertDialog show() {
        final AlertDialog dialog = create();
        dialog.show();
        return dialog;
    }
}
```

# create()方法的必要性

从上面的分析可以看出，核心方法是create()方法，那么我们能否将create()方法中的执行代码全部放在show()方法中实现呢？或者保留create()方法，但是将Builder类去掉，create()和show()移至AlertDialog类中执行呢？

结论是否定的。

即使某种意义上说，去掉了Builder，移动了方法也是能运行的。但这就与AlertDialog，甚至是Builder模式本身的思想背道而驰了。

抽象一下，create()就好比是“生产制造”步骤，而show()好比是“使用体验”步骤，两者显然是相对较独立的方法，当业务逻辑特别复杂时，将“生产”与“使用”分离开来，就显得尤为重要，既增加了代码的可读性，也增加了代码的可扩展性。

更重要的是，Builder.create()本身是对对象构建的一种“约束”，不对外暴露产品细节，用户可以按需构建，同时，用户也可以通过修改“约束”本身来控制对象的构建结果。

# 与实际相结合

既然Builder模式有这么多的好处，又增加了可扩展性等等，并且读过《Effective Java》的朋友可能知道里面有这样一段建议

> 当你有需要三个以上的构造参数时，使用 Builder 去构造这个对象。写起来可能有点啰嗦但是这样伸缩性和可读性都很好

那么，是不是优先使用Builder模式来完成对象的构建呢？结论也是否定的。

设计模式的使用一定要与实际场景相结合，否则就是过度设计。

当你的使用场景满足如下条件时，可以使用Builder模式

* 相同的方法，不同的执行顺序，产生不同的事件结果；或者方法的调用顺序不同也会产生了不同的作用
* 多个部件或零件，都可以装配到同一个对象中，但是产生的运行结果又不相同
* 需要初始化一个对象特别复杂的对象，这个对象有很多参数，且有默认值
* 

总之一点，当你的实际场景尤其复杂时，Builder模式是你的正确选择之一。

除此之外，简单的实现一个链式调用（或者普通的对象构造）是最好的方案。

# 相关概念介绍

## Builder模式

将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示

## 链式调用

```
public class MsgInfo {

    private long msgId;
    private String type;

    pulic MsgInfo() {
        
    }
    
    public MsgInfo setMsgId(long msgId) {
        this.msgId = msgId;
        return this;
    }
    
    public MsgInfo setType(String type) {
        this.type = type;
        return this;
    }
    
    public void show() {
        // do something
    }

}
```

```
MsgInfo msgInfo = new MsgInfo().setMsgId(99).setType("A").show();
```

# 参考资料

* https://juejin.im/post/58f630a544d904006c0eedc9
* http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2013/0527/1299.html
* http://www.jianshu.com/p/cb9cd6a8dd51
* 《Effective Java》
