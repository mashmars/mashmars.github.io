---
title: symfony中dataFixtures及faker的使用
date: 2019-08-23 09:23:20
tags: 
    - symfony
categories: symfony
---

## 如何在symfony中使用dataFixture和faker进行测试数据填充

我们将通过文章和对应的评论进行举例说明；文章和评论的对应关系是一对多。
请先自行创建Article和Comment的Entity类

### 安装doctrine-fixtures-bundle
```
composer require --dev doctrine/doctrine-fixtures-bundle  or  composer req --dev orm-fixtures
```

### 生成fixture
```
php bin/console make:fixture
```
或者手动创建fixture类
```

namespace App\DataFixtures;

use Doctrine\Bundle\FixturesBundle\Fixture;
use Doctrine\Common\Persistence\ObjectManager;

class ArticleFixtures extends Fixture
{
    public function load(ObjectManager $manager)
    {
        // $product = new Product();
        // $manager->persist($product);

        $manager->flush();
    }
}

```

### 编写ArticleFixtures 并生成第一篇文章
```
public function load(ObjectManager $manager)
{
    // $product = new Product();
    // $manager->persist($product);

    $article = new Article();
    $article->setTitle('title');
    $article->setSlug('title-use');
    $article->setContent('content');
    $article->setPublishedAt(new \Datetime());
    $manager->persist($article);

    $manager->flush();
}
```
然后 运行 生成第一篇文章
```
php bin/console doctrine:fixtures:load
```
该命令会清空数据库 然后加载新数据 ，切记不能在线上数据库上使用 哈哈

### 接下来 让我们创建多篇文章

* 通过在load方法中使用 `for` 循环进行处理 
* 更棒的方式 通过创建一个BaseFixtures类 处理通用的多个entity保存

让我们创建BaseFixture类
```
<?php
namespace App\DataFixtures;

use Doctrine\Bundle\FixturesBundle\Fixture;
use Doctrine\Common\Persistence\ObjectManager;

abstract class BaseFixture extends Fixture 
{
    private $manager;

    public function load(ObjectManager $manager)
    {
        $this->manager = $manager;
        $this->loadData($manager);
    }

    /**
     * 加载数据
     */
    abstract protected function loadData(ObjectManager $manager);
}
```
### 修改ArticleFixtures 继承 BaseFixture
现在 在ArticleFixtures中不用处理load方法， 而是处理loadData方法
去除load,新增loadData

### 在BaseFixture中添加createManager方法
```
protected function createMany(string $className, int $count, callable $factory)
{
    for ($i = 0; $i < $count; $i++) {
        $entity = new $className();
        $factory($entity, $i);

        $this->manager->persist($entity);
        //store for usage later as App\Entity\ClassName_#COUNT#
        $this->addReference($className . '_' . $i, $entity); # 多个 fixtures类。当我们这样做时，我们需要能够从其他fixture类中引用在另外一个fixture类中创建的对象。通过调用addReference()，我们所有的对象都会自动存储，并准备使用其类名加索引号的键来获取。
    }
}
```
### 修改ArticleFixtures类中的loadData方法
```
public function loadData(ObjectManager $manager)
{
    $this->createMany(Article::class, 10, function(Article $article, $count) {
        $article->setTitle('title' . $count);
        $article->setSlug('slug' . $count);
        $article->setContent('content' . $count);
        if (mt_rand(1, 10) > 2) {
            $article->setPublishedAt(new \DateTime(sprintf('-%d days', rand(1, 100))));
        }
    });
    $manager->flush();
}
```
### 现在 重新加载数据
```
php bin/console doctrine:fixtures:load
```

yes it is success

