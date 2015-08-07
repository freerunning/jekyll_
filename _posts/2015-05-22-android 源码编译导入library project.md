---
layout: post
title: "android 源码编译导入library project"
date: 2015-05-22 19:07
comments: true
categories: Android
tags: Android libbrary project 源码
---

修改launcher，需要依赖一个library project:password，无法直接在源码中编译。google出来的结果没有合适的方法，最后在源码packageses/apps/Email中找到了答案。

需要修改2个文件:launcher的Android.mk、password的Android.mk

makefile文件目录为:

```
launcher/password/Android.mk
launcher/Android.mk
```


+ 首先编写password的Android.mk

{% highlight makefile %}
CAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)

# 引用了zxing-core.jar,放在password的libs目录下
LOCAL_PREBUILT_STATIC_JAVA_LIBRARIES := zxing-core-password:libs/zxing-core.jar
LOCAL_STATIC_JAVA_LIBRARIES := zxing-core-password

# password的包名
LOCAL_MODULE := com.dyu.password

# src和res
LOCAL_SRC_FILES := $(call all-java-files-under, src)
LOCAL_RESOURCE_DIR := $(LOCAL_PATH)/res
{% endhighlight %}

+ 然后编写launcher的Android.mk

{% highlight makefile %}
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)

LOCAL_MODULE_TAGS := optional
include $(CLEAR_VARS)
LOCAL_MODULE_TAGS := optional
       
# password目录
password_dir := password

# 指向password/libs下的zxing-core.jar
LOCAL_PREBUILT_STATIC_JAVA_LIBRARIES := zxing-core-launcher:$(password_dir)/libs/zxing-core.jar
LOCAL_STATIC_JAVA_LIBRARIES := zxing-core-launcher

res_dir := res $(password_dir)/res    
# src
LOCAL_SRC_FILES := $(call all-java-files-under, $(password_dir)/src)
LOCAL_SRC_FILES += $(call all-java-files-under, src)
# res
LOCAL_RESOURCE_DIR := $(addprefix $(LOCAL_PATH)/, $(res_dir))
# assets
LOCAL_ASSET_DIR := $(LOCAL_PATH)/$(password_dir)/assets

#  链接password
LOCAL_AAPT_FLAGS := --auto-add-overlay
LOCAL_AAPT_FLAGS += --extra-packages com.dyu.password

LOCAL_PACKAGE_NAME := launcher
LOCAL_CERTIFICATE := shared
LOCAL_OVERRIDES_PACKAGES := Home
include $(BUILD_PACKAGE)
{% endhighlight %}
password也可以放在与launcher同级目录，相应的修改 mk文件内的路径就行了。
