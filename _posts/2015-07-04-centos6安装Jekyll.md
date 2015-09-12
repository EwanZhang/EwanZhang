---
layout:     post
title:      CentOS6安装Jekyll
date:       2015-07-04
tags:
    - CentOS
    - Jekyll
---

# CentOS6安装Jekyll


环境：CentOS 6

**编译安装Node.js**  
Jekyll是基于Ruby开发的，用到了Ruby的execjs方法来执行JavaScript代码，而这需要自己指定一个JavaScript runtime；这里我们选择安装Node.js。  
`sudo yum install libtool automake autoconf gcc-c++ openssl-devel wget`

`mkdir ~/soft/`  
`cd ~/soft/`  

`wget http://nodejs.org/dist/v0.12.4/node-v0.12.4.tar.gz`  
`tar -zxvf node-v0.12.4.tar.gz`  
`cd node-v0.12.4`  

`./configure --prefix=/usr`  
`make && sudo make install`  

`node -v`  
`npm -v`  

**安装Ruby，使用rvm管理Ruby版本**  
`gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3`  
`\curl -sSL https://get.rvm.io | bash -s stable`  

`/etc/profile.d/rvm.sh`  
`rvm install 2.2.1`(可能需要重新打开shell 窗口)  

**安装额外的Ruby包和文档:**  
`yum install -y ruby-devel ruby-docs ruby-ri ruby-rdoc` 

**安装RubyGems**  
`yum install -y rubygems`  

**使用RubyGems安装Jekyll**  
`gem install jekyll`
