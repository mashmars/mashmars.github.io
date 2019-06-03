---
title: hexo 标签插件
date: 2019-04-10 10:10:34
tags:
- hexo
categories:
- hexo 
---
标签插件和Front-matter中的标签不同，它们用于在文章中快速插入特定内容的插件
### 引用块
在文章中插入引言，可包含作者、来源和标题。
#### 别号: quote
```
{% blockquote [author[, source]] [link] [source_link_title] %}
content
{% endblockquote %}
```
#### 样例
```
{% blockquote %}
没有提供参数，则只输出普通的blockquote
{% endblockquote %}
```
{% blockquote %}
没有提供参数，则只输出普通的blockquote
{% endblockquote %}
```
{% blockquote David, Wide Awake %}
引用书上的句子
{% endblockquote %}
```
{% blockquote David, Wide Awake %}
引用书上的句子
{% endblockquote %}
```
{% blockquote @DevDocs http://mashuai.imkxa.com %}
我的博客 http://mashuai.imkxa.com
{% endblockquote %}
```
{% blockquote @DevDocs http://mashuai.imkxa.com %}
我的博客 http://mashuai.imkxa.com
{% endblockquote %}
### 代码块
#### 普通的代码块
```
{% codeblock %}
alert('hello world');
{% endcodeblock %}
```
{% codeblock %}
alert('hello world');
{% endcodeblock %}
#### 附加说明
```
{% codeblock Array.map %}
array.map
{% endcodeblock %}
```
{% codeblock Array.map %}
array.map
{% endcodeblock %}
#### 附件说明和网址
```
{% codeblock Array.map http://www.imkxa.com 我的站点 %}
我的站点
{% endcodeblock %}
```
{% codeblock Array.map http://www.imkxa.com 我的站点 %}
我的站点
{% endcodeblock %}
### 反引号代码块
另一种形式的代码块，不同的是他使用三个反引号来包括
### Image
在文章中插入指定大小的图片
```
{% img [class names] /path/to/image [width] [height] [title text [alt text]] %}
```
### Link
在文章中插入链接，并自动给外部链接添加` target="_blank" ` 属性。
```
{% link text url [external] [title] %}
```
### 引用文章
引用其他文章的链接
```
{% post_path slug %}
{% post_link slug [title] %}
```
### 引用资源
引用文章的资源。
```
{% asset_path slug %}
{% asset_img slug [title] %}
{% asset_link slug [title] %}
```
### Raw 
如果想在文章插入Swig标签，可以使用Raw标签，以免发现解析
```
{% raw %}
content
{% endraw %}
```
{% raw %}
``` 这是三个反引号代码块 ```
{% endraw %}

