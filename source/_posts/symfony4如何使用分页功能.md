---
title: symfony4如何使用分页功能
date: 2019-04-01 13:36:32
tags: symfony4 
categories: symfony4
---

## 描述
{% blockquote %}
为了记录symfony4的学习过程，首先准备了后台的文章管理系统，通过一系列的准备工作，比如选择编辑器等；另外就是先解决文章的分页功能，在这儿通过使用knp分页组件完成
{% endblockquote %}
#### 安装knp-paginator-bundle
```
composer require 'knplabs/knp-paginator-bundle'
```
#### Controller里使用分页功能
```
$qb = $this->getDoctrine()->getManager()->getRepository(Article::class)->createQueryBuilder('u');
$paginator = this->get('knp_paginator');
$pagination = $paginator->paginate($qb,$page,$limit);
return $this->render('',['pagination'=>$pagination]);
```
#### Twig里使用
```
{% for data in pagination %}
{{ data.title }}
{% endfor %}
{% knp_pagination_render(pagination) %}
```
