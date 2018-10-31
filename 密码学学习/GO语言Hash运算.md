---
title: GO语言Hash运算
copyright: true
date: 2018-10-19 11:04:08
tags:
     - go语言
     - 加密算法
categories: GO语言
---

散列函数（散列算法，又称哈希函数）是一种从任何一种数据中创建小的数字“指纹”的方法。散列函数把消息或数据压缩成摘要，使得数据量变小，将数据的格式固定下来。该函数将数据打乱混合，重新创建一个叫做散列值的指纹。

### 随机生成

加密密钥需要尽可能的随机，以便生成的密钥很难再现。加密随机数生成器必须生成无法通过计算方法推算出（低于p<.05的概率）的输出。

### 散列函数

基本特性：如果两个散列值是不相同的（根据同一函数），那么这两个散列值的原始输入也是不相同的。这个特性是散列函数具有确定性的结果，具有这种性质的散列函数称为单向散列函数。但另一方面，散列函数的输入和输出不是唯一对应关系的，如果两个散列值相同，两个输入值很可能是相同的，但也可能不同，这种情况称为“散列碰撞”。

### 主要应用场景

- 文件校验
- 数字签名
- 鉴权协议

### Go语言支持

go crypto标准包包含了一些常用的哈希算法，例如md5、sha1、sha256、sha512等。以md5算法为例，了解下go如何生成哈希值。

```
package main
 
import (
	"crypto/md5"
	"encoding/hex"
	"fmt"
)
/*
  使用MD5对数据进行哈希运算方法1：使用md5.Sum()方法
*/
func getMD5String_1(b[]byte) (result string) {
	//给哈希算法添加数据
	res:=md5.Sum(b) //返回值：[Size]byte 数组
	/*//方法1：
	result=fmt.Sprintf("%x",res)   //通过fmt.Sprintf()方法格式化数据
	*/
	//方法2：
	result=hex.EncodeToString(res[:])  //对应的参数为：切片，需要将数组转换为切片。
	return
}
/*
使用MD5对数据进行哈希运算方法2：使用md5.new()方法
*/
func getMD5String_2(b[]byte) (result string) {
	//1、创建Hash接口
	myHash:=md5.New()  //返回 Hash interface
	//2、添加数据
	myHash.Write(b)  //写入数据
	//3、计算结果
	/*
	  执行原理为：myHash.Write(b1)写入的数据进行hash运算  +  myHash.Sum(b2)写入的数据进行hash运算
	              结果为：两个hash运算结果的拼接。若myHash.Write()省略或myHash.Write(nil) ，则默认为写入的数据为“”。
	              根据以上原理，一般不采用两个hash运算的拼接，所以参数为nil
	*/
	res:=myHash.Sum(nil)  //进行运算
	//4、数据格式化
	result=hex.EncodeToString(res) //转换为string
	return
}
func main() {
	str:=[]byte("jiangzhou")
	res:=getMD5String_1(str)
	fmt.Println("方法1运算结果：",res)
 
	res=getMD5String_2(str)
	fmt.Println("方法2运算结果：",res)
	
}
```

