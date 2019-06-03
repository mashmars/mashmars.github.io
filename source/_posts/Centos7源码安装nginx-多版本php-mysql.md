---
title: Centos7源码安装nginx+多版本php+mysql
date: 2019-05-28 17:36:50
tags: 
    - lnmp
    - php
    - nginx
categories: nginx
---
### nginx源码安装
#### 准备条件 安装相关类库 yum -y install gcc gcc-c++ automake autoconf libtool make
#### 准备pcre(重写rewrite),zlib(压缩),openssl,
```
mkdir /mash
cd /mash
```
#### 安装pcre库
从官方网站下载最新的pcre源码包 8.43，使用下面命令下载编译和安装
```
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.43.tar.gz 
tar -zxvf pcre-8.43.tar.gz
cd pcre-8.34
./configure
make
make install
```
#### 安装zlib库
http://zlib.net/zlib-1.2.11.tar.gz 下载最新的 zlib 源码包，使用下面命令下载编译和安装 zlib包：
```
wget http://zlib.net/zlib-1.2.11.tar.gz
tar -zxvf zlib-1.2.11.tar.gz
cd zlib-1.2.8
./configure
make
make install
```
#### 安装ssl
```
wget https://www.openssl.org/source/openssl-1.1.1.tar.gz
tar -zxvf openssl-1.1.1.tar.gz
```
#### 安装nginx
```
wget http://nginx.org/download/nginx-1.16.0.tar.gz
tar -zxvf nginx-1.16.0.tar.gz
cd nginx-1.16.0
 
./configure --sbin-path=/usr/local/nginx/nginx \
--conf-path=/usr/local/nginx/nginx.conf \
--pid-path=/usr/local/nginx/nginx.pid \
--with-http_ssl_module \
--with-pcre=/mash/pcre-8.43 \
--with-zlib=/mash/zlib-1.2.11 \
--with-openssl=/mash/openssl-1.1.1
 
make
make install
```
--with-pcre=/usr/src/pcre-8.43 指的是pcre-8.43 的源码路径。
--with-zlib=/usr/src/zlib-1.2.11 指的是zlib-1.2.11 的源码路径。
#### 启动ngxin
确保系统的80短裤没有被占用
```
netstat -ano | grep 80 
kill port
```
运行下面命令启动nginx
```
/usr/local/nginx/sbin/nginx
```
打开浏览器访问此机器的 IP，如果浏览器出现 Welcome to nginx! 则表示 Nginx 已经安装并运行成功。
#### 添加nginx服务
```
vi /lib/systemd/system/nginx.service

# 写入下面内容

[Unit]
Description=nginx
After=network.target
[Service]
Type=forking
ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true #如果使用sock,请将此处更改为false
[Install]
WantedBy=multi-user.target

# 重新加载
systemctl daemon-reload
# 测试是否可用
sytemctl start | stop | restart | status nginx
# 设置开机启动
systemctl enable nginx
```

### 安装php7.3.5 和 php7.2.18
#### 准备条件 安装php依赖组件（包含nginx依赖）
```
yum -y install  libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel ncurses ncurses-devel curl curl-devel krb5-devel libidn libidn-devel openldap openldap-devel nss_ldap jemalloc-devel cmake boost-devel bison automake libevent libevent-devel gd gd-devel libtool* libmcrypt libmcrypt-devel mcrypt mhash libxslt libxslt-devel readline readline-devel gmp gmp-devel libcurl libcurl-devel openjpeg-devel
```
#### 首先准备php7.3.5 和 php7.2.18 官方下载源码 并解压
```
pwd # /mash
wget https://www.php.net/distributions/php-7.3.5.tar.gz
tar -xvf php-7.3.5.tar.gz
```
#### 创建用户和组
```
groupadd www
useradd -g www www
```
#### 源码编译安装php7.3.5
```
./configure --prefix=/usr/local/php7.3.5 \
--with-config-file-path=/usr/local/php/etc \
--enable-fpm \
--with-fpm-user=www \
--with-fpm-group=www \
--enable-mysqlnd \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--enable-mysqlnd-compression-support

make && make install
```
#### 完整安装后配置php.ini文件 安装完成不会自动生成php.ini
```
pwd # /mash/php7.3.5
cp php.ini-development /usr/local/php7.3.5/etc/php.ini
cp /usr/local/php7.3.5/etc/php-fpm.conf.default /usr/local/php7.3.5/etc/php-fpm.conf
cp /usr/local/php7.3.5/etc/php-fpm.d/www.conf.default /usr/local/php7.3.5/etc/php-fpm.d/www.conf
```
#### 配置www.conf
```
listen = 127.0.0.1:9000
# 如果使用sock 请更改成 listen = /tmp/php7.3.5-sock
listen.owner = www
listen.group = www
listen.mode = 0660
```
#### 至此php7.3.5已经安装完毕 创建php-fpm.service服务(注意此处服务名称没带版本，下个php会更改)
```
vi /lib/systemd/system/php-fpm.service
# 写入以下内容
[Unit]
Description=php-fpm
After=network.target
[Service]
Type=forking
ExecStart=/usr/local/php7.3.5/sbin/php-fpm
PrivateTmp=true # 如果使用sock请将此处更改成false
[Install]
WantedBy=multi-user.target

#设置启动
systemctl daemon-reload
systemctl start | stop | restart | status php-fpm
systemctl enable php-fpm
```
### 设置nginx支持php
#### 设置nginx.conf文件
```
vi /usr/local/nginx/nginx.conf
##
user www www
location / {
    root   html;
    index  index.html  index.php index.htm;
}

location ~ \.php$ {
    root html;
    # fastcgi_pass   127.0.0.1:9000;
    fastcgi_pass unix:/tmp/php7.3.5-fpm.sock;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    include        fastcgi_params;
}
```
#### index.php文件输出phpinfo进行测试
```
mv /usr/local/nginx/html/index.html /usr/local/nginx/html/index.html.bak
vi /usr/local/nginx/html/index.php

```
访问ip，如果出现phpinfo,则证明nginx+单php设置成功

### 安装php7.2.18同上 设置www.conf listen 9001 or sock ，设置 nginx.conf listen







