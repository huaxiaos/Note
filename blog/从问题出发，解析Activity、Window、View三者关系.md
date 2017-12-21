---
title: 从问题出发，解析Activity、Window、View三者关系
date: 2017-08-28 14:04:44
tags: [Android,Window]
categories: Android
---

> 本文为博主原创文章，转载请注明出处。

# 前言

从问题出发，往往能更明确的找到所求。本文将带着一个个的问题，结合源码，逐步解析Activity、Window、View的三者关系。

# 什么地方需要window？

- 一句话总结：有视图的地方就需要window
- Activity、Dialog、Toast...

# PopupWindow和Dialog有什么区别？

两者最根本的区别在于有没有新建一个window，PopupWindow没有新建，而是将view加到DecorView；Dialog是新建了一个window，相当于走了一遍Activity中创建window的流程

PopupWindow的相关源码

```
public PopupWindow(View contentView, int width, int height, boolean focusable) {
    if (contentView != null) {
        mContext = contentView.getContext();
        mWindowManager = (WindowManager) mContext.getSystemService(Context.WINDOW_SERVICE);
    }

    setContentView(contentView);
    setWidth(width);
    setHeight(height);
    setFocusable(focusable);
}
```

```
private void invokePopup(WindowManager.LayoutParams p) {
    if (mContext != null) {
        p.packageName = mContext.getPackageName();
    }

    final PopupDecorView decorView = mDecorView;
    decorView.setFitsSystemWindows(mLayoutInsetDecor);

    setLayoutDirectionFromAnchor();

    mWindowManager.addView(decorView, p);

    if (mEnterTransition != null) {
        decorView.requestEnterTransition(mEnterTransition);
    }
}
```

从源码中可以看出，PopupWindow最终是执行了mWindowManager.addView方法，全程没有新建window

Dialog的相关源码如下

```
Dialog(@NonNull Context context, @StyleRes int themeResId, boolean createContextThemeWrapper) {
    if (createContextThemeWrapper) {
        if (themeResId == 0) {
            final TypedValue outValue = new TypedValue();
            context.getTheme().resolveAttribute(R.attr.dialogTheme, outValue, true);
            themeResId = outValue.resourceId;
        }
        mContext = new ContextThemeWrapper(context, themeResId);
    } else {
        mContext = context;
    }

    mWindowManager = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);

    final Window w = new PhoneWindow(mContext);
    mWindow = w;
    w.setCallback(this);
    w.setOnWindowDismissedCallback(this);
    w.setWindowManager(mWindowManager, null, null);
    w.setGravity(Gravity.CENTER);

    mListenersHandler = new ListenersHandler(this);
}
```

很明显，我们看到Dialog执行了window的创建：final Window w = new PhoneWindow(mContext);

# 一句话概括三者的基本关系？

Activity中展示视图元素通过window来实现，window可以理解为一个容器，盛放着一个个的view，用来执行具体的展示工作

# 各自是在何时被创建的？

以Activity举例

## Activity实例的创建

ActivityThread中执行performLaunchActivity，从而生成了Activity的实例

```
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
    ...
    Activity activity = null;
    try {
        java.lang.ClassLoader cl = r.packageInfo.getClassLoader();
        activity = mInstrumentation.newActivity(
                cl, component.getClassName(), r.intent);
        ...
    } catch (Exception e) {
        ...
    }
    
    try {
        ...
        if (activity != null) {
            ...
            activity.attach(appContext, this, getInstrumentation(), r.token,
                    r.ident, app, r.intent, r.activityInfo, title, r.parent,
                    r.embeddedID, r.lastNonConfigurationInstances, config,
                    r.referrer, r.voiceInteractor);
            ...
        }
        ...
    } catch (SuperNotCalledException e) {
        throw e;
    } catch (Exception e) {
        ...
    }

    return activity;
}
```

## Activity中Window的创建

从上面的performLaunchActivity可以看出，在创建Activity实例的同时，会调用Activity的内部方法attach

在该方法中完成window的初始化

