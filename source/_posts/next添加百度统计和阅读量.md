---
title: next添加百度统计和阅读量
date: 2019-04-03 13:17:10
tags:
    - hexo
    - next
categories: hexo
---
# 添加百度统计和文章阅读量
### 开启hexo图片功能
#### 使用hexo3.0以上的资源引用功能
```
站点配置 _config.yml开启 post_asset_folder: true
hexo new title 的同时会生成一个同名文件夹 ，把相关图片放在这个目录下
{% asset_img slug [title] %}  引用图片
```

#### 使用插件方式
```
npm install hexo-asset-image --save
[图片上传失败...(image-43fc5f-1510018038370)]
```

#### markdown 语法
```
![](/path/img)
```

### 打开百度统计 注册登录  添加应用并找到代码 复制到 主题配置文件_config.yml baidu_analytics: [id]
### 文章阅读量 
#### 打开leancloud 注册登录 并创建应用 找到appkey等信息 如图
{% asset_img appkey.png %}
#### 复制app_id,app_key到站点配置文件 leancloud_visitors下的配置项
#### 创建存储classes 
{% asset_img classes.png %}
#### 配置安全域名
{% asset_img yuming.png %}
#### 打开页面查看效果
{% asset_img result.png %}

# 附件 添加评论功能 
### 使用valine 
#### 为什么使用valine? 
{% blockquote %}
因为和leadcloud共用appid appkey
{% endblockquote %}
#### 开启valine
{% blockquote %}
找到主题配置文件 valine 开启 enable: true 设置appid appkey 即可
{% endblockquote %}
