---
title: 如何python中合理的跨文件使用相对路径
date: 2020-04-13 21:06:32
tags:
	- python
	- scheme
---

![python_path](/images/python_path.jpg)

python 在package/module引用过程是很轻松的访问模块，同级目录import了解一下，总是OK的，就算不懂相对路径基本也不会有问题，但是有的时候引用了其他目录（尤其是非子目录），就会发现引用的module失败，这里就涉及到相对引用和绝对引用。

<!--more-->

## 引用的基础知识

`from .util import log`

`import .abc`

是相对引用（开头带点的），`.`一个点指当前目录找，`..`两个点上一级目录，以此类推；



`from util import log` 

`import abc`  

是绝对引用（正常开头的），系统设定的包搜寻目录里找，即`sys.path



因为PEP 328的规定，相对引用相对来说受限（ `__main__`不含有包的信息 ），所以我们不探讨import相对问题。

并且，“当前路径” 代表的是被执行的脚本文件的所在路径，为了防止程序乱掉，所以我直接使用绝对引用介绍。



## 目录结构

目录结构

![python_path](/images/python_path_folder.png)



## 相同目录情况



```python
## Algo.py
import foo

r = foo.function()
print(r)
```

```python
## foo.py
import os
def function():
    file = "../data/config.json"
    return os.path.exists(file)
```

```shell
➜  src python Algo.py
True
```

Algo.py 作为主程序调用了同目录的foo.py的函数，Algo 通过使用 相对路径 `import foo`合理。

foo使用了相对路径也没有问题，因为Algo和foo有相同的目录，绝对路径和相对路径都一样。

## 不同目录情况

其实，稍微大一些的程序，还是会用到调用一个不再统一目录的情况，比如Algo调用log内的函数。

此时log内函数也可以暂时使用相对路径，Algo和log对应的父目录不同的特殊情况。



```python
## Algo.py
import os
import sys
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
sys.path.append(BASE_DIR)
import util.log as log

r = log.function()
print(r)
```

```python
## log.py
import os
def function():
    file = "../data/config.json"
    return os.path.exists(file)
```

方法是，将主程序Algo的环境中添加父目录，这样Algo就可以通过调用  util.log ,访问根目录下的子目录，这样需要注意每个文件的位置，保证他们向上都最终指向根目录`BASE_DIR`，注意log.py的所在目录必须有`__init__.py` 来说明这是一个模块。

## 被调用的module内部

被调用的module内部使用先对路径其实是不对的，因为

**“当前路径” 代表的是被执行的脚本文件的所在路径，**

所以在被调用的module内部应该最好不使用相对位置，而是使用绝对位置。

这里我建议使用的是针对于根目录的相对位置。

```python
import os
import sys
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
sys.path.append(BASE_DIR)

def log():
   config_json = os.path.join(BASE_DIR,"data","config.json")
```

首先将环境变量增加根目录`BASE_DIR`,然后利用`os.path.join`拼出来所需要的绝对路径。这里要注意，不要使用字符串拼接，因为在不同操作系统中可能不兼容，推荐使用`os.path`。



## 结束语

本文，主要就是想说，在项目中尽量使用带有根目录作为基础的绝对路径，这样项目内的文件组织起来不容易错乱，否则经常会遇到找不到module，找不到file的问题。

其次就是绝对路径的拼接要使用`os.path`,兼容多平台。