## 使用Faker进行数据填充
### 安装faker
```
composer require fzaninotto/faker --dev
```
### BaseFixture中使用faker
```
+ use Faker\Factory;
# BaseFixture.php
+ protected $faker;

public function load(ObjectManager $manager)
{
    $this->manager = $manager;
    $this->faker = Factory::create();
    $this->loadData($manager);        
}
```
### 数据填充
打开ArticleFixtures。我们已经在publishedAt上有一点随机性了。但是，Faker甚至可以在这里提供帮助：
如果$this->faker->boolean()第一个参数是获得的机会，true则将其更改为。
让我们使用70：每篇文章发表的几率为70％。
```
private static $articleTitles = [
    'Why Asteroids Taste Like Bacon',
    'Life on Planet Mercury: Tan, Relaxing and Fabulous',
    'Light Speed Travel: Fountain of Youth or Fallacy',
];

private static $articleContents = [
    'Why Asteroids Taste Like Bacon',
    'Life on Planet Mercury: Tan, Relaxing and Fabulous',
    'Light Speed Travel: Fountain of Youth or Fallacy',
];


public function loadData(ObjectManager $manager)
{
    $this->createMany(Article::class, 10, function(Article $article, $count) {
        $article->setTitle($this->faker->randomElement(self::$articleTitles));
        $article->setSlug($this->faker->slug);
        $article->setContent($this->faker->randomElement(self::$articleContents));
        if ($this->faker->boolean(70)) {
            $article->setPublishedAt($this->faker->dateTimeBetween('-100 days', '-1 days'));
        }
        # $article->setHeartCount($this->faker->numberBetween(5, 100))
    });
    $manager->flush();
}
```
### 运行
```
php bin/console doctrine:fixtures:load
```

yes it is success

## 处理关联
### 创建CommentFixtures
```
php bin/console make:fixtures

<?php

namespace App\DataFixtures;

use App\DataFixtures\BaseFixture;
use Doctrine\Common\Persistence\ObjectManager;
use App\Entity\Comment;
use App\Entity\Article;

class CommentFixtures extends BaseFixture
{
    protected function loadData(ObjectManager $manager)
    {
        $this->createMany(Comment::class, 100, function(Comment $comment, $count) {
            $comment->setContent(
                $this->faker->boolean ? $this->faker->paragraph : $this->faker->sentences(2, true)
            );
            $comment->setArticle(
                $this->getReference(Article::class.'_'.$this->faker->numberBetween(0, 9))
            );
        });
        $manager->flush();
    }
}

```

### 使用通用的随机关联
```
abstract class BaseFixture extends Fixture
{
    private $referencesIndex = [];

    protected function getRandomReference(string $className) {
        if (!isset($this->referencesIndex[$className])) {
            $this->referencesIndex[$className] = [];
            foreach ($this->referenceRepository->getReferences() as $key => $ref) {
                if (strpos($key, $className.'_') === 0) {
                    $this->referencesIndex[$className][] = $key;
                }
            }
        }
        if (empty($this->referencesIndex[$className])) {
            throw new \Exception(sprintf('Cannot find any references for class "%s"', $className));
        }
        $randomReferenceKey = $this->faker->randomElement($this->referencesIndex[$className]);
        return $this->getReference($randomReferenceKey);
    }    
}
```
### CommentFixtures中的修改
```
#$comment->setArticle($this->getReference(Article::class.'_'.$this->faker->numberBetween(0, 9)));
$comment->setArticle($this->getRandomReference(Article::class));
```

yes it is success ~ ~ ~ 

## 最后 fixtures类的执行顺序问题
默认情况下，它按字母顺序加载它们。所以 如果有关联模型的 一定 要定义依赖性来调整执行顺序
### CommentFixtures的执行 需要先有ArticleFixtures
```
# CommentFixtures.php
use Doctrine\Common\DataFixtures\DependentFixtureInterface;
use App\DataFixtures\ArticleFixtures;

class CommentFixture extends BaseFixture implements DependentFixtureInterface
{
    ... lines 
    public function getDependencies()
    {
        return [ArticleFixtures::class];
    }
}
```
that's all ~~~~







