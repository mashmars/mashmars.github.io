---
title: Centos7yum安装nginx+php7+mysql
date: 2019-05-24 09:40:52
tags: 
    - lnmp
    - php
    - nginx
categories: nginx
---
### 安装nginx
#### 设置nginx安装源
``` 
vim /etc/yum.repos.d/nginx.repo
```
#### 输入以下配置
```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```
#### 查看nginx版本
```
yum list nginx
# nginx.x86_64 1:1.16.....
```
#### 安装nginx
```
yum -y install nginx
```
#### 查看安装版本和configure参数
```
nginx -v | nginx -V
```
#### 启动nginx
```
systemctl start nginx 
```
#### 查看nginx状态
```
systemctl status nginx
```
#### 停止nginx
```
systemctl stop nginx
```
#### 重新加载配置信息(不停止服务)
```
systemctl raload nginx
```
#### 设置开机启动
```
systemctl enable nginx
```
#### 取消开机启动
```
systemctl disable nginx
```
#### 查看是否设置开机启动
```
systemctl is-enabled nginx
```
### 安装mysql
#### 设置mysql5.7安装源
```
rpm -Uvh  http://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
```
#### 安装mysql
```
yum -y install mysql-community-srver
```
#### 启动|停止|重新|开机启动|状态 mysql 
```
systemctl start | stop | restart | enable |status mysqld
```
#### mysql安装完成之后，在/var/log/mysqld.log文件中给root生成了一个默认密码。通过下面的方式找到root默认密码，然后登录mysql进行修改
```
cat /var/log/mysqld.log | grep "A temporary password is generated for root"
2019-01-03T05:41:47.164940Z 1 [Note] A temporary password is generated for root@localhost: zMnep.TsF3tE
```
#### zMnep.TsF3tE便是root密码，修改root密码
```
mysql -uroot -pzMnep.TsF3tE
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.24

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'jmsite.cn';
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'Jmsite.cn:80';
Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye
```
#### 设置编码
```
vim /etc/my.cnf
```
#### 如下设置
```
[mysqld]
character_set_server=utf8
init_connect='SET NAMES utf8'
```
#### 重启mysql
```
systemctl restart mysqld
```
### 安装php7.2
#### 设置centos7的php7安装源
```
rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
```
#### 安装php7.2和各种扩展
```
yum install php72w php72w-cli php72w-common php72w-devel php72w-embedded php72w-fpm php72w-gd php72w-mbstring php72w-mysqlnd php72w-opcache php72w-pdo php72w-xml
```
#### 启动php
```
systemctl start php-fpm
```
#### 设置开机启动
```
systemctl enable php-fpm
```
### 配置nginx支持php
#### 修改配置
```
vi /etc/nginx/conf.d/default.conf
```
#### server段中去掉下面的注释,并更改成如下配置
```
 location ~ \.php$ {
        root           /usr/share/nginx/html;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
```
#### 创建测试php文件
```
vim /usr/share/nginx/html/phpinfo.php
```
#### 重新加载nginx配置
```
systemctl reload nginx
```
#### 浏览器输入:你的ip地址/phpinfo.php



