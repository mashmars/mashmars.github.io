---
title: hexo+github博客搭建实战首篇
date: 2019-04-01 10:26:23
tags: 
- hexo
categories:
- hexo 
top: true
---

# hexo搭建简单步骤

具体详情请参考[hexo文档](https://hexo.io/zh-cn/)

安装 node.js
安装 git
### 安装 
hexo npm install -g hexo-cli
### 初始化 hexo 文件夹 
```
hexo init <folder> 
cd <folder>
nmp install
```
### 下载 next 主题 配置next
### 开启 分类和标签 页 
```
hexo new page categories
hexo new page tags

```
### 创建新文章
```
hexo new <layout> title
```
### 开启网站
```
hexo server -p 4000 [...]
```
### 生成静态页
```
hexo clean
hexo generate or hexo g
```
## 发布到github并设置自定义域名
#### 购买域名 设置 A记录（192.30.252.153 or 192.30.252.154） 配置CNAME指向该A记录
#### 创建github仓库 xxxx.github.io
#### 安装 git-deploy
```
npm install hexo-deployer-git --save
```
#### hexo _config.yml 配置 deploy
```
deploy:
    type: git
    repo: https://github.com/xxx/xxx.github.io
    branch: master
```
#### 创建自定义域名 `在source文件夹下创建All type文件 CNAME 填写自定义的域名`
### 一键部署到github并关联自定义域名
```
hexo deploy or hexo d
```
### 访问 xxx.github.io or 自定义的域名可以看到自己的blog了 < %_% >




