---
title: replaceFirst与java.util.regex.PatternSyntaxException
date: 2017-07-15 22:19:21
tags: [Android,replaceFirst,PatternSyntaxException]
categories: Android
---

# 前言

使用`replaceFirst`方法时，出现`java.util.regex.PatternSyntaxException`错误

# 问题原因

这个问题起初困扰了我很久，最后经过分析发现使用`replaceFirst`替换的字符串中，有一个不完整的"\("，然后猜想不会是因为正则表达式的某种原因造成的吧。通过查看`replaceFirst`的源代码，终于发现了问题的根源：`replaceFirst`使用了正则表达式

# replaceFirst

源码如下

```
/**
 * Replaces the first match for {@code regularExpression} within this string with the given
 * {@code replacement}.
 * See {@link Pattern} for regular expression syntax.
 *
 * <p>If the same regular expression is to be used for multiple operations, it may be more
 * efficient to reuse a compiled {@code Pattern}.
 *
 * @throws PatternSyntaxException
 *             if the syntax of the supplied regular expression is not
 *             valid.
 * @throws NullPointerException if {@code regularExpression == null}
 * @see Pattern
 * @since 1.4
 */
public String replaceFirst(String regularExpression, String replacement) {
    return Pattern.compile(regularExpression).matcher(this).replaceFirst(replacement);
}
```

相信各位看到这里都清楚了

# 延伸

不仅`replaceFirst`，`replaceAll`也使用了正则表达式。甚至String的`split`也是用到了正则表达式，使用的时候需要注意。

值得一提的是，`replace`方法没有使用正则表达式
```
/**
 * Returns a copy of this string after replacing occurrences of the given {@code char} with another.
 */
public String replace(char oldChar, char newChar) {
    String s = null;
    int _count = count;
    boolean copied = false;
    for (int i = 0; i < _count; ++i) {
        if (charAt(i) == oldChar) {
            if (!copied) {
                s = StringFactory.newStringFromString(this);
                copied = true;
            }
            s.setCharAt(i, newChar);
        }
    }

    return copied ? s : this;
}
```