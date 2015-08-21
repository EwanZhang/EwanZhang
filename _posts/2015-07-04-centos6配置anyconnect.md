---
layout:     post
title:      CentOS6配置AnyConnec
date:       2015-07-04
tags:
    - CentOS
    - AnyConnec
---

# CentOS6配置AnyConnec


环境：CentOS 6

ocserv需要3.1版以上的gnutls，gnutls需要2.7版以上的nettle

这两个在repo仓库里均没有，所以我们自己编译

首先保证系统里已安装openssl、gcc、make等常用软件

**1.安装编译环境及依赖，如部分软件不能安装请先安装epel源**

`rpm -ivh http://dl.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm`

`rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm`

`yum install pam-devel readline-devel http-parser-devel`

`yum install tar gzip xz wget gcc make autoconf `

`yum install openssl openssl-devel`

**2.编译nettle**

安装gmp

`yum install gmp-devel gmp`

`wget http://ftp.gnu.org/gnu/nettle/nettle-3.1.tar.gz`

`tar zxf nettle-3.1.tar.gz && cd nettle-3.1`

`./configure --prefix=/usr &&`

`make`

`make install &&`
`chmod   -v   755 /usr/lib/lib{hogweed,nettle}.so &&`  
`install -v -m755 -d /usr/share/doc/nettle-3.1.1  &&`  
`install -v -m644 nettle.html /usr/share/doc/nettle-3.1.1`  

**2、编译unbound**

`cd`

安装expat-devel

`yum install expat-devel`

`wget http://unbound.nlnetlabs.nl/downloads/unbound-1.5.4.tar.gz`

`tar zxf unbound-1.5.4.tar.gz && cd unbound-1.5.4`

`groupadd -g 88 unbound &&`  
`useradd -c "Unbound DNS resolver" -d /var/lib/unbound -u 88 \`  
`        -g unbound -s /bin/false unbound`  
`./configure --prefix=/usr     \`  
`            --sysconfdir=/etc \`  
`            --disable-static  \`  
`            --with-pidfile=/run/unbound.pid &&`  
`make`

`make install &&`  
`mv -v /usr/sbin/unbound-host /usr/bin/`

**3、编译gnutls**

安装libgpg-error:
`cd`

`wget ftp://ftp.gnupg.org/gcrypt/libgpg-error/libgpg-error-1.11.tar.gz`

`tar zxvf libgpg-error-1.11.tar.gz`

`cd libgpg-error-1.11`

`./configure`

`make`

`make install`

安装libgcrypt:

`cd`

`wget ftp://ftp.gnupg.org/gcrypt/libgcrypt/libgcrypt-1.6.3.tar.bz2`

`tar jxvf libgcrypt-1.6.3.tar.bz2`

`cd libgcrypt-1.6.3`

`./configure --prefix=/usr &&`

`make`

`make install &&`  
`install -v -dm755   /usr/share/doc/libgcrypt-1.6.3 &&`  
`install -v -m644    README doc/{README.apichanges,fips*,libgcrypt*} \`  
`                    /usr/share/doc/libgcrypt-1.6.3`  


安装libtasn1

`cd`

`wget http://mirror.hust.edu.cn/gnu/libtasn1/libtasn1-4.3.tar.gz`

`tar zxvf libtasn1-4.3.tar.gz`

`cd libtasn1-4.3`

`./configure --prefix=/usr --disable-static &&`

`make`

`make install`


安装gnutls

`cd`

`wget ftp://ftp.gnutls.org/gcrypt/gnutls/v3.4/gnutls-3.4.4.1.tar.xz`

`xz -d gnutls-3.4.4.1.tar.xz`

`tar xf gnutls-3.4.4.1.tar`

`cd gnutls-3.4.4.1`

`./configure --prefix=/usr \`  
`            --without-p11-kit &&`  
`make`

`make install`

**4、编译ocserv**

`wget ftp://ftp.infradead.org/pub/ocserv/ocserv-0.10.5.tar.xz`

`xz -c -d ocserv-0.10.5.tar.xz | tar x`

`cd ocserv-0.10.5`

