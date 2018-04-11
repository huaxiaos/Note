# 核心方法

```
View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot)
```

从该方法中可以看出，Android系统采用的是PULL方式解析xml

> xml解析方式总共有三种：DOM、SAX、PULL，其中DOM不适合xml文档较大，内存较小的场景；SAX和PULL类似，但PULL的操作方式更为简单易用

这个方法主要有下面几个步骤：

1. 首先查找根节点，如果整个xml文件解析完毕也没看到根节点，会抛出异常；
2. 如果查找到的根节点名称是merge标签，会调用`rInflate`方法继续解析布局，最终返回root；
3. 如果是其他标签（View、TextView等），会调用`createViewFromTag`生成布局根View，并调用`rInflate`递归解析余下的子View，添加至布局根View中，最后视root和attachToRoot参数的情况最终返回view或者root

#root & attachToRoot

## root == null 

attachToRoot是什么都没有意义，此时传进来的布局会被加载成为一个View并直接返回，布局根View的android:layout_width(height)等属性会被忽略

## root != null

### attachToRoot == true

传进来的布局会被加载成为一个View并作为子View添加到root中，最终返回root；
而且这个布局根节点的`android:layout`参数会被解析用来设置View的大小。

### attachToRoot == false

传进来的布局会被加载成为一个View并直接返回。

布局根View的`android:layout`属性会被解析成LayoutParams并保留。(root只用来参与生成布局根View的LayoutParams)

# rInflate方法

```
void rInflate(XmlPullParser parser, View parent, Context context,
AttributeSet attrs, boolean finishInflate) throws XmlPullParserException, IOException {
	...
    while (((type = parser.next()) != XmlPullParser.END_TAG ||
    parser.getDepth() > depth) && type != XmlPullParser.END_DOCUMENT) {

    if (type != XmlPullParser.START_TAG) {
    continue;
    }

    final String name = parser.getName();

    if (TAG_REQUEST_FOCUS.equals(name)) {
    	...
    } else if (TAG_TAG.equals(name)) {
    	...
    } else if (TAG_INCLUDE.equals(name)) {
    	...
    } else if (TAG_MERGE.equals(name)) {
    	...
    } else {
    	// 核心部分
        final View view = createViewFromTag(parent, name, context, attrs);
        final ViewGroup viewGroup = (ViewGroup) parent;
        final ViewGroup.LayoutParams params = viewGroup.generateLayoutParams(attrs);
        rInflateChildren(parser, view, attrs, true);
        viewGroup.addView(view, params);
    }
    }
    ...
}
```

这个方法主要有下面几个步骤

1. 几个特殊标签的处理，如`requestFocus`、`include`等
2. 调用了`createViewFromTag`生成当前解析到的View节点，并递归调用`rInflate`逐层生成子View，添加到各自的上层View节点中

# createViewFromTag 方法

```
View createViewFromTag(View parent, String name, Context context, AttributeSet attrs, boolean ignoreThemeAttr) {
	...

    try {
    	View view;
    if (mFactory2 != null) {
    	view = mFactory2.onCreateView(parent, name, context, attrs);
    } else if (mFactory != null) {
    	view = mFactory.onCreateView(name, context, attrs);
    } else {
    	view = null;
    }

    if (view == null && mPrivateFactory != null) {
        view = mPrivateFactory.onCreateView(parent, name, context, attrs);
    }

    if (view == null) {
    	final Object lastContext = mConstructorArgs[0];
    	mConstructorArgs[0] = context;
    	try {
    		if (-1 == name.indexOf('.')) {
    			view = onCreateView(parent, name, attrs);
    		} else {
    			view = createView(name, null, attrs);
    		}
    	} finally {
    	mConstructorArgs[0] = lastContext;
    	}
    }

    return view;
    } catch (InflateException e) {
    	...
    } ...
}
```

这个方法主要有下面几个步骤

1. 依次判断mFactory2、mFactory、mPrivateFactory是否为null，执行其onCreateView方法
2. 如果上述变量都为null，则执行自己的onCreateView方法或者createView方法

> onCreateView内部调用的也是createView方法

# createView 方法

```
public final View createView(String name, String prefix, AttributeSet attrs)
            throws ClassNotFoundException, InflateException {
        Constructor<? extends View> constructor = sConstructorMap.get(name);
        Class<? extends View> clazz = null;
        try {
            if (constructor == null) {
                clazz = mContext.getClassLoader().loadClass(
                        prefix != null ? (prefix + name) : name).asSubclass(View.class);
                
                ...
                constructor = clazz.getConstructor(mConstructorSignature);
                sConstructorMap.put(name, constructor);
            } else {
                ...
            }
            Object[] args = mConstructorArgs;
            args[1] = attrs;
            return constructor.newInstance(args);
        } catch (NoSuchMethodException e) {
            ...
        }
}
```

判断缓存中是否有View的实例，如果没有的话，通过反射创建一个，并返回

# 参考资料

http://allenfeng.com/2017/02/24/how-android-layout-inflater-work/

https://blog.csdn.net/guolin_blog/article/details/12921889



