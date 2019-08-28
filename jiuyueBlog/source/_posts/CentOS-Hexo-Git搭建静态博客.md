---
title: CentOS+Hexo+Git搭建静态博客
date: 2017-5-10 16:27:25
tags: hexo静态博客
---
#### 1. 使用EPEL安装nodeJs
>  EPEL（Extra Packages for Enterprise Linux）企业版Linux的额外软件包，是Fedora小组维护的一个软件仓库项目，为RHEL/CentOS提供他们默认不提供的软件包。

因为Hexo是跑在nodejs环境下的,所以要装nodejs环境

首先查看当前系统下是否安装了nodeJs
```
$ node -v
```
如果没有安装则继续下面的步骤
```
# 先确认系统是否已经安装了epel-release包：
$ yum info epel-release
```
如果有输出有关epel-release的已安装信息，则说明已经安装，如果提示没有安装或可安装，则安装
```
$ sudo yum install epel-release
```
安装完后，就可以使用yum命令安装nodejs了，安装的一般会是较新的版本，并且会将npm作为依赖包一起安装
```
$ sudo yum install nodejs
```
安装完成后，验证是否正确的安装，node -v，如果输出版本信息，说明成功安装
```
$ node -v
```
#### 2.安装git
查看系统是否安装了git
```
$ git --version
```
如果没有版本信息,则继续下面的步骤
```
$ yum install git
```
然后使用yum –version命令查看是否有版本信息,有版本信息则安装成功

然后配置一下全局的git用户信息,先查看配置信息
```
$ git config --list
```
如果没有配置过,则
```
$  git config --global user.name "输入你的用户名"
$  git config --global user.email "输入你的邮箱"
```
#### 3.安装Hexo
终于到了安装Hexo的一步了
```
$ npm install hexo-cli -g
```
一个命令搞定,然后建立博客目录
```
$ hexo init "blogPath"(你想要存放博客环境的路径)
```
现在进入博客目录
```
$ cd blogPath
```
然后安装server服务跑起来看一下是否成功

在Hexo 3.0 后server被单独出来了，需要安装server，安装的命令如下：
```
$ npm install hexo-server --save
```
安装此server后执行
```
$ hexo server
```
或者 ‘hexo s’ 这是上面命令的缩写,这个命令主要是用于让Hexo在本地运行一个虚拟服务器环境,可以看到修改的效果

如果你是在远程的服务器在浏览器访问
```
http://你的服务器IP:4000
```
如果你是本地,则访问
```
http://localhost:4000
```
页面显示出来则成功建立了一个博客

#### 4.配置Hexo
Hexo的配置文件是在博客目录下的_config.yml文件,在博客目录下执行
```
$ vim _config.yml
```
最上面呈现的是博客的基本信息
```
title: #网站标题
subtitle: #子标题
description: #博客简洁
author: #博客作者
language: zh-Hans #使用的语言环境 zh-Hans为中文
email: #邮箱
keywords: #关键字
```
关键点在于后面的这些信息,一会有用到,先混个脸熟
```
...
theme: #使用模板
...
deploy: #部署信息配置
type: git #部署类型
repository: xxxx #你的git仓库地址
branch: master #推送的分支
```
#### 5.创建Repository
##### 登录github,创建一个新的Repository
命名规则为’你的github用户名.github.io’ 一定要按照这个规则命名

创建成功后copy这个仓库的SSH地址
```
大致长这个样子
git@github.com:用户名/用户名.github.io.git
```
然后编辑上面说到的Hexo配置文件,找到部署信息配置,在repository后面加上刚刚复制的地址(冒号后面记得加空格),其他字段按照上面的示例配置.

然后保存配置文件
```
esc键 然后直接输入:wq 就保存了
```
##### 设置SSH key
检验是否已存在key

分别执行命令
```
$ cd ~
$ cd  .ssh
```
再执行命令 ls 查看是有已有key文件,一般存在key的话都会显示id_rsa.pub 和 id_dsa.pub这两个文件,没有key什么都不会显示

添加一个 SSH key
```
# email为你的github登录邮箱
$ ssh-keygen -t rsa -C "your_email@mail.com"
```
会提示你指定文件名,如果什么都不输入,直接回车,会默认使用id_rsa.pub为ssh key文件名

回车之后,需要输入两次密码,这个密码是你push文件的时候要输入的密码，而不是github的密码,如果不打算使用密码,直接回车就好,然后就会看到ssh key添加成功

然后使用命令显示ssh key内容
```
$ cat /roor/.ssh/id_rsa.pub
```
会出来一大坨东西,直接全部复制,然后到你的github账户

点击右上角你的头像,选择’settings’,选择左边的’SSH and GPG keys’ 在SSH keys区域,选择New SSH key,然后把刚刚复制的粘贴进,保存,就OK了

回到命令行,输入下面命令,测试是否添加正确

```
$ ssh -T git@github.com
```
如果成功,他会显示
```
Hi xxx! You've successfully authenticated, but GitHub does not provide shell access.
```
到这里,git上的所有操作,基本结束.

#### 6.部署到github
Hexo提供了两个命令,先使用generate生成静态文件,然后使用deploy命令,他会根据配置文件里,我们填入的git信息,执行push

```
$ hexo generate
$ hexo deploy
```
实际使用中每次都要写两个命令太麻烦了,所以Hexo提供了缩写命令’g -d’ 生成后,直接推送到git

```
$ hexo g -d
```
等待命令执行完毕后,可以查看代码是否已提交到github上,然后在浏览器输入’你的用户名.github.io’就可以访问了

#### 7.使用Hexo写博客
##### 删除博客

默认Hexo会有一片hello,word的博客示例
正好拿来练手

删除博客需要进入博客根目录/source/_posts目录中删除文件后执行下面的命令就好了

```
$ hexo clean
```
##### 新建博客

在博客目录下,使用以下命令,即可在/source/_posts目录下新建一个.md文件

```
$ hexo new 文章名
```
找到这个文件,对他进行编辑就可以了.

#### 8. 使用Hexo主题

我这里用的是大名鼎鼎的Next,我就用Next举例子了,其他的主题,可以看他们对应的说明,都会有教程的

cd到博客目录下,使用下面的Git命令,将next拉到本地

```
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```
编辑Hexo配置文件,将theme一项后面改为next(记得冒号后面加空格)

执行 hexo s –debug 开启本地站点,并进入调试模式,看下效果

next提供了三种风格,关于next的配置,需要到博客目录/themes/next下面 找到_config.yml文件进行配置,具体配置到next的官网看就可以了.

##### 注意

* themes目录是Hexo的主题模板文件夹
* Hexo的配置文件命名风格都是以_config.yml命名,模板的_config.yml和Hexo的_config.yml是两回事,千万不要改错了!!!

```
hexo配置文件路径
博客目录/_config.yml
模板配置文件路径
博客目录/themes/对应的模板/_config.yml
```
大致的从搭建到使用的步骤就是这些.

从上周开始我尝试了wordpress,hexo和jekyll.也踩了很多坑,文字太多,没办法一一表述.

虽然最后都解决了,并且可以正常运行了,但是犹豫再三还是放弃了

最终选择了Hexo,其实主要还是觉得利用上github比较酷,感觉很专业

国内还可以选择coding替代github,coding可以选择北京和香港的节点,比github要快,需要的自行百度

我个人也很喜欢jekyll,还有喵神写的那个他博客用的模板.
