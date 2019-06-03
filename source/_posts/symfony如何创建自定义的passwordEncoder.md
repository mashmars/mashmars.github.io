---
title: symfony如何创建自定义的passwordEncoder
date: 2019-04-03 14:26:13
tags:
- symfony
categories:
- symfony
---

## symfony创建自定义passwordEncoder并使用
#### 创建passwordEncoder 必须实现PasswordEncoderInterface
```
namespace App\Security\Encoder;
use Symfony\Component\Security\Core\Encoder\PasswordEncoderInterface;

class Md5PasswordEncoder implements PasswordEncoderInterface
{
    /**
    * Encoder the raw password
    *
    * @param string $raw
    * @param string $salt
    * @return string the encoderd password
    */
    public function encodePassword($raw,$sale)
    {
        return md5($raw);
    }
    /**
    * check a raw password against an encoded password
    * @param string $encoderd 
    * @param $raw 
    * @param $salt
    * @return bool true or false
    */
    public function isPasswordValid($encoded, $raw, $salt)
    {
        return $encoded === md5($raw);
    }
}

```

#### 注册服务 
```
#services.yml
md5_encoder:
    class: 'App\Security\Encoder\Md5PasswordEncoder'
```

#### 配置实体使用md5 encoder
```
#security.yml
encoders:
    App\Entity\User: {id: md5_encoder }
```

#### 控制器使用加密or验证处理
##### 加密
```
use Symfony\Component\Security\Core\Encoder\UserPasswordEncoderInterface;
public function encodePassword(UserPasswordEncoderInterface $passwordEncoder)
{
    $passwordPlant = 'ssdfa1231';
    $password = $passwordEncoder->encodePassword($user , $passwordPlant);
    $user->setPassword($password);
}
```
##### 密码验证
```
public function  passwordValid(UserPasswordEncoderInterface $passwordEncoder)
{
    return $passwordEncoder->isPasswordValid($user,$passwordPlant);
}
```
