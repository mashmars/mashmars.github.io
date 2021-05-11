---
title: grpc的基本使用
date: 2021-05-11 09:00:06
tags: 
    - grpc
categories: grpc
---
### grpc安装 见文档
### 使用go初始化项目
``` go mod init grpcc ```
### 创建proto文件 ProductInfo.proto
```
syntax = "proto3";

package ecommerce;

option go_package = "../gopd";

service ProductInfo {
    rpc addProduct(Product) returns (ProductID);
    rpc getProduct(ProductID) returns (Product);
}

message Product {
    string id = 1;
    string name = 2;
    string description = 3;
}

message ProductID {
    string value = 1;
}
```

### 生成go版本的pb文件
```
protoc -I . --go_out=plugins=grpc:. ./ProductInfo.proto
```
### 创建go server端 server.go
```
package main

import (
	"context"
	"net"
	"log"
	"errors"
	pb "grpcc/gopd"
	"google.golang.org/grpc"
	"github.com/gofrs/uuid"
)


const (
	Address = "127.0.0.1:50052"
)

//productInfo  自定义保存信息
type ProductInfo struct{
	productMap map[string]*pb.Product
}
func (pinfo *ProductInfo) AddProduct(ctx context.Context, in *pb.Product) (*pb.ProductID, error) {
	out, err := uuid.NewV4()
	if err != nil {
		return nil,err
	 }
	 in.Id = out.String()
	 if pinfo.productMap == nil {
		pinfo.productMap = make(map[string]*pb.Product)
	 }
	 pinfo.productMap[in.Id] = in
	 log.Println(pinfo.productMap)
	 return &pb.ProductID{Value: in.Id}, nil
}

func (pinfo *ProductInfo) GetProduct(ctx context.Context, in *pb.ProductID) (*pb.Product, error) {
	value, exists := pinfo.productMap[in.Value]
	if exists {
		return value, nil
	}
	return nil, errors.New("product does not exist")
}

func main() {
	listen, err := net.Listen("tcp", Address)
	if err != nil {
		log.Fatalf("listen error: %v", err)
	}

	s := grpc.NewServer()
	pb.RegisterProductInfoServer(s, &ProductInfo{})

	if err := s.Serve(listen); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}
```
### 创建go client端  client.go
```
package main

import (
	"context"
	"log"
	"time"
	pb "grpcc/gopd"
	"google.golang.org/grpc"
)

const (
	Address = "127.0.0.1:50052"
)

func main() {
	conn, err := grpc.Dial(Address, grpc.WithInsecure())
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()

	client := pb.NewProductInfoClient(conn)

	name := "mash name"
	description := "grpc name descirtionsssss"

	ctx, cancel := context.WithTimeout(context.Background(), time.Second)
	defer cancel()

	r, err := client.AddProduct(ctx, &pb.Product{Name:name, Description:description})
	if err != nil {
		log.Fatalf("cound not add product : %v", err)
	}

	log.Printf("proudct id  %s added successfully", r.Value)

	product, err := client.GetProduct(ctx, &pb.ProductID{Value: r.Value})
	if err != nil {
		log.Fatalf("could not get product： %v", err)
	}

	log.Printf("Product is : ", product.String())
}
```
### 运行两端
```
go run server.go
go run client.go
```

### 使用php创建客户端
#### 通过proto文件生成客户端接口文件
```
protoc -I ./ --php_out=../ --grpc_out=../ --plugin=protoc-gen-grpc=/usr/local/bin/grpc_php_plugin  ProductInfo.proto
```
#### 创建composer.json文件
```
{
    "require": {
      "grpc/grpc": "^v1.34.0",
      "google/protobuf": "^v3.14.0"
    },
    "autoload": { 
      "psr-4": {
        "Ecommerce\\": "Ecommerce/", 
        "GPBMetadata\\": "GPBMetadata/", 
        "Proto\\": "Proto/"  
      }
    }
  }
```
#### 创建php的客户端文件 client.php
```
<?php
require __DIR__ . '/vendor/autoload.php';

$client = new \Ecommerce\ProductInfoClient(
    '127.0.0.1:50052', [
        'credentials' => \Grpc\ChannelCredentials::createInsecure()
    ]
    );

$product = new \Ecommerce\Product();
$product->setId('123'); //不用
$product->setName("php go");
$product->setDescription("php go go ssdfsdf");

$dd = $client->addProduct($product);
var_dump($dd);

list($Id, $status) = $client->addProduct($product)->wait();
var_dump($status, $Id->getValue());

//实例化Id类
$Id2 = new \Ecommerce\ProductID();
//赋值
//$Id->setValue('d4d71589-8f7f-4637-96f0-0597f75d21a9');
$Id2->setValue($Id->getValue());
//调用
/**
 * @var $User \Proto\UserInfo
 */
list($productInfo, $status) = $client->getProduct($Id2)->wait();
var_dump($status, $productInfo->getId(), $productInfo->getName(), $productInfo->getDescription());
```
#### 运行client
```
php client.php
```
### 基本使用完成
### grpc stream
### grpc拦截器
### grpc安全性