```
final void attach(Context context, ActivityThread aThread,
        Instrumentation instr, IBinder token, int ident,
        Application application, Intent intent, ActivityInfo info,
        CharSequence title, Activity parent, String id,
        NonConfigurationInstances lastNonConfigurationInstances,
        Configuration config, String referrer, IVoiceInteractor voiceInteractor) {
    ...

    mWindow = new PhoneWindow(this);
    mWindow.setCallback(this);
    
    ...
    
    mWindow.setWindowManager(
            (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
            mToken, mComponent.flattenToString(),
            (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);
    ...
}
```
## DecorView的创建

- 用户执行Activity的setContentView方法，内部是调用PhoneWindow的setContentView方法，在PhoneWindow中完成DecorView的创建
- 流程
    - Activity中的setContentView
    - PhoneWindow中的setContentView
    - PhoneWindow中的installDecor

```
public void setContentView(@LayoutRes int layoutResID) {
    getWindow().setContentView(layoutResID);
    initWindowDecorActionBar();
}
```

```
@Override
public void setContentView(int layoutResID) {
    ...
    if (mContentParent == null) {
        installDecor();
    } else if (!hasFeature(FEATURE_CONTENT_TRANSITIONS)) {
        mContentParent.removeAllViews();
    }
    ...
}
```

```
private void installDecor() {
    if (mDecor == null) {
        mDecor = generateDecor();
        mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
        mDecor.setIsRootNamespace(true);
        if (!mInvalidatePanelMenuPosted && mInvalidatePanelMenuFeatures != 0) {
            mDecor.postOnAnimation(mInvalidatePanelMenuRunnable);
        }
    }
    ...
}
```

# Activity中的mDecor和Window里面的mDecor有什么关系？

- 两者指向同一个对象，都是DecorView
- Activity中的mDecor是通过ActivityThread中的handleResumeActivity方法来赋值的

```
final void handleResumeActivity(IBinder token,
        boolean clearHide, boolean isForward, boolean reallyResume) {
    ...
    if (r != null) {
        ...

        if (r.window == null && !a.mFinished && willBeVisible) {
            r.window = r.activity.getWindow();
            View decor = r.window.getDecorView();
            ...
            a.mDecor = decor;
            ...
        } else if (!willBeVisible) {
            ...
        }
        ...
    } else {
        ...
    }
}
```

# ViewRoot是什么？ViewRootImpl又是什么？

- ViewRoot的实现类是ViewRootImpl，是WindowManagerService和DecorView的纽带
- ViewRoot不是View的根节点
- View的绘制是从ViewRootImpl的performTraversals方法开始的

# ViewRootImpl何时创建？

当window被装进WindowManager时，完成ViewRootImpl的创建，最终是通过WindowManagerGlobal.addView方法中进行创建的

```
public void addView(View view, ViewGroup.LayoutParams params, Display display, Window parentWindow) {
    ...
    root = new ViewRootImpl(view.getContext(), display);

    view.setLayoutParams(wparams);

    mViews.add(view);
    mRoots.add(root);
    mParams.add(wparams);
    ...
    try {
        root.setView(view, wparams, panelParentView);
    } catch (RuntimeException e) {
      ...
    }
    ...
}
```

# Activity中的Window何时被装进WindowManager？

- 发生在Activity的onResume阶段
- 执行顺序
    - ActivityThread中的handleResumeActivity
    - Activity中的makeVisible

```
final void handleResumeActivity(IBinder token, boolean clearHide, boolean isForward, boolean reallyResume) {
    ...
    if (r != null) {
        ...
        // The window is now visible if it has been added, we are not
        // simply finishing, and we are not starting another activity.
        if (!r.activity.mFinished && willBeVisible
                && r.activity.mDecor != null && !r.hideForNow) {
            ...
            if (r.activity.mVisibleFromClient) {
                r.activity.makeVisible();
            }
        }
        ...
    } else {
        ...
    }
}
```

