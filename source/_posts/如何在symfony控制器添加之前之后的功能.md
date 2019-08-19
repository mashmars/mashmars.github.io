---
title: 如何在symfony控制器添加之前之后的功能
date: 2019-07-03 18:30:50
tags: 
 - symfony
 - event
categories: symfony
---
> 在web程序开发中，经常会遇到某个操作需要添加操作前或操作后的功能；
很多web框架中都有现成的preExceute()和postExceute(),但是symfony没有这样的定义。
但是可以利用自身组件event dispatcher 干扰request->response的过程。
yes just it

举例：比如某些控制器里的方法的访问需要定义个token，其他控制器不需要， 可进行如下操作.
## 在使用kernel.controller事件之前的处理 => preExceute()
### 定义允许的token参数
```
# src/config/services.yaml

parameters:
    tokens:
        client1: pass1
        client2: pass2
```
### 定义需要验证的controller,
> 为了能够区别需要验证和不验证的控制器， 可定义个接口，需要验证的控制器需要实现该接口

#### 定义接口
```
# src/Controller/TokenAuthenticatedController.php

namespace App\Controller;

interface TokenAuthenticatedController
{
    // none
}
```
#### 定义需要验证的控制器
```
# src/Controller/FooController.php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\ControllerAbstractController;
use Symfony\Component\Routing\Annotation\Route;
use App\Controller\TokenAuthenticatedController;

class FooController extends AbstractController implements TokenAuthenticatedController
{
    /**
     * @Route("/bar")
     */
    public function bar()
    {
        return $this->render('...twig');
    }
}
```
### 创建事件订阅者
```
# src/EventSubscriber/TokenSubscriber.php

namespace App\EventSubscriber;

use App\Controller\TokenAuthenticatedController;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\HttpKernel\Event\FilterControllerEvent;
use Symfony\Component\HttpKernel\Exception\AccessDeniedHttpException;
use Symfony\Component\HttpKernel\KernelEvents;

class TokenSubscriber implements EventSubscriberInterface
{
    private $tokens;

    public function __construct($tokens)
    {
        $this->tokens = $tokens;
    }

    public function onKernelController(FilterControllerEvent $event)
    {
        $controller = $event->getController();

        if(!is_array($controller)){
            return;
        }

        if($controller[0] instanceof TokenAuthenticatedController){
            $token = $event->getRequest()->query->get('token');
            if(!in_array($token, $this->tokens)){
                throw new AccessDeniedHttpException('this action needs a valid token');
            }
        }
    }

    public static function getSubscribedEvents()
    {
        return [
            KernelEvents::CONTROLLER => 'onKernelController',
        ];
    }

}
```
### 给TokenSubscriber传递参数tokens
```
# src/config/services.yaml

services:
    App\EventSubscriber\TokenSubscriber:
        arguments: ['%tokens%']
```
现在如果访问/bar or /bar?token=xxxx(!pass1|pass2)，会报错
但是访问 /bar?token=pass1 则正常

## 使用kernel.response处理事件之前 => postExceute()
### 在onKernelController方法里验证通过后增加
```
# src/Controller/FooContoller.php 

+ $event->getRequest()->attributes->set('auth_token', $token);
```
### 定义response处理方法
```
# src/Controller\FooController.php

use Symfony\Component\HttpKernel\Event\FilterResponseEvent;

public function onKernelResponse(FilterResponseEvent $event)
{
    if(!$token = $event->getRequest()->attributes->get('auth_token')){
        return;
    }

    $response = $event->getResponse();
    $hash = sha1($response->getContent() . $token);
    $resposne->headers->set('X-CONTENT-HASH', $hash);
}

public static function getSubscribedEvents()
{
    return [
        KernelEvent::CONTROLLER => 'onKernelController',
+       KernelEvents::RESPONSE => 'onKernelResponse',
    ];
}
```
### 这是效果
![](show.png)
