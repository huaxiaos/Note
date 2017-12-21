---
title: Constraintlayout填坑之ViewStub
date: 2017-07-15 22:18:46
tags: [Android,Constraintlayout,ViewStub]
categories: Android
---

# 前言

Constraintlayout是Google最新推出的视图层控件，具体的介绍我就不在这里多说了。本文主要针对使用过程中的一些“坑”做一些记录

# ViewStub不能正确显示的问题

## 场景

如果有3个view，其中ViewStub夹在其他两个View中间，那么在运行时会只显示ViewStub，而不显示第3个view

## 解决方案

ViewStub中有这样一个属性：inflatedId，官方文档中的解释如下：

> android:inflatedId---Overrides the id of the inflated View with this value

将该属性的值，与ViewStub的`android:id`的值设置的完全一样，即可解决问题
