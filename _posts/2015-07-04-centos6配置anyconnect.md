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

首先保证系统里已安装openssl、gcc等常用软件

**1.安装编译环境及依赖**

```

yum install -y gcc openssl openssl-devel gmp-devel gmp  readline-devel

```

**2.安装nettle**

```

wget http://ftp.gnu.org/gnu/nettle/nettle-3.1.tar.gz

tar zxf nettle-3.1.tar.gz && cd nettle-3.1

./configure --prefix=/usr && make

make install &&
chmod   -v   755 /usr/lib/lib{hogweed,nettle}.so &&
install -v -m755 -d /usr/share/doc/nettle-3.1  &&
install -v -m644 nettle.html /usr/share/doc/nettle-3.1

```

**3、安装gnutls**

安装libtasn1  

```

cd

wget http://mirror.hust.edu.cn/gnu/libtasn1/libtasn1-4.3.tar.gz

tar zxvf libtasn1-4.3.tar.gz

cd libtasn1-4.3

./configure --prefix=/usr --disable-static &&

make

make install

```

安装gnutls  

```

cd

wget ftp://ftp.gnutls.org/gcrypt/gnutls/v3.4/gnutls-3.4.4.1.tar.xz

xz -d gnutls-3.4.4.1.tar.xz

tar xf gnutls-3.4.4.1.tar

cd gnutls-3.4.4.1

./configure --prefix=/usr \
            --without-p11-kit &&
make

make install

```

**4、安装ocserv**

```html

wget ftp://ftp.infradead.org/pub/ocserv/ocserv-0.10.5.tar.xz

xz -c -d ocserv-0.10.5.tar.xz | tar x

cd ocserv-0.10.5

增大 route 数量限制  
vi /root/ocserv-0.10.5/src/vpn.h
修改:  
#define MAX_CONFIG_ENTRIES 200

./configure && make && make install

```

**5、配置ocserv**

```html

创建ca证书和服务器证书（参考http://www.infradead.org/ocserv/manual.html#heading5）

certtool --generate-privkey --outfile ca-key.pem

cat << _EOF_ >ca.tmpl
cn = "stunnel.info VPN"
organization = "stunnel.info"
serial = 1
expiration_days = 3650
ca
signing_key
cert_signing_key
crl_signing_key
_EOF_

certtool --generate-self-signed --load-privkey ca-key.pem --template ca.tmpl --outfile ca-cert.pem

certtool --generate-privkey --outfile server-key.pem

cat << _EOF_ >server.tmpl
cn = "stunnel.info VPN"
o = "stunnel"
serial = 2
expiration_days = 3650
signing_key
encryption_key #only if the generated key is an RSA one
tls_www_server
_EOF_

certtool --generate-certificate --load-privkey server-key.pem --load-ca-certificate ca-cert.pem --load-ca-privkey ca-key.pem --template server.tmpl --outfile server-cert.pem

```

把证书复制到ocserv的配置目录

```html

mkdir -p /usr/local/etc/ocserv/ ; cp server-cert.pem /usr/local/etc/ocserv/ && cp server-key.pem /usr/local/etc/ocserv/

cp ca-cert.pem /usr/local/etc/ocserv/

```

复制配置文件样本

```html
cp doc/sample.config /usr/local/etc/ocserv/ocserv.conf
```

编辑配置文件

```html
vim /usr/local/etc/ocserv/ocserv.conf
```

修改如下：

```html

auth = "plain[/usr/local/etc/ocserv/ocpasswd]"
ocserv支持多种认证方式，这是自带的密码认证，使用ocpasswd创建密码文件  
ocserv还支持证书认证，可以通过Pluggable Authentication Modules (PAM)使用radius等认证方式  

run-as-group = nobody

isolate-workers = false

max-same-clients = 10
同一个用户最多同时登陆数  

server-cert = /usr/local/etc/ocserv/server-cert.pem
server-key = /usr/local/etc/ocserv/server-key.pem

ca-cert = /usr/local/etc/ocserv/ca-cert.pem

证书路径  

#default-domain = example.com
注释掉这行  

ipv4-network = 192.168.10.0
分配给VPN客户端的IP段  

dns = 192.168.10.1


#route = 192.168.1.0/255.255.255.0
#route = 192.168.5.0/255.255.255.0
注释掉这两行。route参数留空表示所有流量均走VPN。  
ocserv可以给客户端下发路由表。比如可以把公司内网IP段、所有国外IP走VPN出去。  

```

创建认证用的用户文件

```html
ocpasswd -c /usr/local/etc/ocserv/ocpasswd username
username为设置的用户
```

修改系统配置，允许转发

```html
vi /etc/sysctl.conf
修改:/net.ipv4.ip_forward = 1
```

sysctl -p

防火墙设置

```html

iptables -t nat -A POSTROUTING -s 192.168.10.0/24 -o venet0 -j MASQUERADE
iptables -A FORWARD -s 192.168.10.0/24 -j ACCEPT
iptables -A INPUT -p tcp -m state --state NEW --dport 443 -j ACCEPT
iptables -A INPUT -p udp -m state --state NEW --dport 443 -j ACCEPT

service iptables save

IP段和venet0接口要根据自己的情况修改

```

最后运行服务

```html
/usr/local/sbin/ocserv -c /usr/local/etc/ocserv/ocserv.conf
```

开机启动

```html

vi /etc/rc.d/rc.local

添加:
/usr/local/sbin/ocserv -c /usr/local/etc/ocserv/ocserv.conf

```

```html

制作客户端证书
cd   #回到当前用户的目录
certtool --generate-privkey --outfile user-key.pem
cat << _EOF_ >user.tmpl
	cn = "user"
	unit = "admins"
	uid = "anything you want"
	expiration_days = 3650
	signing_key
	tls_www_client
	_EOF_
 
certtool --generate-certificate --load-privkey user-key.pem --load-ca-certificate ca-cert.pem --load-ca-privkey ca-key.pem --template user.tmpl --outfile user-cert.pem

客户端证书 user-cert.pem 还不能直接使用，需通过 OpenSSL转换成 .p12格式
openssl pkcs12 -export -inkey user-key.pem -in user-cert.pem -name "client" -certfile ca-cert.pem -caname "VPN CA" -out user-cert.p12

#按提示设置证书使用密码，或直接回车不设密码
 




```

在iOS上安装Cisco AnyConnect即可连接服务器

Android上也有Cisco AnyConnect（需要root），不过Android可选择的太多，推荐Shadowsocks

Windows、MAC OS也有Cisco的官方客户端

http://bgp.he.net/ AS查询

---  