```
void makeVisible() {
    if (!mWindowAdded) {
        ViewManager wm = getWindowManager();
        wm.addView(mDecor, getWindow().getAttributes());
        mWindowAdded = true;
    }
    mDecor.setVisibility(View.VISIBLE);
}
```

# 什么是WindowManagerGlobal？WindowManager、WindowManagerGlobal、WindowManagerImpl、WindowManagerPolicy有什么区别？

- WindowManagerGlobal中实现了ViewManager中addView、removeView、updateViewLayout这个三个view相关的方法
- WindowManager是一个接口类，对应的实现类是WindowManagerImpl，该实现类中持有mGlobal对象，这个mGlobal对象就是WindowManagerGlobal，具体的实现交给了WindowManagerGlobal，WindowManagerImpl相当于WindowManagerGlobal的代理类
- WindowManagerPolicy提供所有和UI有关的接口，PhoneWindowManager实现了该接口。需要注意的是PhoneWindowManager并不是WindowManager的子类。WindowManagerService中持有mPolicy对象，这个mPolicy就是PhoneWindowManager
    ```
    public class WindowManagerService extends IWindowManager.Stub implements Watchdog.Monitor, WindowManagerPolicy.WindowManagerFuncs {  
        ... 
        final WindowManagerPolicy mPolicy = new PhoneWindowManager();  
        ...
    }
    ```
- 下面是Android视图框架图，可以直观的看到window相关的整体设计

