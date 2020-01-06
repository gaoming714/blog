---
title: 利用Msys2定制出Windows平台上Linux的shell方案
date: 2019-12-26 21:46:49
tags:
    - Msys2
    - shell
---

![](/images/Msys2-Shell.png)

## 背景

有那么一些程序员，喜欢Linux的shell，同时也很喜欢Windows的环境，
（或者说平时习惯了在Windows上的软件，但是写代码的时候却偏偏钟情于Linux，或者不得不用Linux）
那么我这里就为你提供了一个定制方案。（本人没怎么使用过Mac，暂且略过）

简单概括本文介绍了如何在Windows平台与Windows编程环境（本文以Python和nodejs为例）搭建一个几乎和Linux环境一样的shell，用作开发。这个桥梁就是[Msys2][1]，本文就是如何巧妙的运用Msys2构建开发环境。从用户角度看更接近一个Linux开发环境，却可以使用很多原本习惯的Windows软件。

<!--more-->

## 类似的解决方案

说一说背景，同时能够满足Linux和Windows的类似方案有以下几种：

>  1. 图形化的Linux。不得不说近些年Linux发展已经可以在PC上安装一个gnome或者KDE，普通的多媒体功能都是轻轻松松，尤其像Ubuntu可以说是开箱即用，当然如果想在这里玩个Windows大型游戏可以绕路了，有没有软件全看有没有Linux版本了。（完美Linux，勉强照顾Windows）
>  2. 用虚拟机。Windows/Linux + vbox/vmware， 虚拟机真是的好东西，优点几乎完美的兼容两个环境， 缺点需要设计两套系统的数据交互，还有就是太占资源，如果恰好还是那种图像或者数据，需要吃硬件的，这个方法基本就不要想了，双倍的东西。（原生系统满分，勉强照顾虚拟机环境）
>  3. Linux环境中装Windows的运行环境，比如wine，是一个能够在多种 POSIX-compliant 操作系统（诸如 Linux，Mac OSX 及 BSD 等）上运行 Windows 应用的兼容层。 我个人使用的感觉是兼容性有点呵呵，Windows这个变化的速度，这个兼容性稳定性不敢恭维。（原生系统给你满分，Windows拼人品）。
>  4. Windows环境中装Linux的运行环境，MinGW & Cygwin， 和上面一个类似，不过呢好在Linux的兼容层稳定一些，使用感觉也是一般，大的程序还是困难。

注意我要说的第五种方案来了，就是[Msys2][2]，他其实是一个 Windows平台上的 MinGW Cygwin pacman 融合到了一起的产物，好处就是他真的提供了一个比较舒服的 POSIX-compliant 操作系统，包管理使用的是pacman，内置了大部分常用的shell命令（例如 wget expect tcl cat grep vim-huge ），但是他存在一个问题，就是有一些程序需要调用的Windows环境，还是不能很好地运行，这个时候关键的地方来了，要是能干脆抛开Msys2的环境直接使用Windows版本的软件，只是将Msys2的Shell作为入口呢？对哦，可以的。（cmd能做到的，我都能做到，而且更像Linux）

没错！从头到尾，我定制的底层都是在Windows上，但是开发过程由于针对Python这类脚本语言，很多交互都限于Msys2的shell和python之间，所以几乎可以模拟出Linux的感觉。对于移植到服务器上更为贴近。

## 定制

接下来就是具体的定制步骤，当然了开箱即用也是绝美的，就可以轻松运行Windows程序，我只是定制的更适合。

> tutorial 请从官网下载Msys2，尝试运行一下 /c/Windows/System32/notepad.exe , 如果也已经换新鼓舞，我想你已经懂我的意思了。直接让Bash调用Windows的python程序就可以了，但是也有一些小技巧要注意，下文其实大部分都是我的定制过程。

### 特别提醒

Msys2中运行某些CLI交互的Windows版本程序，需要使用`winpty`，例如 `winpty python`，否则命令行显示字符有问题。

### 下载软件

为了方便介绍，我这里将Python和nodejs还有Visual studio code一并下载下来。
对于python我用的winpython，因为安装方便，官网的应该也是可以，需要你尝试一下。
官网下载 Msys2  `http://www.Msys2.org/`
Python的发行版，WinPython下载  `http://winpython.github.io/`
官网下载 nodejs `https://nodejs.org/en/`
官网下载 Visual Studio Code  `https://code.visualstudio.com/`

### 解压软件

Visual Studio Code 安装随意，我只是喜欢这个编辑器。下文经常编辑文件可以用这个。
编辑的文件目录，如果是 / 开头表示的是Linux的地址，也就是Msys的相对地址，
例如  `/etc/profile.d/lang.sh`  表示的是   `D:\Software\Msys2\etc\profile.d\lang.sh`

Msys2 Python nodejs 软件
请务必解压到“英文前缀，不包含空格或者中文，字符较短”的目录中。例如

    D:\Software\Msys2
    D:\Software\Python
    D:\Software\nodejs

