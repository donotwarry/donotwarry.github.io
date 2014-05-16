---
layout: post
title: Android中动态加载APK的原理解析
---

{{ page.title }}
=========================

> 2014.5.16 - Shanghai


插件框架原理
--------
######代码的加载######

Android系统中提供了一个`DexClassLoader`类，使得程序可以动态载入额外的程序代码。
###Class Overview###
A class loader that loads classes from .jar and .apk files containing a classes.dex entry. This can be used to execute code not installed as part of an application.

######资源的加载######

ADT会在编译时动态构建R文件对静态资源文件做索引，每个工程会生成自己的R文件，如果我们不做任何事，在插件项目里面直接使用`getResouces()`去获取资源，那样就只会去主项目的资源文件里寻找，会出现找不到或者找错的结果。

Resouces对象支持动态创建，构造函数见官方说明如下：
#####public Resources (AssetManager assets, DisplayMetrics metrics, Configuration config)#####
- Added in API level 1

> Create a new Resources object on top of an existing set of assets in an AssetManager.

Parameters

> `assets`	Previously created AssetManager.
 
> `metrics`	Current display metrics to consider when selecting

> `computing` resource values.
config	Desired device configuration to consider when selecting/computing resource values (optional).

其中`metrics`和`config`我们都可以直接继承父`Resources`对象的，`assets`对象需要重新创建

AssetManager的构造方法是`@hide`的，我们可以通过反射：

    AssetManager am = (AssetManager) AssetManager.class.newInstance();
光构造出来还是不够，我们需要设定资源的加载路径，AssetManager中有个隐藏的方法

    /**
     * Add an additional set of assets to the asset manager.  This can be
     * either a directory or ZIP file.  Not for use by applications.  Returns
     * the cookie of the added asset, or 0 on failure.
     * {@hide}
     */
    public final int addAssetPath(String path) {
        int res = addAssetPathNative(path);
        return res;
    }

通过反射，我们指定插件包路径为AssetPath，这样就ok了。
