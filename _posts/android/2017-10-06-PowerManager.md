---
layout: default
title: "PowerManager"
category: android
---

## PowerManager 

分析PowerManagerService可以先从PowerManager入手



PowerManager是面向应用的，javadoc解释的比较多

几种常见的锁  

PARTIAL_WAKE_LOCK cup锁，用户按了power键之后不会隐式释放

SCREEN_DIM_WAKE_LOCK 屏幕保存不灭，但是可以dim

SCREEN_BRIGHT_WAKE_LOCK 屏幕保持不灭，亮度为设置的值，键盘背光可以灭

FULL_WAKE_LOCK 屏幕和键盘背光保持不灭

PROXIMITY_SCREEN_OFF_WAKE_LOCK 近感灭屏锁不会阻止系统进入休眠，除非灭屏原因是近感所致。

常见的flag    
ACQUIRE_CAUSES_WAKEUP 获得后会导致系统亮屏

常用的接口  
userActivity（） 使用这个接口，如果系统已经休眠，则不会导致系统唤醒。如果屏幕亮屏，则会重新计算灭屏超时，如果屏幕处于dim状态，则屏幕亮度会提升

goToSleep（）覆盖所有持有的锁，进入睡眠。一般是power键按下

wakeUp（）唤醒系统（屏幕）。

睡眠深度顺序  wake-dream-doze-sleep

isPowerSaveMode() 是否在省电/低电模式（关心的是有电没电的问题）

isDeviceIdleMode（）是否在deviceidle（深度空闲）模式（关心的是设备现在是否空闲的问题）

isLightDeviceIdleMode（）是否在浅度空闲模式

isIgnoringBatteryOptimizations（）是否在电池优化白名单

----------

## PowerManagerService 

PowerManager大多数接口都会调用到PowerManagerService中来  