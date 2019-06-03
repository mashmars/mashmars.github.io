---
title: hexo上传静态文件及源码
date: 2019-05-31 18:36:18
tags:
    - hexo
    - next
categories: hexo
---
#### 创建github分支 master 和 source, 设置master为默认分支
#### 配置hexo下的_config.yml 发布到master
```
deploy:
  type: git
  repo: https://github.com/mashmars/mashmars.github.io
  branch: master
```
#### clone源码分支source仓库到本地
```
git clone -b source https://github.com/yourname/yourname.github.io.git
```
#### 上传源码
```
git add .
git commit
git push origin source
```
#### 发布静态文件
```
hexo clean
hexo generate -d 
```
yes
