---
layout:     post
title:      CentOS安装DNS服务dnsmasq
date:       2015-07-04
tags:
    - CentOS
    - dnsmasq
    - DNS
---

# CentOS安装DNS服务dnsmasq

dnsmasq是个非常小巧的dns服务器，可以解决小范围的dns查询问题，建议内网终端不要超过50台主机为佳。

优先使用本地自定义dns和host，在中国这个网络格局下，非常实用。

提供dhcp服务，方便内网主机和移动设备管理。

1.编译安装dnsmasq

```

wget http://www.thekelleys.org.uk/dnsmasq/dnsmasq-2.71.tar.gz  
tar -zxvf dnsmasq-2.71.tar.gz && cd dnsmasq-2.71  
make install  

 
cp dnsmasq.conf.example /etc/dnsmasq.conf  
mkdir -p /etc/dnsmasq.d   #这个目录备用

```

2.dnsmasq配置

主要有三个文件：

```

/etc/dnsmasq.conf
/etc/dnsmasq.d/resolv.dnsmasq.conf
/etc/dnsmasq.d/dnsmasq.hosts

```

第一个是系统默认必须的，后面两个可以自行建立，放置的路径也可以根据自己需要定义。

vi /etc/dnsmasq.conf  

自带的配置文件有很多说明，可以直接在上部加入以下内容，保留之前的可以当帮助文件用，也可直接删除原先的内容。

```

#conf-dir=/etc/dnsmasq.d
 
#让dnsmasq读取你设定的resolv-file
#no-resolv
resolv-file=/etc/dnsmasq.d/resolv.dnsmasq.conf
 
no-poll
strict-order
 
#不读取系统hosts，读取你设定的
no-hosts
addn-hosts=/etc/dnsmasq.d/dnsmasq.hosts
 
#dnsmasq日志设置
log-queries
log-facility=/var/log/dnsmasq.log
 
#dnsmasq缓存设置
cache-size=1024
 
#单设置127只为本机使用，加入本机IP为内部全网使用(若配合ocserv使用，此处的本机IP应为VPN网络下的本机IP)
listen-address=127.0.0.1,192.168.10.1

```

在/etc/dnsmasq.d目录下新建2个文件
vi /etc/dnsmasq.d/resolv.dnsmasq.conf

此处的DNS应根据本机所在位置决定，我的服务器在搬瓦工佛罗里达机房，国内DNS服务器这两个相应时间最短。  
之所以采用国内DNS，因为网站是根据DNS服务器的位置来分配CDN的，edns-client-subnet在此处没有作用，因为发起请求的是国外的VPS。

```

nameserver 219.150.32.132
nameserver 219.141.140.10

```

vi /etc/dnsmasq.d/dnsmasq.hosts  
给出以下样本,可以根据需求自己制作。

```

173.194.120.88   0.docs.google.com
216.239.32.39    0.docs.google.com
173.194.120.88   0.drive.google.com
216.239.32.39    0.drive.google.com

```

dnsmasq启动脚本  
编译安装比较麻烦的就是这件事了，当然也可直接手动操作：

启动： /usr/local/sbin/dnsmasq
验证：netstat -tunlp|grep 53
关闭：killall -KILL dnsmasq
重启： pkill -9 dnsmasp && /usr/local/sbin/dnsmasq -h

验证是否正确启动

```

netstat -tunlp|grep 53

```

验证是否正确工作  
需要命令dig和nslookup，如果没有，安装一下。

```

yum install bind-utils

```

记得在iptables防火墙开放53端口，tcp和udp都要开

```

vi /etc/sysconfig/iptables

-A INPUT -p udp -m state --state NEW --dport 53 -j ACCEPT
-A INPUT -p tcp -m state --state NEW --dport 53 -j ACCEPT

```

/etc/dnsmasq.conf中dns配置范例

如果你想使用dnsmasq的泛解析功能，这里提供一些范例参考。

设定某些网址使用国内的DNS服务器，这里选用了天津电信服务器，你也可以选择适合你的最快DNS服务器地址。  
而被污染的域名可以使用国外的DNS服务器来解析。

```

#Specify DNS server China
server=/baidu.com/219.150.32.132
server=/cn/219.150.32.132
server=/taobao.com/219.150.32.132
server=/weibo.com/219.150.32.132

#Specify DNS server out of china
server=/google.com/8.8.4.4
server=/twitter.com/8.8.4.4
server=/facebook.com/8.8.4.4

```







