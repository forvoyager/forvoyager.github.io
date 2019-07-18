---
title: 8.python性能分析之函数执行时间
brief: python性能分析
key: 'python性能分析,执行时间'
tags:
  - python
toc: true
comments: true
date: 2019-07-18 10:51:04
---

统计python执行过程中的耗时情况……
![](/pic/8/1.jpg)

<!-- more -->

# 生成性能分析文件

用cProfile（Python默认的性能分析器）生成性能分析文件，命令格式如下：
```
python -m cProfile -o result.cprofile your_python_file.py [parameters]
```
如下
```
python -m cProfile -o result.cprofile exe_pre.py -r 600549
```
最后会在当前路径下生成一个result.cprofile的分析文件。

# 可视化分析文件

可选的可视化工具比较多，如gprof2dot、Matlab、snakeviz等。以snakeviz为例：

### 安装snakeviz
```
pip install snakeviz
```

### 可视化
```
snakeviz result.cprofile
```
效果如图，风格自选：
![](/pic/8/2.jpg)
