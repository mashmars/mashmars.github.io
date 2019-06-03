---
title: symfony4上传文件
date: 2019-04-03 10:20:06
tags: symfony
categories: symfony
---

可以考虑使用`VichUploaderBunde`组件实现
手动处理分两种，一个是控制器直接处理 一个是注册服务，控制器引用
# 控制器处理
### Entity引入 `user Symfony\Compoent\Validator\Constraints as Assert ` 
### 定义字段 `@Assert\File(mimeTypes={"image/png","image/jpeg"})`
### 表单类Form 引入字段 ` ->add('file',FileType::class)`, 编辑的时候可以加上参数`['data_class'=>null,]`,否则会报错
### 新增提交后处理
```
$file = $form->get('thumb')->getData();
if($file !== null){
    $fileName = $this->generateUniqueFileName() . '.' . $file->guessExtension();
    $file->move($this->getParameter('upload_path'),$fileName);
    $article->setThumb($fileName);
    $article = $form->getData();
}
```
### 设置上传路径参数 `App\config\service.yaml`
```
parameters:
    upload_path: '%kernel.project_dir%/public/uploads'
```
# 服务
### 定义服务
```
namespace App\Service;
use Symofny\Component\HttpFoundation\File\UploadFile;
class FileUploader
{
    private $targetDirectory;
    public function __construct($targetDirectory)
    {
        $this->targetDirectory = $targetDirectory;
    }
    public function upload(UploadFile $file)
    {
        $fileName = md5(uniqid()) . '.' . $file->guessExtension();
        $file->move($this->getTargetDirectory(),$fileName);
        return $fileName;
    }
    public function getTargetDirectory()
    {
        return $this->targetDirectory;
    }
}
```

### 注册服务
```
#config/services.yaml
services:
    App\Service\FileUploader:
        arguments:
            $targetDirectory: '%upload_path%'
```

### 控制器使用
```
use App\Service\FileUploader
public function add(Request $request,FileUploader $fileUploader)
{
    $file = $this->get('thumb')->getData();
    if($file !== null){
        $fileName = $fileUploader->upload($file);
        $article->setThumb($fileName);
    }
    $article = $form->getData();
}
```

