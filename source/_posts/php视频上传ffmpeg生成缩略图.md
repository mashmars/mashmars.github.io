---
title: php视频上传ffmpeg生成缩略图
date: 2019-04-03 14:20:40
tags: 
- php
- ffmpeg
categories: php
---
## 如何上传视频并生成缩略图
#### composer依赖库
```
composer require php-ffmpeg/php-ffmpeg
```
#### 代码使用
```
use FFMpeg\Coordinate\TimeCode;
use FFMpeg\FFMpeg;
+use FFMpeg\Coordinate\Dimension;
    $ffmpeg = FFMpeg::create(array(
                //windows下
                'ffmpeg.binaries' => 'D:\ffmpeg\bin\ffmpeg.exe',//插件下载地址
                'ffprobe.binaries' => 'D:\ffmpeg\bin\ffprobe.exe',
                //linux 下
                //'ffmpeg.binaries'  => '/usr/local/bin/ffmpeg',
                //'ffprobe.binaries' => '/usr/local/bin/ffprobe',
    
                'timeout' => 0,
                'ffmpeg.threads' => 12
            ));
     $video = $ffmpeg->open("upload/".$_FILES["file"]["name"]);
     $video->frame(TimeCode::fromSeconds(20))->save('frame.jpg'); //生成frame.jpg的文件
     $video->gif(TimeCode::fromSeconds(20),new Dimension(300,400),100)->save('upload/frame1.gif');
```
#### 准备条件
windows or linux下下载 https://ffmpeg.org/download.html#build-linux
```
wget https://ffmpeg.org/releases/ffmpeg-4.1.tar.bz2
tar -xjvf ffmpeg-4.1.tar.bz2
cd ffmpeg-4.1
./configure --enable-shared --prefix=/ffmpeg
如果现在直接执行configure配置的话，可能会报如错误
yasm/nasm 包不存在或者很旧，可以使用--disable-yasm禁用这个选项编译，yasm是一款汇编器，并且是完全重写了nasm的汇编环境，接收nasm和gas语法，支持x86和amd64指令集，所以这里安装一下yasm即可，下载地址是：http://yasm.tortall.net/Download.html 进入后下载1.3.0的源码包，执行下面命令安装
wget http://www.tortall.net/projects/yasm/releases/yasm-1.3.0.tar.gz
tar -xvzf yasm-1.3.0.tar.gz
cd yasm-1.3.0/
./configure
make
make install
```
#### yasm安装结束后 重新安装ffmpeg
```
./configure --enable-shared
make
make install
```
#### symfony中使用代码
```
$uploadedFile = new UploadedFile($video,'');
$newFileName =  md5(uniqid()) ;
$newFile = $newFileName . '.' . strtolower($uploadedFile->guessExtension());
$ext = strtolower($uploadedFile->guessExtension());
$size = $uploadedFile->getSize();
if(!in_array($ext,['mp4','mov'])) return $this->json(['code'=>1,'msg'=>'上传文件后缀不允许']);
if($size > 100000000) return $this->json(['code'=>1,'msg'=>'上传文件不能超过100M']);

$uploadedFile->move('./uploads/task/video/kid/',$newFile);             
$from = './uploads/task/video/kid/'.$newFile;
$to = 'task/'.$courseHourPlan->getGrade()->getSign().'/video/' . $userid . '/' .$newFile;
$return = $this->aliyunoss($to,$from);
if($return){
    return $this->json(['code'=>1,'msg'=>$return]);
}else{
    //
    $url =  $to ;
}
$taskVideo = new ApiYdTaskVideo();
$taskVideo->setTask($task);
$taskVideo->setUser($this->getUser()->getUser());
$taskVideo->setUrl($url);
$em->persist($taskVideo);
//生成缩略图
$ffmpeg = FFMpeg::create(array(
    //'ffmpeg.binaries' => 'E:\ffmpeg\bin\ffmpeg.exe',//插件下载地址
    //'ffprobe.binaries' => 'E:\ffmpeg\bin\ffprobe.exe',
    //linux 下
    'ffmpeg.binaries'  => '/usr/local/bin/ffmpeg',
    'ffprobe.binaries' => '/usr/local/bin/ffprobe',

    'timeout' => 0,
    'ffmpeg.threads' => 12
));
$video = $ffmpeg->open($from);
$newFileImg = $newFileName .'.jpg';
$img = './uploads/task/video/kid/'.$newFileImg;
$video->frame(TimeCode::fromSeconds(5))->save($img);  
$to = 'task/'.$courseHourPlan->getGrade()->getSign().'/video/' . $userid . '/' .$newFileImg;
$this->aliyunoss($to,$img);

//删除本地视频
unlink($from);
unlink($img);
```

