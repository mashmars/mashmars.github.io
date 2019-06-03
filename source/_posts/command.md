---
title: hexo 命令
date: 2019-04-10 09:27:44
tags:
- hexo
categories:
- hexo
---
### install
```
hexo init [folder] #初始化目录
cd folder
npm install
```
新建一个网站。如果没有设置` folder `,hexo默认在当前文件夹建立网站
### new
```
$ hexo new [layout] <title>
```
新建一篇文章。如果没有设置` layout `的话，默认使用_config.yml中的` default_layout `参数代替。如果标题有空格，请用引号括起来。
### generate
```
$ hexo generate|g [-d|--deploy #文件生成后立即部署网站] [-w|--watch #监视文件变动]  
```
生成静态文件
### publish
```
$ hexo publish [layout] <filename>
```
发布草稿
### server
```
$ hexo server [-p|--port 4001 #重设端口] [-s|--static #只使用静态文件] [-l|--log #启动日志记录使用覆盖记录格式]
```
### deploy
```
$ hexo deploy|d [-g,--generate #部署之前预先生成静态文件]
```
部署网站
### render
```
$ hexo render <file1> [file2] ... [-o|--output #设置输出路径]
```
渲染文件
### migrate
```
$ hexo migrate <type>
```
从其他博客系统迁移内容
### clean
```
$ hexo clean
```
清除缓存文件(` db.json `)和已生成的静态文件(` public `)
在某些情况下比如更换主题后，如果发现站点的更改没有生效，可以运行该命令
### list
```
$ hexo list <type>
```
列出网站资料
### version
```
$ hexo version
```
显示hexo版本
