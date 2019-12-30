---
title: vim 炫酷定制计划
date: 2019-12-30 20:30:48
tags:
    - vim
    - Vundle
---

![](/images/vim-01.png)
![](/images/vim-02.png)
![](/images/vim-03.png)

有小伙伴咨询为什么我的 VI 那么好看，其实就是一个主题和插件，并没什么高大上的，筛选适合自己的插件可能久一些，这里讲讲方法和我的定制步骤，欢迎继续吐槽我。

> 1. 安装插件管理器 Vundle
> 2. 插件 & 使用技巧

<!--more-->

## 安装插件管理器 Vundle

要想让vim功能强大，插件是必不少的，安装插件方法有的很多，思路大体上都是复制插件到指定的目录(~/.vim/)。

> 列出了一些安装插件的方法：
> Vundle, NeoBundle, VimPlug, Pathogen
> 我只讲我认为最方便的Vundle，其他请自行百度。
> Vundle的作用是帮助你把需要的插件安装到恰当的位置。

利用git克隆项目，

    git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim

创建或者打开 `~/.vimrc`， 文件内粘贴以下内容：

```plain
set nocompatible              " be iMproved, required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
" alternatively, pass a path where Vundle should install plugins
"call vundle#begin('~/some/path/here')

" let Vundle manage Vundle, required
Plugin 'VundleVim/Vundle.vim'
" theme
Plugin 'morhetz/gruvbox'
Plugin 'vim-airline/vim-airline'
Plugin 'vim-airline/vim-airline-themes'
" beauty
Plugin 'majutsushi/tagbar'
Plugin 'scrooloose/nerdtree'
Plugin 'bronson/vim-trailing-whitespace'
Plugin 'airblade/vim-gitgutter'
Plugin 'yggdroot/indentline'
" edit
Plugin 'ervandew/supertab'
Plugin 'tpope/vim-surround'
Plugin 'godlygeek/tabular'
Plugin 'scrooloose/nerdcommenter'
Plugin 'mattn/emmet-vim'
Plugin 'jiangmiao/auto-pairs'

" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
" To ignore plugin indent changes, instead use:
"filetype plugin on
"
" Brief help
" :PluginList       - lists configured plugins
" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
" :PluginSearch foo - searches for foo; append `!` to refresh local cache
" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
"
" see :h vundle for more details or wiki for FAQ
" Put your non-Plugin stuff after this line

" Basic settings
set nu
set expandtab
set tabstop=4
set shiftwidth=4
set softtabstop=4
set smartindent
set cursorline
set backspace=indent,eol,start

" Theme
set t_Co=256
syntax enable
set background=dark
colorscheme gruvbox

" airline
let g:airline_theme="luna"
let g:airline_powerline_fonts = 1
let g:airline#extensions#tabline#enabled = 1

" Map
let g:user_emmet_leader_key='<C-Y>'
map <C-n> :NERDTreeToggle<CR>
map TT :TagbarToggle<CR>
map \\ \c<space>
```

简单介绍一下，包含两部分，整体是vundle推荐模板，前一部分 （Basic settings 之前）是列出需要载入的插件，一部分是对插件的一些配置，也有我自己的个人偏好。
最后的Map是我自定义的快捷键。先copy上，看看效果。

保存这个文件。打开 vim ，这个时候可能提示缺失一些东西，因为我们还没真正安装，直接回车进入。
输入 `:PluginInstall` 确认安装。等待，可能需要几分钟或者十几分钟。

然后关闭vim后重新打开，世界如此美好。

---

## 插件 & 使用技巧

### Vundle

    Plugin 'VundleVim/Vundle.vim'

插件管理器本身，没啥讲的。

---

### 主题 和 Robbin

    Plugin 'morhetz/gruvbox'
    Plugin 'vim-airline/vim-airline'
    Plugin 'vim-airline/vim-airline-themes'

打开后的最上面的缓存信息和下面两行的各种必要信息，可以说是几乎无所不包，重点是强大，并且好看。
有的下伙伴可能会遇到看不到都是方框，没有三角箭头的问题，具体我还需要定位问题所在，目前我知道 `let g:airline_powerline_fonts = 1` 这个会影响，原因我还不清楚。

---

### 美化界面

    Plugin 'majutsushi/tagbar'
    Plugin 'scrooloose/nerdtree'
    Plugin 'bronson/vim-trailing-whitespace'
    Plugin 'airblade/vim-gitgutter'
    Plugin 'yggdroot/indentline'

tagbar 的快捷键我定制的 是TT，需要ctags进行支持，（先要在目录用 ctags -R 生成索引文件才行）。
nerdtree 的快捷键是 <C-n>, ctrl+n， 可以查看所在文件夹的文件，并且快速切换。
vim-trailing-whitespace， 结尾多空格会提示。
vim-gitgutter， 用git进行版本控制的话，可以在左侧看到变化的行。
indentline， 缩进的时候有醒目一些的字符。

---

### 增强编辑

    Plugin 'ervandew/supertab'
    Plugin 'tpope/vim-surround'
    Plugin 'godlygeek/tabular'
    Plugin 'scrooloose/nerdcommenter'
    Plugin 'mattn/emmet-vim'
    Plugin 'jiangmiao/auto-pairs'

这几个增强编辑的功能不是一两句能讲明白的，百度一下很容易的，而且有一些语言支持的也不全面，你或许可以先安装好，有空再探索一下。我只说两个，surround 和 nerdcommenter

surround用起来和容易处理旁边的一对(){}""等等，方法只需要输入 `ct(` 将最近的一对符号变成() `cs"'` 最近的双引号变成单引号。
nerdcommenter，快速注释和反注释，只需要选好位置，输入 `\\`  (这里我做了Map，原始输入的是 `\c<space>`）

---

## 总结

折腾vim纯属爱好，重要的是还利用vim编程，代码水平并不会因为编辑器提高。
其实更多的时候编程并不会直接就用vim，我也非常不建议用vim进行高强度的项目开发，那么多好用的IDE那么贵也是有原因的，IDE都是很有针对性的，可以很好地完成项目，VIM只是在处理一些琐事，作为一个小而美的工具存在，不要因为喜欢骑行就月月西藏青海新疆。

编程虽苦，但使用这么舒服的工具，也是很欣喜啊。

PS：需要网络不是太好，我可以直接打个包发给你我定制的VIM，邮箱gaoming714@126.com，顺便扫个码。哈哈哈。

更多插件可以查这个这个网站
[Vim Awesome][1]

 [1]: https://vimawesome.com/