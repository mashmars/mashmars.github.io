---
title: 单独使用symfony的eventDispatcher的应用
date: 2019-07-04 11:18:28
tags: 
 - symfony
 - event
categories: symfony
---
> 本篇文章介绍如何单独使用symfony的event-dispatcher组件
思路是 创建个单独的事件处理类 以及 该事件的订阅者类 通过调度者触发
## 使用订阅者 subscriber
### 配置文件系统结构
```
composer require symfony/event-dispatcher
mkdir src
vi composer.json
+ "autoload": {
    "psr-4":{
        "App\\Mash\\" : "src"
    }
}

cat index.php
```
### index.php
```
<?php 
require_once __DIR__ . '/vendor/autoload.php';
# 第一步 创建自定义event
use App\Mash\Event\CustomerEvent;
# 第二步 创建订阅者
use App\Mash\EventSubscriber\CustomerSubscriber;
# 第三步 创建调度者进行调度
use Symfony\Component\EventDispatcher\EventDispatcher;

$event = new CustomerEvent();
$subscriber = new CustomerSubscriber();
$dispatcher = new EventDispatcher();

$dispatcher->addSubscriber($subscriber);

$dispatcher->dispatch($event, CustomerEvent::NAME);

echo 'index';
```
### CustomerEvent.php
```
<?php
namespace App\Mash\Event;


class CustomerEvent
{
    public const NAME = 'customer.event';

    public function getCustomerEventAction()
    {
        echo '<pre>';
        var_dump(__FILE__ );
    }
}
```
### CustomerSubscriber.php
```
<?php
namespace App\Mash\EventSubscriber;

use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use App\Mash\Event\CustomerEvent;

class CustomerSubscriber implements EventSubscriberInterface
{
    public static function getSubscribedEvents()
    {
        return [
            CustomerEvent::NAME => 'onCustomerSubscriberAction',
        ];
    }

    public function onCustomerSubscriberAction(CustomerEvent $event)
    {
        //调用CustomerEvent 处理
        $event->getCustomerEventAction();
        var_dump(__FILE__);
    }

}
```
### 访问index.php
![](show.png)

## 使用监听者 listener
### 创建 CustomerListener.php
```
# src/EventListener/CustomerListener.php

<?php
namespace App\Mash\EventListener;

use App\Mash\Event\CustomerEvent;

class CustomerListener
{
    public function getCustomerListenerAction(CustomerEvent $event)
    {
        $event->getCustomerEventAction();
        var_dump(__FILE__);
    }
}
```
### index.php 使用
```
# index.php

# or use listener
use App\Mash\EventListener\CustomerListener;

$event = new CustomerEvent();
$subscriber = new CustomerSubscriber();
$dispatcher = new EventDispatcher();

$dispatcher->addSubscriber($subscriber);


+ $listener = new CustomerListener();
+ $dispatcher->addListener(CustomerEvent::NAME, [$listener, 'getCustomerListenerAction']);

$dispatcher->dispatch($event, CustomerEvent::NAME); # 同时触发subscriber、 listener

echo 'index';
```
### 访问index.php
![](show1.png)


具体信息请参考官方文档 [symfony document](https://symfony.com/doc/current/components/event_dispatcher.html)
