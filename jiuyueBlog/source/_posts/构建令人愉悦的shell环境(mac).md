---
title: 构建令人愉悦的shell环境(mac)
date: 2017-11-16 14:38:56
tags: Mac,Linux
---
> 工作中或多或少会用到命令行,无论你使用的是windows,Mac或者是Liunx,都应当拥有一个令人愉悦的命令行环境,让枯燥的命令行操作变得生动,易于上手.

### 基本知识小科普
#### 什么是Shell
shell 英文翻译为壳
在windows的CMD以及Mac OS的Terminal应当理解为命令行解释器 (Command Line Interface shell).
他是用来连接命令行与系统的,用来管理你和操作系统间的交互.

#### 什么是Bash
bash 是一个为GNU项目编写的Unix shell.
在Mac 或 Linux,这些基于Unix的系统的Shell版本都为Bash.

#### 为什么要换掉Bash
在Linux中默认的Shell版本为Bash,而我们使用Linux的场景一般往往是连接服务器,所以也就懒得更换掉这个默认的Shell了.
但是在Mac OS中,系统默认是预装了除了Bash之外的其他版本Shell的,而Mac OS我们的使用场景往往是自己的工作机,所以替换一个舒适的Shell环境大有可为.
这里我推荐的是zsh.
###### 什么是zsh
zsh 是一个完全兼容了Bash,且更强大,使用更舒服的Shell.
即便你熟悉了Bash的操作,也可以不需要很多学习成本的切换过来
zsh的好处有很多,列举几点
* zsh 的补全模式很方便 按两下TAB键可以触发zsh的补全,所有的补全选项都可以通过方向键或者Ctrl-n/p/f/b来选择
* zsh 支持命令选项补全 例如ls - 连按两下TAB键,即可出现ls的所有选项供你选择
* zsh 支持命令参数补全 例如kill进程,原本需要根据进程名查询进程ID,然后记下ID,再kill掉进程,而zsh可以做到只要kill <进程名> 它会自动补全进程的ID
* zsh 强大的快速目录切换 经常会有的场景是你需要在两个目录下不停切换,总要CD很长的路径,zsh可以根据1-9的数字,快速访问你的历史访问路径.还可以通过d来查看目录访问历史

还有什么目录补全之类的好处就不多赘述了,总之是一个使用起来很舒服的Shell.
###### 什么是oh-my-zsh
但是zsh 的默认配置及其复杂繁琐,新上手的同学推荐使用oh-my-zsh
oh-my-zsh 它基于 zsh 命令行，提供了主题配置，插件机制，已经内置的便捷操作。给我们一种全新的方式使用命令行。

#### iTerm
iTerm是用来替换Mac下命令行工具Terminal(终端)的,因为他的功能比Terminal更强大,我目前用到最实际的感受就是分屏功能确实爽,可以拆出很多个小的命令行窗口,并且各种快捷键操作非常舒服.

安装iTerm 只需要到官网下载,正常安装即可.
Windows下可以使用xShell替换CMD

### 替换开始
Mac OS 首先安装iTerm

#### 切换Shell
首先查看本地安装的Shell
```
$ cat /etc/shells
```
Mac OS默认安装了如下6个Shell
```
/bin/bash
/bin/csh
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh
```
正常的切换到oh-my-zsh的步骤应当是
1. clone oh-my-zsh的git库
2. 备份zshrc文件
3. 替换新的zshrc文件为oh-my-zsh

这里我们直接使用脚本一键替换
```
# 这个脚本为我们做了上述的事情
$ curl -L https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh | sh
```
切换Shell为zsh
```
$ chsh -s /bin/zsh
```
安装完成,重启iTerm.

#### 美化shell环境
##### iTerm美化
我是用的主题是 [Tomorrow](https://github.com/chriskempson/tomorrow-theme)
下载Tomorrow并解压.
打开iTerm -> Preferences -> profiles -> Colors标签 ->右下角Color Presets ->import
选择解压后的Tomorrow目录,找到目录下iTerm2文件夹,里面有5个主题文件,全选,导入.
再次选择右下角Color Presets,会发现导入的主题已经出现在下面,我用的是Tomorrow Night Eighties.

##### oh-my-zsh美化
oh-my-zsh也是可以使用主题的
我是用的是ys主题
本地主题可以在 ~/.oh-my-zsh/themes/中看到
主题的样式可以去[oh-my-zsh官网主题列表](https://github.com/robbyrussell/oh-my-zsh/wiki/themes)查看,官网还提供了除了141个自带主题之外的其他主题.
我用的是自带的ys主题,因为有些主题为了好看,需要用到一些特殊字符,还要导入指定的字体,我嫌麻烦.
想要修改主题也很简单
```
# 编辑配置文件
$ vim ~/.zshrc
# 修改ZSH_THEME字段 ZSH_THEME="主题名"
ZSH_THEME="ys"
# 保存并退出
$ wq
```
重启iTerm

##### vim 美化
我们美化了命令行工具,美化了Shell,但是最常用的Vim却不在他们美化的范围内,所以我们需要对vim处理一下,不然Vim打开的文件,都是纯白字色,辨识度很低.
打开刚才下载的iTerm的Tomorrow主题文件夹
> 没错,tomorrow主题不仅仅支持iTerm,还支持Atom,Eclipse,Xcode,Visual Studio,notepad++,OS X Terminal等等多种工具.

找到vim文件夹 -> colors
复制里面的vim主题文件到 ~/.vim/colors文件夹下
编辑vim配置文件 ~/.vimrc  (mac 系统下默认好像是没有的,需要自己创建)
```
# 我使用的依然是 Tomorrow-Night-Eighties
colorscheme Tomorrow-Night-Eighties
```
以下是我的配置文件信息,我加入了注释,可以直接复制我的配置文件使用

```
$ vim ~/.vimrc

" Configuration file for vim
set modelines=0     " CVE-2007-2438
" 语法高亮
syntax on
" 主题
colorscheme Tomorrow-Night-Eighties
" 显示行号
set number

" 自动缩进
set autoindent
set cindent
" Tab键的宽度
set tabstop=4
" 统一缩进为4
set softtabstop=4
set shiftwidth=4


"搜索逐字符高亮
 set hlsearch
 set incsearch

 " Normally we use vim-extensions. If you want true vi-compatibility
 " remove change the following statements
 " Use Vim defaults instead of 100% vi compatibility
 " 不使用vi的键盘模式 使用vim自己的
 set nocompatible
 " more powerful backspacing
 set backspace=2

 " Don't write backup file if vim is being called by "crontab -e"
 au BufWrite /private/tmp/crontab.* set nowritebackup nobackup
 " Don't write backup file if vim is being called by "chpass"
 au BufWrite /private/etc/pw.* set nowritebackup nobackup
 let skip_defaults_vim=1
```
ps:开启行号是个很纠结的事,因为复制的时候会把行号也复制出来,但是不开启又看着不舒服.自由取舍吧.

以下是我的iTerm截图
![iTerm截图](/img/1510820449852.jpg)
![iTerm截图](/img/1510820508851.jpg)
![iTerm截图](/img/1510820994873.jpg)

祝你愉快!
