---
title: next主题添加分享功能
date: 2019-04-03 16:26:58
tags:
- hexo
- next
categories:
- hexo
---

## hexo next主题添加分享功能
### 使用 [sharesdk](http://sharesdk.mob.com) 实现分享
#### 注册登录 创建应用 复制appkey
#### 添加分享模板
```
\themes\next\layout\_partials\share
新建一个文件名为 sharesdk.swig ，并输入以下代码

<!--MOB SHARE BEGIN-->
<div class="-mob-share-ui-button -mob-share-open">分享</div>
<div class="-mob-share-ui" style="display: none">
    <ul class="-mob-share-list">
        <li class="-mob-share-weibo"><p>新浪微博</p></li>
        <li class="-mob-share-tencentweibo"><p>腾讯微博</p></li>
        <li class="-mob-share-qzone"><p>QQ空间</p></li>
        <li class="-mob-share-qq"><p>QQ好友</p></li>
        <li class="-mob-share-weixin"><p>微信</p></li>
        <li class="-mob-share-douban"><p>豆瓣</p></li>
        <li class="-mob-share-renren"><p>人人网</p></li>
        <li class="-mob-share-kaixin"><p>开心网</p></li>
        <li class="-mob-share-facebook"><p>Facebook</p></li>
        <li class="-mob-share-twitter"><p>Twitter</p></li>
        <li class="-mob-share-pocket"><p>Pocket</p></li>
        <li class="-mob-share-google"><p>Google+</p></li>
        <li class="-mob-share-youdao"><p>有道云笔记</p></li>
        <li class="-mob-share-mingdao"><p>明道</p></li>
        <li class="-mob-share-pengyou"><p>朋友网</p></li>
        <li class="-mob-share-tumblr"><p>Tumblr</p></li>
        <li class="-mob-share-instapaper"><p>Instapaper</p></li>
        <li class="-mob-share-linkedin"><p>LinkedIn</p></li>
    </ul>
    <div class="-mob-share-close">取消</div>
</div>
<div class="-mob-share-ui-bg"></div>
<script id="-mob-share" src="http://f1.webshare.mob.com/code/mob-share.js?appkey={{ theme.shareSDKappkey }}"></script>
<!--MOB SHARE END-->

```

#### 打开模板文件 `\themes\next\layout\post.swig` 在 block content里添加以下代码
```
<div class="post-spread">
    {% if theme.jiathis %}
    {% include '_partials/share/jiathis.swig' %}
    {% elseif theme.baidushare %}
    {% include '_partials/share/baidushare.swig' %}
    {% elseif theme.add_this_id %}
    {% include '_partials/share/add-this.swig' %}
    {% elseif theme.duoshuo_shortname and theme.duoshuo_share %}
    {% include '_partials/share/duoshuo_share.swig' %}
    {% elseif theme.sharesdk %}
    {% include '_partials/share/sharesdk.swig' %}
    {% endif %}
</div>
```

#### 添加主题配置文件添加以下内容
```
sharesdk: true
shareSDKappkey: 你的appkey
```

