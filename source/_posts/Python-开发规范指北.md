---
title: Python 开发规范指北
date: 2025-08-04 09:00:55
update:
tags:
  - python
  - 实践
categories: python
keywords:
  - python
  - 开发规范
top_img:
cover:
---

## 日志打印

1. 在运行 cmd，shell 等命令时，以至少`info`级别打印执行的命令，以至少`debug`级别打印命令的原始输出。

2. 在进行单文件 io 操作前，至少以`info`级别打印 io 对象的路径。同时操作复数同类文件，如`["1.jpg","2.jpg","3.jpg","4.jpg"...]`则至少要以`debug`级别打印

## 路径操作

### 获取根目录

在源码运行、pyinstaller 打包、nuitka 打包等不同运行条件下，`__file__`的值并不相同。一般情况下我们更倾向于推荐使用`sys.argv[0]`作为获取目录的方法，但这个方法也有需要注意的地方。请看下面这个 fastapi 的例子。

```bash
uvicorn main:app --reload --port 8001
```

在这个例子中，我们使用了 uvicorn 来启动`main.py`中的 fastapi 服务。但如果在这个程序中执行`sys.argv[0]`，将获取到的路径会是`uvicorn`的而非`main.py`的。

### 不要直接使用内置函数获取工作目录

任何情况下，我们都强烈建议**不要**使用`os.getcwd()`和`Path.cwd()`方法。来看下面的例子。

```bash
foo/
├─ foo1/
│  └─ ...
└─ foo2/
   └─ main.py
```

如果在 foo1 尝试执行`python foo/foo2/main.py`，可能会产生意料外的状况。

```bash
/foo/foo1> python /foo/foo2/main.py
```

此时获取的结果时`foo1`而非`foo2`。
