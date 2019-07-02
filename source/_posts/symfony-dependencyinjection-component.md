---
title: symfony-dependencyinjection-component
date: 2019-07-02 10:36:20
tags: 
 - symfony
 - di
categories: symfony
---
> 使用symfony di
### 创建项目 引入dependencyInjection component
```
mkdir test
composer require symfony/dependency-injeciton
cat index.php
```
### 创建文件夹 src,放自己的类并加命名空间App\Mash
```
mkdir src
vi composer.json 
+ "autoload" : {
    "psr-4" : {
        "App\\Mash\\" : "src"
    }
}

composer dump
```
### 创建App\Mash\Mailer类
```
# src/Mailer.php
<?php
namespace App\Mash;

class Mailer
{
    private $transport;

    public function __construct($transport)
    {
        $this->transport = $transport;
    }

    public function getTransport()
    {
        return $this->transport;
    }
}
```
### 创建App\Mash\NewsletterManager
```
# src/NewsletterManager.php
<?php
namespace App\Mash;
use App\Mash\Mailer;

class NewsletterManager
{
    private $mailer;

    public function __construct(Mailer $mailer)
    {
        $this->mailer = $mailer;
    }

    public function getTransport()
    {
        return $this->mailer->getTransport();
    }
}
```
### 编辑index.php 
#### 使用方法一 no use DI
```
<?php
require_once __DIR__ . '/vendor/autoload.php';

use App\Mash\Mailer;
use App\Mash\NewsletterManager;

$mailer = new Mailer(465);
$newsletterManager = new NewsletterManager($mailer);
echo $newsletterManager->getTransport(); // 465
```
#### 使用php 设置服务 use DI
```
require_once __DIR__ . '/vendor/autoload.php';

use App\Mash\Mailer;
use App\Mash\NewsletterManager;
use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\DependencyInjection\Reference;

/* $containerBuilder = new ContainerBuilder();
$containerBuilder->setParameter('mailer.transport', 22);
$containerBuilder
    ->register('mailer', 'App\Mash\Mailer')
    //->addArgument('465') or 
    ->addArgument('%mailer.transport%')
    ; */

/* 
$mailer = $containerBuilder->get('mailer');
echo $mailer->getTransport(); */

// method 1
/* $containerBuilder
    ->register('newsletter_manager', 'App\Mash\NewsletterManager')
    ->addArgument($containerBuilder->get('mailer'))
    ;

$newsletterManager = $containerBuilder->get('newsletter_manager');
echo $newsletterManager->getTransport(); */

// method2
/* $containerBuilder
    ->register('newsletter_manager', NewsletterManager::class)
    ->addArgument(new Reference('mailer'))
    ;
$newsletterManager = $containerBuilder->get('newsletter_manager');
echo $newsletterManager->getTransport(); */
```
#### 使用配置文件 配置服务
> 除了如上所述使用PHP设置服务之外，您还可以使用配置文件。
* 这允许您使用XML或YAML编写服务的定义，而不是使用PHP来定义服务，
* 如上例所示。除了最小的应用程序之外，
* 通过将服务定义移动到一个或多个配置文件中来组织服务定义是有意义的。
* 为此，您还需要安装 Config组件。composer require symfony/config
* Maybe require Yaml Component composer require symfony/yaml

##### 在src下创建config文件加 及 services.yaml
```
# src/config/services.yaml
parameters:
  mailer.transport: 465

services:
  mailer:
    class: 'App\Mash\Mailer'
    arguments: ['%mailer.transport%']

  newsletter_manager:
    class: 'App\Mash\NewsletterManager'
    arguments: ['@mailer'] # '@mailer' error setArguments() must be of the type array, object given, 
    #setMethod
    #calls:
      #- [setMailer, ['@mailer']]
```
##### index.php 使用方法
```
# index.php
<?php
require_once __DIR__ . '/vendor/autoload.php';

use App\Mash\Mailer;
use App\Mash\NewsletterManager;
use Symfony\Component\DependencyInjection\ContainerBuilder;
//use Symfony\Component\DependencyInjection\Reference;
use Symfony\Component\Config\FileLocator;
use Symfony\Component\DependencyInjection\Loader;

$containerBuilder = new ContainerBuilder();
$loader = new Loader\YamlFileLoader($containerBuilder, new FileLocator(__DIR__ . '/src/config'));
$loader->load('services.yaml');

$newsletterManager = $containerBuilder->get('newsletter_manager');
echo $newsletterManager->getTransport(); # 465
```
