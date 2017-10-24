---
layout: default
title: "build_system"
category: android
istop: "true"
---

# build system

安卓编译系统主要由build/core/*.mk和各个模块的Android.mk文件组成,可以编译多种类型的模块，静态库、动态库、可执行文件、apk、jar等，具体有Android.mk文件来配置.支持单模块编译，准确说每一个mk文件定义的target都可以单独编译出来

构建大版本
{% highlight sh %}
. ./build/envsetup
lunch
make 
{% endhighlight %}
单编
{% highlight sh %}
make -j8 system.img
make -j8 boot.img
make -j8 framework
make -j8 services
make -j8 mudule-name
{% endhighlight %}
常用的编译options： -j8，showcommands
使用`mm,mmm,make`编译时可以加上`-j8`,8表示cpu的核数，可以加快编译

### build/envsetup.sh

可以提高工作效率的函数
>     lunch:     lunch <product_name>-<build_variant>
>     croot:     Changes directory to the top of the tree.
>     m:         Makes from the top of the tree.
>     mm:        Builds all of the modules in the current directory, but not their dependencies.
>     mmm:       Builds all of the modules in the supplied directories, but not their dependencies.
>                To limit the modules being built use the syntax: mmm dir/:target1,target2.
>     mma:       Builds all of the modules in the current directory, and their dependencies.
>     mmma:      Builds all of the modules in the supplied directories, and their dependencies.
>     cgrep:     Greps on all local C/C++ files.
>     ggrep:     Greps on all local Gradle files.
>     jgrep:     Greps on all local Java files.
>     resgrep:   Greps on all local res/*.xml files.
>     mangrep:   Greps on all local AndroidManifest.xml files.
>     mgrep:     Greps on all local Makefiles files.
>     sepgrep:   Greps on all local sepolicy files.
>     sgrep:     Greps on all local source files.
>     godir:     Go to the directory containing a file.

### build/core/main.mk

编译的主要流程控制核心文件
最开始定义虚目标`droid`,默认编译目标`droid_targets`依赖droid
{% highlight sh %}
    .PHONY: droid
    DEFAULT_GOAL := droid
    $(DEFAULT_GOAL): droid_targets

    .PHONY: droid_targets
    droid_targets:
{% endhighlight %}
根据是否单编app，`droid_targets`依赖不同
{% highlight sh %}
    droid_targets: apps_only
    ...
    droid_targets: droidcore dist_files
{% endhighlight %}
`droidcore`就是基本就是整个安卓系统完整的版本了包括`system.img,boot.img，data.img`...
{% highlight sh %}
    .PHONY: droidcore
    droidcore: files \
        *systemimage* \
        $(INSTALLED_BOOTIMAGE_TARGET) \
        $(INSTALLED_RECOVERYIMAGE_TARGET) \
        $(INSTALLED_USERDATAIMAGE_TARGET) \
        $(INSTALLED_CACHEIMAGE_TARGET) \
        $(INSTALLED_VENDORIMAGE_TARGET) \
        $(INSTALLED_SYSTEMOTHERIMAGE_TARGET) \
        $(INSTALLED_FILES_FILE) \
        $(INSTALLED_FILES_FILE_VENDOR) \
        $(INSTALLED_FILES_FILE_SYSTEMOTHER)
{% endhighlight %}

### build/core/config.mk

定义了很多全局变量以及`include $(BUILD_SYSTEM)/envsetup.mk`
编译时用到的*头文件*路径，另外`$(BUILD_SYSTEM)/pathmap.mk`中也有一些头文件路径的定义
{% highlight sh %}
SRC_HEADERS := \
	$(TOPDIR)system/core/include \
	$(TOPDIR)system/media/audio/include \
	$(TOPDIR)hardware/libhardware/include \
	$(TOPDIR)hardware/libhardware_legacy/include \
	$(TOPDIR)hardware/ril/include \
	$(TOPDIR)libnativehelper/include \
	$(TOPDIR)frameworks/native/include \
	$(TOPDIR)frameworks/native/opengl/include \
	$(TOPDIR)frameworks/av/include \
	$(TOPDIR)frameworks/base/include
{% endhighlight %}

**编译用到的tools**

{% highlight sh %}
...
ACP := $(prebuilt_sdk_tools_bin)/acp
AIDL := $(prebuilt_sdk_tools_bin)/aidl
AAPT := $(prebuilt_sdk_tools_bin)/aapt
AAPT2 := $(prebuilt_sdk_tools_bin)/aapt2
ZIPALIGN := $(prebuilt_sdk_tools_bin)/zipalign
SIGNAPK_JAR := $(prebuilt_sdk_tools)/lib/signapk$(COMMON_JAVA_PACKAGE_SUFFIX)
...
JACK := $(HOST_OUT_EXECUTABLES)/jack
LEX := prebuilts/misc/$(BUILD_OS)-$(HOST_PREBUILT_ARCH)/flex/flex-2.5.39
BISON_PKGDATADIR := $(PWD)/external/bison/data
BISON := prebuilts/misc/$(BUILD_OS)-$(HOST_PREBUILT_ARCH)/bison/bison
YACC := $(BISON) -d

YASM := prebuilts/misc/$(BUILD_OS)-$(HOST_PREBUILT_ARCH)/yasm/yasm
...
PROGUARD := external/proguard/bin/proguard.sh
JAVATAGS := build/tools/java-event-log-tags.py
RMTYPEDEFS := $(HOST_OUT_EXECUTABLES)/rmtypedefs
APPEND2SIMG := $(HOST_OUT_EXECUTABLES)/append2simg
VERITY_SIGNER := $(HOST_OUT_EXECUTABLES)/verity_signer
BUILD_VERITY_TREE := $(HOST_OUT_EXECUTABLES)/build_verity_tree
BOOT_SIGNER := $(HOST_OUT_EXECUTABLES)/boot_signer
FUTILITY := prebuilts/misc/$(BUILD_OS)-$(HOST_PREBUILT_ARCH)/futility/futility
VBOOT_SIGNER := prebuilts/misc/scripts/vboot_signer/vboot_signer.sh
...
{% endhighlight %}

**编译宏控**

{% highlight sh %}
HOST_GLOBAL_CFLAGS += $(COMMON_GLOBAL_CFLAGS)
HOST_RELEASE_CFLAGS += $(COMMON_RELEASE_CFLAGS)

HOST_GLOBAL_CPPFLAGS += $(COMMON_GLOBAL_CPPFLAGS)
HOST_RELEASE_CPPFLAGS += $(COMMON_RELEASE_CPPFLAGS)

TARGET_GLOBAL_CFLAGS += $(COMMON_GLOBAL_CFLAGS)
TARGET_RELEASE_CFLAGS += $(COMMON_RELEASE_CFLAGS)

TARGET_GLOBAL_CPPFLAGS += $(COMMON_GLOBAL_CPPFLAGS)
TARGET_RELEASE_CPPFLAGS += $(COMMON_RELEASE_CPPFLAGS)
{% endhighlight %}

### build/core/envsetup.mk
设置了编译输出目录的组织结构，重点是导入了`include $(BUILD_SYSTEM)/product_config.mk``include $(board_config_mk)`这两个和产品和板级有关的配置mk  
{% highlight sh %}
...
TARGET_COPY_OUT_SYSTEM := system
TARGET_COPY_OUT_SYSTEM_OTHER := system_other
TARGET_COPY_OUT_DATA := data
TARGET_COPY_OUT_OEM := oem
TARGET_COPY_OUT_ODM := odm
TARGET_COPY_OUT_ROOT := root
TARGET_COPY_OUT_RECOVERY := recovery
...
...
PRODUCT_OUT := $(TARGET_PRODUCT_OUT_ROOT)/$(TARGET_DEVICE)

OUT_DOCS := $(TARGET_COMMON_OUT_ROOT)/docs

BUILD_OUT_EXECUTABLES := $(BUILD_OUT)/bin
...
...
ifeq ($(TARGET_IS_64_BIT),true)
# /system/lib always contains 32-bit libraries,
# and /system/lib64 (if present) always contains 64-bit libraries.
TARGET_OUT_SHARED_LIBRARIES := $(target_out_shared_libraries_base)/lib64
else
TARGET_OUT_SHARED_LIBRARIES := $(target_out_shared_libraries_base)/lib
endif
TARGET_OUT_RENDERSCRIPT_BITCODE := $(TARGET_OUT_SHARED_LIBRARIES)
TARGET_OUT_JAVA_LIBRARIES := $(TARGET_OUT)/framework
TARGET_OUT_APPS := $(TARGET_OUT)/app
TARGET_OUT_APPS_PRIVILEGED := $(TARGET_OUT)/priv-app
TARGET_OUT_KEYLAYOUT := $(TARGET_OUT)/usr/keylayout
TARGET_OUT_KEYCHARS := $(TARGET_OUT)/usr/keychars
TARGET_OUT_ETC := $(TARGET_OUT)/etc
TARGET_OUT_NOTICE_FILES := $(TARGET_OUT_INTERMEDIATES)/NOTICE_FILES
TARGET_OUT_FAKE := $(PRODUCT_OUT)/fake_packages
...
{% endhighlight %}
 