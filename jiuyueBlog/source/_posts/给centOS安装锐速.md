---
title: 给centOS安装锐速
date: 2016-12-25 11:53:56
tags: ss,vps
---
> 最近鼓捣服务器玩脱了,导致ssh连不上,端口也没占用,防火墙也没开,就是不行,索性直接重装,正好趁这个机会把以前没有写下的东西记录下来

## 什么是锐速
锐速serverspeeder是一款TCP网络加速软件，能在Linux系统和Windows系统的服务器中安装，安装后能启到提高网络连接稳定性、带宽利用率、低访问失败率等作用，从而提高服务器网络访问速度。锐速并非实际增大服务器带宽，只是提高网络的稳定性和利用率而已。蜗牛在为服务器安装锐速后，测试服务器全球下载、本地上传下载速度变化不大；但使用超级ping发现，丢包现象明显减少。另外一个明显变化就是在同一VPS安装科学上网工具观看YouTube，没安装锐速前观看YouTube 720P视频非常不流畅，经常会出现缓冲现象；而安装锐速后能流畅观看YouTube 720P视频。

## 安装锐速
这里使用网上找到的脚本 直接copy以下命令
```
#如果没有安装wget下载工具,执行这一条进行安装
$ yum -y install wget

#下载脚本
$ wget -N --no-check-certificate https://raw.githubusercontent.com/wn789/serverspeeder/master/serverspeeder.sh

#执行脚本
$ bash serverspeeder.sh
```
安装过程很简单，如果你的VPS内核支持安装，根本无需你手动操作，直接一键完成。

如果你VPS内核没有找到匹配的锐速版本，会自动提示选择接近版本。

当然你还肯会遇到内核不支持的情况，那么需要我们先手动更改可以匹配锐速的内核。目前此破解版锐速支持的内核有：
* CentOS-6.8：2.6.32-642.el7.x86_64
* CentOS-7.2：3.10.0-327.el7.x86_64
* CentOS：4.4.0-x86_64-linode63
* Ubuntu_14.04：4.2.0-35-generic
* Debian_8：3.16.0-4-amd64。

替换内核的方法请看下面:
[为你的CentOS替换内核](#替换内核)


### 锐速serverspeeder常用命令:
```
service serverSpeeder start #启动
service serverSpeeder stop #停止
service serverSpeeder reload #重新加载配置
service serverSpeeder restart #重启
service serverSpeeder status #状态
service serverSpeeder stats #统计
service serverSpeeder renewLic #更新许可文件
service serverSpeeder update #更新
chattr -i /serverspeeder/etc/apx* && /serverspeeder/bin/serverSpeeder.sh uninstall -f #卸载
```
<span id = "替换内核"></span>
## 为你的CentOS替换内核

#### 根据系统版本更换内核
查看自己的系统版本

```
head -n 1 /etc/issue
```
##### CentOS 6.8

CentOS 6支持安装锐速的内核：
2.6.32-504.3.3.el6.x86_64

首先运行下面命令为自己的VPS下载安装内核。
```
#查看当前内核版本
uname -r
#下载内核
wget http://ftp.scientificlinux.org/linux/scientific/6.6/x86_64/updates/security/kernel-2.6.32-504.3.3.el6.x86_64.rpm
#安装内核
rpm -ivh kernel-2.6.32-504.3.3.el6.x86_64.rpm --force
```
##### CentOS 7

```
#查看当前内核版本
uname -r
#下载内核
wget http://ftp.scientificlinux.org/linux/scientific/7.0/x86_64/updates/security/kernel-3.10.0-327.el7.x86_64.rpm
#安装内核
rpm -ivh kernel-3.10.0-327.el7.x86_64.rpm --force

```
> 正常系统执行到这里就可以使用"reboot"命令重启,然后"uname -r"查看是否替换成功了,但是我的vps服务商是linode,原本没问题,这次更换内核发现,安装成功后怎么也无法默认启动这个内核,reboot之后还是原来的内核,如果你也有这样的问题,请接着下面的操作

#### linode调整默认启动内核
Linode 的机器全部用的是定制的内核,并且在控制台强制限制了机器默认启动的内核,所以需要做一些其他工作
##### 更新系统
```
#更新系统,目前会更新到6.9
$ yum install epel-release -y
#更新yum
$ yum update -y
```
##### 安装 grub 引导
```
$ yum install grub -y
```
##### 新建 menu.lst 引导文件
先看看你机器安装的最新内核是什么
```
$ ls -l /boot
```
会显示一大堆东西
```
总用量 98992
-rw-r--r--. 1 root root   106312 12月 16 2014 config-2.6.32-504.3.3.el6.x86_64
-rw-r--r--. 1 root root   108108 2月  24 2017 config-2.6.32-642.15.1.el6.x86_64
-rw-r--r--. 1 root root   108103 5月  10 2016 config-2.6.32-642.el6.x86_64
-rw-r--r--. 1 root root   108169 10月  5 21:27 config-2.6.32-696.13.2.el6.x86_64
drwxr-xr-x. 3 root root     4096 3月   1 2017 efi
drwxr-xr-x. 2 root root     4096 10月 12 09:46 grub
-rw-------. 1 root root 17532160 10月 12 09:43 initramfs-2.6.32-504.3.3.el6.x86_64.img

```
显而易见我们要的是2.6.32-504.3.3.el6.x86_64的内核

输入命令
```
$ nano /boot/grub/menu.lst
```
然后按照下面的格式填写menu.lst中的内容模版,每行的版本号替换成你系统安装的内核版本
```
default 0
timeout 5

title CentOS 2.6.32-504.3.3.el6.x86_64

root (hd0)

kernel /boot/vmlinuz2.6.32-504.3.3.el6.x86_64 root=/dev/sda

initrd /boot/initramfs-2.6.32-504.3.3.el6.x86_64.img
```
如果文件已存在,有多个title时,说明你有安装多个内核,找到对应可用的内核,或者新添加一个,将default改为你需要的顺序即可,default的意思是系统默认启动第几个title
##### Linode 控制面板修改启动方式
对应 VPS 的系统盘 Edit 下的 Kernel 选择 【GRUB(Legacy)】保存后，Reboot
##### 启动后，验证内核
```
$ uname -a
```

执行命令“rpm -qa | grep kernel”，查看内核是否安装成功。如果显示你安装的内核版本，表示安装成功。
```
rpm -qa | grep kernel
```

重启VPS，查看内核是否修改成功。
```
reboot #重启VPS
uname -r #当前使用内核版本
```