### 更改Msys2的源

原始Msys2用的源比较慢，这里推荐一个快一些的，更新这几个文件的内容即可。

    /etc/pacman.d/mirrorlist.mingw32
    Server = http://mirrors.ustc.edu.cn/Msys2/mingw/i686/

    /etc/pacman.d/mirrorlist.mingw64
    Server = http://mirrors.ustc.edu.cn/Msys2/mingw/x86_64/

    /etc/pacman.d/mirrorlist.Msys
    Server = http://mirrors.ustc.edu.cn/Msys2/Msys/$arch

### 更新语言

我这里通过更改minttyrc引入英文配置，暂时不需要更改lang.sh，详见下文minttyrc部分

    不清楚的原因，有些软件可能出现乱码，请删除这个从环境变量中过去语言的选项。
    即删除这个文件中的这一行。
        configure /etc/profile.d/lang.sh
        # test -z "${LC_ALL:-${LC_CTYPE:-$LANG}}" && export LANG=""

### 安装一些必要的软件包

按照我个人的习惯我安装了一下几个。
注意 `winpty` 能够解决很多现实不正常的情况。

    pacman -S vim
    pacman -S zsh
    pacman -S p7zip
    pacman -S rsync
    pacman -S git
    pacman -S winpty

    ln -s /bin/vim.exe /bin/vi  # 给vim做个软链接

### 配置主目录

也是根据我的喜好定制了这个，/etc/nsswitch.conf

    # Begin /etc/nsswitch.conf

    passwd: files db
    group: files db

    db_enum: cache builtin

    db_home: /root
    db_shell: /usr/bin/bash
    db_gecos: cygwin desc

    # End /etc/nsswitch.conf

这一步完成后请重启Msys2

### 安装oh-my-zsh 并启用 zsh

不要问我为什么，这个东西太好用了，执行

    sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"

再次编辑一下 /etc/nsswitch.conf ， 启用zsh

    # Begin /etc/nsswitch.conf

    passwd: files db
    group: files db

    db_enum: cache builtin

    db_home: /root
    db_shell: /usr/bin/zsh
    db_gecos: cygwin desc

    # End /etc/nsswitch.conf

这一步完成后请重启Msys2, 注意上面三步顺序，更改 db_home => 安装 oh-my-zsh => 更改 db_shell

### 定制Msys2 字体和背景色

安装个我喜欢的字体，[Source Code Pro Semibold][2]
配置文件 ~/.minttyrc，
（如果上文定制了db_home，则是root下的，如果没有定制root，则可能是Windows用户目录下）

BoldAsFont=-1
CursorType=block
Font=Source Code Pro Semibold
FontHeight=12
FontWeight=600
Scrollbar=none
Columns=100
Rows=35
Locale=C
Charset=UTF-8

ForegroundColour=235, 219, 178
BackgroundColour=  0,  43,  54
CursorColour=     65, 255,  65

### 定制编程链

如果没有这一步，前面都是假把式，毕竟我是为了编程才引入的Msys2.
其实很简单的，就是把Python的脚本目录加入到环境变量中就可以了，但是需要winpty和python作为前缀。
所以我这里写了一个Shell的函数供大家使用。
请把这几行加入到你的  .bashrc 或者 .zshrc 中。

    # Python
    PATH_python=/d/Portable/Python/python-3.7.4.amd64
    function py(){ winpty "${PATH_python}/python" "${PATH_python}/Scripts/$@" ;}
    function python(){ winpty "${PATH_python}/python" "${PATH_python}/Scripts/ipython" "$@" ;}
    # Node.js
    PATH_node=/d/Portable/nodejs
    PATH=${PATH_node}:$PATH

使用python 执行脚本（我个人用ipython代替了python）：

    bash# python

使用python的其他系统脚本，例如pip安装package，前面加py作为前缀：

    bash# py pip install

使用nodejs 执行脚本，因为已经在系统变量中，所以锁心所欲啦。

    bash# node --verison

## 补充

最后，有人可能会问这不是多此一举么，都在Windows上还要绕一下Linux干啥，其实你仔细品，对开发而言，入口是shell，编程环境是python，在不关心底层实现的情况下，这样开发过程更接近Linux，对于提高Linux才做也是一个帮助，同时有一些Windows的编辑器（墙裂推荐visual studio code）真的是提高开发效率。

我之所以这么用，是因为工作中需要使用expect，这个是只有Linux版本，Windows没有，但是恰巧的是在构建的Msys这个虚拟的Linux中，有expect就很好地满足了要求。

最后的最后，如果你觉得定制步骤太麻烦，也可以打赏我，后台留言你的邮箱，一步到位这个定制好的Msys2压缩包。

  [1]: http://www.Msys2.org/
  [2]: https://github.com/silkeh/latex-sourcecodepro