![image](http://7xuvhf.com1.z0.glb.clouddn.com/Fn4thXempKDxNRbFlpxTRrMZn-n6)

# Window的作用是什么？

- 实现了Activity与View的解耦，Activity将视图的全部工作都交给Window来处理
- WindowManager支持对多条View链的管理

# Dialog为什么不能使用Application的Context

- Dialog窗口类型是TYPE_APPLICATION，与Activity一样
- TYPE_APPLICATION要求Token不能为null，Application没有AppWindowToken

源码分析如下

```
Dialog(@NonNull Context context, @StyleRes int themeResId, boolean createContextThemeWrapper) {
    ...
    mWindowManager = (WindowManager) context.getSystemService(Context.WINDOW_SERVICE);
    ...
    w.setWindowManager(mWindowManager, null, null);
    ...
}
```

```
public void setWindowManager(WindowManager wm, IBinder appToken, String appName)
```

上面是Dialog的初始化方法，注意setWindowManager方法中，第二个参数是appToken，Dialog初始化默认直接传入的是null，这是与Activity的一个显著区别。此时又会有一个新的疑问，那无论Context是来自Application还是Activity，都是传入null，那为什么Application会报错呢。

其实，上面的getSystemService也是问题的原因之一。

我们先看一下Activity中的getSystemService方法

```
@Override
public Object getSystemService(@ServiceName @NonNull String name) {
    if (getBaseContext() == null) {
        throw new IllegalStateException(
                "System services not available to Activities before onCreate()");
    }

    if (WINDOW_SERVICE.equals(name)) {
        return mWindowManager;
    } else if (SEARCH_SERVICE.equals(name)) {
        ensureSearchManager();
        return mSearchManager;
    }
    return super.getSystemService(name);
}
```

Activity中重写了getSystemService方法，按照Dialog中初始化的代码分析，此时返回的是当前Activity的mWindowManager，这个mWindowManager中存在Activity的Token

Application中的Context没有重写getSystemService方法，那么在setWindowManager内部，会通过执行createLocalWindowManager方法获得一个WindowManager接口类的实现类WindowManagerImpl，在WindowManagerImpl中mDefaultToken默认也是null

```
public void setWindowManager(WindowManager wm, IBinder appToken, String appName,
        boolean hardwareAccelerated) {
    mAppToken = appToken;
    mAppName = appName;
    mHardwareAccelerated = hardwareAccelerated
            || SystemProperties.getBoolean(PROPERTY_HARDWARE_UI, false);
    if (wm == null) {
        wm = (WindowManager)mContext.getSystemService(Context.WINDOW_SERVICE);
    }
    mWindowManager = ((WindowManagerImpl)wm).createLocalWindowManager(this);
}
```

```
public WindowManagerImpl createLocalWindowManager(Window parentWindow) {
    return new WindowManagerImpl(mDisplay, parentWindow);
}
```

那么为什么appToken是null，就会报错呢？

```
Caused by: android.view.WindowManager$BadTokenException: Unable to add window -- token null is not for an application
    at android.view.ViewRootImpl.setView(ViewRootImpl.java:685)
    at android.view.WindowManagerGlobal.addView(WindowManagerGlobal.java:342)
    at android.view.WindowManagerImpl.addView(WindowManagerImpl.java:93)
    at android.app.Dialog.show(Dialog.java:316)
```

从Logcat中的error信息可以看出，问题是出在了ViewRootImpl的setView方法中

```
public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView) {
    synchronized (this) {
        if (mView == null) {
            ...
            int res; /* = WindowManagerImpl.ADD_OKAY; */
            ...
            try {
                ...
                res = mWindowSession.addToDisplay(mWindow, mSeq, mWindowAttributes,
                        getHostVisibility(), mDisplay.getDisplayId(),
                        mAttachInfo.mContentInsets, mAttachInfo.mStableInsets,
                        mAttachInfo.mOutsets, mInputChannel);
            } catch (RemoteException e) {
                ...
            } finally {
                ...
            }
            ...
            
            if (res < WindowManagerGlobal.ADD_OKAY) {
                ...
                switch (res) {
                    ...
                    case WindowManagerGlobal.ADD_NOT_APP_TOKEN:
                        throw new WindowManager.BadTokenException(
                                "Unable to add window -- token " + attrs.token
                                + " is not for an application");
                    ...
                }
                ...
            }
            ...
        }
    }
}
```

从上述源码中可以看出res在执行完addToDisplay方法后，被置为了一个非法值，从而报错。

mWindowSession是IWindowSession接口类，IWindowSession的实现类是Session，从Session的源码中我们发现，最终是由WindowManagerService中的addWindow方法对res进行了赋值

```
@Override
public int addToDisplay(IWindow window, int seq, WindowManager.LayoutParams attrs,
        int viewVisibility, int displayId, Rect outContentInsets, Rect outStableInsets,
        Rect outOutsets, InputChannel outInputChannel) {
    return mService.addWindow(this, window, seq, attrs, viewVisibility, displayId,
            outContentInsets, outStableInsets, outOutsets, outInputChannel);
}
```

下面是addWindow方法的源码，我们看到由于AppWindowToken为null，从而返回了非法值

```
public int addWindow(Session session, IWindow client, int seq,
        WindowManager.LayoutParams attrs, int viewVisibility, int displayId,
        Rect outContentInsets, Rect outStableInsets, Rect outOutsets,
        InputChannel outInputChannel) {
    ...
    final int type = attrs.type;

    synchronized(mWindowMap) {
        ...
        WindowToken token = mTokenMap.get(attrs.token);
        if (token == null) {
            ...
        } else if (type >= FIRST_APPLICATION_WINDOW && type <= LAST_APPLICATION_WINDOW) {
            AppWindowToken atoken = token.appWindowToken;
            if (atoken == null) {
                Slog.w(TAG, "Attempted to add window with non-application token "
                      + token + ".  Aborting.");
                return WindowManagerGlobal.ADD_NOT_APP_TOKEN;
            } else if (atoken.removed) {
                ...
            }
            ...
        } 
        ...
    }
    ...
}
```

# 参考资料

- http://www.silencedut.com/2016/08/10/Android%E8%A7%86%E5%9B%BE%E6%A1%86%E6%9E%B6Activity,Window,View,ViewRootImpl%E7%90%86%E8%A7%A3/
- http://www.jianshu.com/p/a533467f5af5
- https://juejin.im/entry/571338c7c4c9710054cea455
- http://www.jianshu.com/p/634cd056b90c
- http://www.jianshu.com/p/628ac6b68c15
- http://blog.csdn.net/jasonwang18/article/details/72319705