---
title: go标准库学习之bufio
date: 2020-12-29 09:22:48
tags: 
- golang
categories:
- golang 
---

### 使用Scanner扫描文件
```
package main

import (
	"fmt"
	"bufio"
	"os"
)

func main() {
	file, err := os.Open("./data.txt")	//打开文件
	if err != nil {
		fmt.Println(err)
		return
	}
	defer file.Close()					//关闭文件

	scanner := bufio.NewScanner(file)	//生成文件scanner

	for scanner.Scan() {				//逐行扫描
		fmt.Println(scanner.Text())		//返回行的文本
	}

	if err := scanner.Err(); err != nil {
		fmt.Fprintln(os.Stderr, "reading standard input:", err)
	}

}
```

### 使用scanner统计单词数量
```
package main

import (
	"fmt"
	"bufio"
	"os"
	"strings"
)

func main() {
	const input = "Now is the winter of our discontent,\nMade glorious summer by this sun of York.\n"
	scanner := bufio.NewScanner(strings.NewReader(input))

	scanner.Split(bufio.ScanWords)          //分割

	count := 0

	for scanner.Scan() {
		count++
		fmt.Println(scanner.Text())
	}

	if err := scanner.Err(); err != nil {
		fmt.Fprintln(os.Stderr, "reading input:", err)
	}

	fmt.Printf("%d \n", count)
}
```
