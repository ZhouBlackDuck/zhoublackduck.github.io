---
date: 2025-04-02
tags:
  - python
categories:
  - 记录
  - 学习
title: python笔记（一）
---

### Backports

这是一个pip包，用于给早期版本的python编译器（如3.2）提供新版具有的功能

### Conda环境

自定义环境路径

```bash
conda config --add envs_dirs /path/to/custom/envs/
```

### 解包

支持`*`解包，需要实现`__iter__`

支持`**`解包，需要实现`__getitem__`和`keys`
