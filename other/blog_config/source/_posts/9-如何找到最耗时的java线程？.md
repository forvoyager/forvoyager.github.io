---
title: 9.如何找到最耗时的java线程？
brief: 副标题
key: 'java,线程,耗时'
tags:
  - JAVA
toc: true
comments: true
date: 2019-07-19 13:32:16
---

![](/pic/unclassified/d2.jpg)

<!-- more -->

如何找到最耗时的java线程？

## 找到JAVA进程pid

得先找到目标进程
```
ps -ef|grep java
或者
jps -mlv
```
结果如下
```
root     102945 100545  0 13:44 pts/0    00:00:00 grep --color=auto java
root     123398      1  0 7月18 ?       00:04:26 java -jar ./jar/p-discovery.jar
root     123584      1  0 7月18 ?       00:03:33 java -jar ./jar/p-config-service.jar
root     124627      1  1 7月18 ?       00:20:17 java -jar ./jar/p-invest-service.jar
```
第二列就是pid

## 找到线程中最耗时的线程TID

命令如下
```
top -Hp pid
```

本例如下
```
top -Hp 123584

结果如下：
top - 13:49:20 up 158 days,  1:52,  2 users,  load average: 0.34, 0.20, 0.16
Threads:  63 total,   0 running,  63 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.4 us,  1.0 sy,  0.0 ni, 98.4 id,  0.2 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 32528944 total,   252228 free, 28587204 used,  3689512 buff/cache
KiB Swap: 16777212 total, 14890172 free,  1887040 used.  3287160 avail Mem

   PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
123584 root      20   0 13.583g 1.136g  13268 S  0.0  3.7   0:00.00 java
123601 root      20   0 13.583g 1.136g  13268 S  0.0  3.7   0:21.79 java
123602 root      20   0 13.583g 1.136g  13268 S  0.0  3.7   0:23.72 java
123605 root      20   0 13.583g 1.136g  13268 S  0.0  3.7   0:38.36 java
123626 root      20   0 13.583g 1.136g  13268 S  0.0  3.7   0:01.59 java
123627 root      20   0 13.583g 1.136g  13268 S  0.0  3.7   0:00.00 java
123629 root      20   0 13.583g 1.136g  13268 S  0.0  3.7   0:18.94 java
123684 root      20   0 13.583g 1.136g  13268 S  0.0  3.7   0:00.39 java
```

找到最耗时的tid，转成16进制，如上面的123605耗时比较大
```
printf "%x\n" 123605

结果为：1e2d5
```

## 打印线程栈信息

得到上面的信息后，就可以找到目标：耗时长的线程。命令如下
```
jstack [pid] | grep [tid16进制]
```

本例如下
```
jstack 123584 | grep 1e2d5

结果如下：
"SimplePauseDetectorThread_0" #20 daemon prio=9 os_prio=0 tid=0x00007f9cac059000 nid=0x1e2ed waiting on condition [0x00007f9cbc8c1000]
```
**注**
实际过程中，这个命令需要多执行几次，观察栈的变化才能真正的确认问题。

