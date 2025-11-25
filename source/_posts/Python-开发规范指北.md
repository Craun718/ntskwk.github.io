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

### 不要用内置函数获取工作目录

任何情况下，我们都强烈建议**不要**使用`os.getcwd()`和`Path.cwd()`方法。来看下面的例子。

```bash
foo/
├─ foo1/
│  └─ ...
└─ foo2/
   └─ main.py
```

如果在 foo1 尝试执行`python ../foo2/main.py`，可能会产生意料外的状况。

```bash
/foo/foo1> python ../foo2/main.py
```

此时获取的结果是`foo1`而非`foo2`。

### 谨慎使用 with 语句操作临时文件和临时目录

来看一个 Fastapi 中的例子。

```python
@app.get("/repeat")
def read_root(msg: str):
    """Repeat the message in a file and return it as a file response."""
    with TemporaryDirectory() as tmpdir:
        with open(f"{tmpdir}/msg.txt", "w", encoding="utf-8") as f:
            f.write(msg)

        return FileResponse(f"{tmpdir}/msg.txt", filename="msg.txt")
```

在这个例子中，使用了`TemporaryDirectory`来创建一个临时目录并在其中创建了一个文件，然后将这个文件作为响应返回给客户端。然而，这里存在一个问题——这个函数执行`return`语句之前，会先对`with`语句进行后处理。在`with`语句结束时，`TemporaryDirectory`就会被删除，所以返回的`FileResponse`实际上是在外部进行处理的。也就是说这段代码实际上是会在`with`语句的外面读取这个文件，因此将会收到一个`FileNotFound`错误。（我个人使用的做法是，将文件写入 buffer 变量，然后再进行返回）

## http请求

### 在日志中记录请求的完整信息

在处理 http 请求时，除了记录请求的完整信息，还要记录请求的响应。这对排查与外部服务有关的问题时非常有帮助。

## Python 的语言特性

### 不会发生的变量遮盖

在 Python 中，变量初始化语句和复制的语句是相同的。同时，`for`语句不具备单独的作用域。因此，在`for`循环中，如果将程序过程的中途变量定义为与外部相同，则会错误的修改外部变量。

```javascript
// javascript
if (true) {
  let a = 1;
}

a; // ReferenceError: a is not defined

for (let i = 0; i < 10; i++) {
  let b = 2;
}

b; // ReferenceError: b is not defined
```

```Python
# python
if True:
    a = 1

a  # 1

for i in range(10):
    b = 2

b  # 2
```