增大 route 数量限制  
`vi /root/nettle-2.7.1/unbound-1.4.22/gnutls-3.2.12/ocserv-0.10.5/src/vpn.h`  
修改:  
`#define MAX_CONFIG_ENTRIES 200`

`./configure && make && make install`

**5、配置ocserv**

创建ca证书和服务器证书（参考http://www.infradead.org/ocserv/manual.html#heading5）

`certtool --generate-privkey --outfile ca-key.pem`

`cat << _EOF_ >ca.tmpl`  
`cn = "stunnel.info VPN"`  
`organization = "stunnel.info"`  
`serial = 1`  
`expiration_days = 365`  
`ca`  
`signing_key`  
`cert_signing_key`  
`crl_signing_key`  
`_EOF_`  

`certtool --generate-self-signed --load-privkey ca-key.pem --template ca.tmpl --outfile ca-cert.pem`

`certtool --generate-privkey --outfile server-key.pem`

`cat << _EOF_ >server.tmpl`  
`cn = "stunnel.info VPN"`  
`o = "stunnel"`  
`serial = 2`  
`expiration_days = 365`  
`signing_key`  
`encryption_key #only if the generated key is an RSA one`  
`tls_www_server`  
`_EOF_`

`certtool --generate-certificate --load-privkey server-key.pem --load-ca-certificate ca-cert.pem --load-ca-privkey ca-key.pem --template server.tmpl --outfile server-cert.pem`

把证书复制到ocserv的配置目录

`mkdir -p /usr/local/etc/ocserv/ ; cp server-cert.pem /usr/local/etc/ocserv/ && cp server-key.pem /usr/local/etc/ocserv/`

复制配置文件样本

`cp doc/sample.config /usr/local/etc/ocserv/ocserv.conf`

编辑配置文件

`vim /usr/local/etc/ocserv/ocserv.conf`

修改如下：

`auth = "plain[/usr/local/etc/ocserv/ocpasswd]"`  
ocserv支持多种认证方式，这是自带的密码认证，使用ocpasswd创建密码文件  
ocserv还支持证书认证，可以通过Pluggable Authentication Modules (PAM)使用radius等认证方式  

`run-as-group = nobody`

`isolate-workers = false `

`max-same-clients = 10`  
同一个用户最多同时登陆数  

`server-cert = /usr/local/etc/ocserv/server-cert.pem`

`server-key = /usr/local/etc/ocserv/server-key.pem`

证书路径  

`#default-domain = example.com`  
注释掉这行  

`ipv4-network = 192.168.10.0`  
分配给VPN客户端的IP段  

`dns = 8.8.8.8`

`dns = 8.8.4.4`

`#route = 192.168.1.0/255.255.255.0`  
`#route = 192.168.5.0/255.255.255.0`  
注释掉这两行。route参数留空表示所有流量均走VPN。  
ocserv可以给客户端下发路由表。比如可以把公司内网IP段、所有国外IP走VPN出去。  

创建认证用的用户文件

`ocpasswd -c /usr/local/etc/ocserv/ocpasswd username`  
username为设置的用户  

修改系统配置，允许转发

注意把网卡接口名称改成你服务器上对应的接口  

`vi /etc/sysctl.conf`  
修改:`/net.ipv4.ip_forward = 1`  

`sysctl -p`

`iptables -t nat -A POSTROUTING -s 192.168.10.0/24 -o venet0 -j MASQUERADE`  
`iptables -A FORWARD -s 192.168.10.0/24 -j ACCEPT`  
`iptables -A INPUT -p tcp -m state --state NEW --dport 443 -j ACCEPT`  
`iptables -A INPUT -p udp -m state --state NEW --dport 443 -j ACCEPT`  

`service iptables save`

IP段和venet0接口要根据自己的情况修改

最后运行服务

`/usr/local/sbin/ocserv -c /usr/local/etc/ocserv/ocserv.conf`

开机启动

`vi /etc/rc.d/rc.local `

添加:`/usr/local/sbin/ocserv -c /usr/local/etc/ocserv/ocserv.conf`

在iOS上安装Cisco AnyConnect即可连接服务器

Android上也有Cisco AnyConnect（需要root），不过Android可选择的太多，推荐Shadowsocks

Windows、MAC OS也有Cisco的官方客户端

---  
