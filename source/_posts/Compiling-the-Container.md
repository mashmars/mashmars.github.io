---
title: Compiling-the-Container
date: 2019-07-02 11:33:01
tags: 
 - symfony
 - di
categories: symfony
---
> 服务容器因为多种原因需要被编译。比如解析参数和去掉不用的服务
它通过运行 `$container->compile()`进行编译，如果不编译
比如无法解析参数 在config下创建config.yaml增加该一些参数配置
如果不编辑会提示 `There is no extension able to load the configuration for "mash"` 

### 创建参数配置文件
```
# src/config/config.yaml
mash:
  foo: fooValue
  bar: barValue
```
### 创建扩展
```
# src/MashExtension.php
<?php
namespace App\Mash;

use Symfony\Component\Config\FileLocator;
use Symfony\Component\DependencyInjection\Loader;
use Symfony\Component\DependencyInjection\Extension\ExtensionInterface;
use Symfony\Component\DependencyInjection\ContainerBuilder;

class MashExtension implements  ExtensionInterface
{
    public function load(array $configs, ContainerBuilder $container)
    {
        $loader = new Loader\YamlFileLoader($container, new FileLocator(__DIR__ . '/config'));
        $loader->load('services.yaml');
        
        /**
         * 此时 已经从编译容器处获得了外部配置参数
         * 上面加载了服务 可以处理服务参数了
         */
        var_dump($configs);exit;
        /**
         * array [ 0 => [
             'foo' => 'fooValue', 'bar' => 'barValue'
         ] ]
         */
    }

    public function getAlias()
    {
        return 'mash';
    }

    /**
     * Returns the namespace to be used for this extension (XML namespace).
     *
     * @return string The XML namespace
     */
    public function getNamespace()
    {}

    /**
     * Returns the base path for the XSD files.
     *
     * @return string The XSD base path
     */
    public function getXsdValidationBasePath()
    {}
}
```
### index.php 只有在编译容器时才处理其中的值
```
<?php
require_once __DIR__ . '/vendor/autoload.php';

use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\Config\FileLocator;
use Symfony\Component\DependencyInjection\Loader;
use App\Mash\MashExtension;

$containerBuilder = new ContainerBuilder();
$containerBuilder->registerExtension(new MashExtension);


$loader = new Loader\YamlFileLoader($containerBuilder, new FileLocator(__DIR__ . '/src/config'));
$loader->load('config.yaml');

$containerBuilder->compile(); # 若无 error There is no extension able to load the configuration for "mash" 
```
### 虽然您可以手动管理合并不同的文件，但使用Config组件合并和验证配置值要好得多。使用配置处理，您可以通过以下方式访问配置值：
#### 创建configuration类
```
# src/Configuration.php
<?php
namespace App\Mash;

use Symfony\Component\Config\Definition\Builder\TreeBuilder;
use Symfony\Component\Config\Definition\ConfigurationInterface;
use Symfony\Component\Config\Definition\Builder\ArrayNodeDefinition;

class Configuration implements ConfigurationInterface
{
    public function getConfigTreeBuilder()
    {
        $treeBuilder = $treeBuilder = new TreeBuilder('config'); // load config.yaml

        $treeBuilder->getRootNode()
            ->children()
                ->scalarNode('foo')->end()
                ->scalarNode('bar')->end()
            ->end();
        return $treeBuilder;

        // or 加载mash.yaml
        $treeBuilder = $treeBuilder = new TreeBuilder('mash');

        $treeBuilder->getRootNode()
            ->children()
                ->scalarNode('foo')->end()
                ->scalarNode('bar')->end()
            ->end();
        return $treeBuilder;
    }
}
```
#### 配置config.yaml 
```
# src/config/config.yaml
mash:
  foo: fooValue1
  bar: barValue2
#### or 

```
#### MashExtension.php内容变化
```
# src/MashExtension.php
use App\Mash\Configuration;

public function load(array $configs, ContainerBuilder $container)
{
    $loader = new Loader\YamlFileLoader($container, new FileLocator(__DIR__ . '/config'));
    $loader->load('services.yaml');
    
    /**
        * 此时 已经从编译容器处获得了外部配置参数
        * 上面加载了服务 可以处理服务参数了
        */
    //var_dump($configs);exit;

    /**
        *  array 0=> [ 'foo' => 'fooValue', 'bar' => 'barValue']
        * $foo = $configs[0]['foo']; //fooValue $bar = $configs[0]['bar']; //barValue
        * 虽然您可以手动管理合并不同的文件，
        * 但使用Config组件合并和验证配置值要好得多。
        * 使用配置处理，您可以通过以下方式访问配置值：  
        * use Symfony\Component\Config\Definition\Processor;      
        */

    $configuration = new Configuration();
    $processor = new Processor();
    $config = $processor->processConfiguration($configuration, $configs);

    $foo = $config['foo']; //fooValue
    $bar = $config['bar']; //barValue

    $container->setParameter('mailer.foo', $foo);
    var_dump($config);exit;

    if ($config['advanced']) {
        $loader->load('advanced.yaml'); # 加载其他服务定义文件
    }
}
```
### 在实现以下load() 方法之前，Extension可以在调用方法 之前添加任何Bundle的配置PrependExtensionInterface：
```
use Symfony\Component\DependencyInjection\Extension\PrependExtensionInterface;
// ...

class AcmeDemoExtension implements ExtensionInterface, PrependExtensionInterface
{
    // ...

    public function prepend(ContainerBuilder $container)
    {
        // ...

        $container->prependExtensionConfig($name, $config);
        /**
         * $name  => service id
         * $config => service id的相关配置         
         */
        // ...
    }
}
```