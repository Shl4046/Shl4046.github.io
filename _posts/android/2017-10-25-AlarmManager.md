---
layout: default
title: "AlarmManager"
category: android
istop: "true"
---
>     {{ page.date | date: "%Y-%m-%d," }} {{ page.content | number_of_words | append: "chars"}}
>     {{ page.tags }}

##　AlarmManager

系统定时器 ！

两种时间  
绝对时间 墙上时间：   就是几年几月几日几时的意思   
相对时间 流逝时间：   就是开机到现在流逝了多少时间  相差多少时间的意思  
  
闹钟类型 

RTC_WAKEUP = 0  使用绝对时间（System.currentTimeMillis()），到时将唤醒系统  

RTC = 1 使用绝对时间（System.currentTimeMillis()），到时不会唤醒系统

ELAPSED_REALTIME_WAKEUP = 2  使用相对时间（SystemClock.elapsedRealtime()），到时唤醒系统

ELAPSED_REALTIME =3 使用相对时间（SystemClock.elapsedRealtime()），到时不会唤醒系统

几个常见的flag

FLAG_STANDALONE  这个alarm放在一个batch

FLAG_WAKE_FROM_IDLE 可以在设备空闲状态触发执行，设备将退出空闲状态

FLAG_ALLOW_WHILE_IDLE 可以在设备空闲状态触发执行，设备不会退出空闲状态，可能比设定的时间提前

FLAG_IDLE_UNTIL 退出空闲状态的alarm,设置的时候表示系统进入IDLE，触发的时候表示系统退出IDLE

基本上所有的接口最后都调用到`setImpl`
``` java
private void setImpl(int type, long triggerAtMillis, long windowMillis, long intervalMillis,
            int flags, PendingIntent operation, final OnAlarmListener listener, String listenerTag,
            Handler targetHandler, WorkSource workSource, AlarmClockInfo alarmClock)；
```

几个注意点：  
除了alarmclock，其他的定时器都十不准的  

SDK 19（KITKAT）之前的定时器都是准的

----------

##　AlarmManagerService