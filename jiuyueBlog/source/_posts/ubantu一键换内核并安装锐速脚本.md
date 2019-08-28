---
title: ubantu一键换内核并安装锐速脚本(收藏)
date: 2017-10-19 17:04:40
tags: ss,vps
---
{% note danger %}
* 没有任何多余的判断，非ubuntu16.04和ubuntu14.04请勿运行
* 安装完会自动重启 正常现象 重启后就安装好了锐速，可以使用ps aux | grep appex来检测是否运行
* 如果锐速没有自动安装可以使用/appex/appexinstall.sh来安装
{% endnote %}


#### 说明
查看ubantu系统内核,非ubuntu16.04和ubuntu14.04请勿继续运行
```
$ dpkg --list | grep linux-image
```
请将[脚本内容](#脚本内容)中的代码复制,保存为ruisu.sh 存放到ubuntu中,然后执行下面的命令
#### 执行脚本
```
$ bash ruisu.sh
```
<span id = "脚本内容"></span>
#### 脚本内容
```
#!/bin/bash

# Ubuntu/14.04/3.16.0-43-generic/x64
# Ubuntu/16.04/4.4.0-47-generic/x64

[ -n "`cat /etc/issue | grep "Ubuntu 16.04"`" ] && echo "Ubuntu 16.04" && KER_VER="4.4.0-47-generic"
[ -n "`cat /etc/issue | grep "Ubuntu 14.04"`" ] && echo "Ubuntu 14.04" && KER_VER="3.16.0-43-generic"

cp /etc/default/grub /etc/default/grub.old
sed -ir "s/GRUB_DEFAULT=.*/GRUB_DEFAULT=\"Advanced options for Ubuntu>Ubuntu, with Linux $KER_VER\"/g" /etc/default/grub
# update-grub
apt-get update
apt-get install -y linux-image-extra-$KER_VER
# reboot

mkdir -p /appex
cat > /appex/appexinstall.sh << TEMPEOF
wget --no-check-certificate -qO /tmp/appex.sh "https://raw.githubusercontent.com/0oVicero0/serverSpeeder_Install/master/appex.sh" && bash /tmp/appex.sh 'install' << EOF

EOF
cp /etc/rc.local.old.ruisu /etc/rc.local
rm /etc/rc.local.old.ruisu
TEMPEOF

# bash /appex/appexinstall.sh
# ps aux | grep appex


if [ ! -f "/etc/rc.local.old.ruisu" ]; then
echo "first time run this script, backup the rc.local"
cp /etc/rc.local /etc/rc.local.old.ruisu
fi
# cp /etc/rc.local /etc/rc.local.old.ruisu
sed -i '$d' /etc/rc.local
echo "bash /appex/appexinstall.sh" >> /etc/rc.local
echo "exit 0" >> /etc/rc.local
reboot

```

