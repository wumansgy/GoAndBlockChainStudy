---
title: 非对称加密RSA详解
copyright: true
date: 2018-11-03 12:51:19
tags:
     - go语言
     - 加密算法
categories: GO语言
---

### 非对称加密通信流程

下面我们来看一看使用公钥密码的通信流程。和以前一样、我们还是假设Alice要给Bob发送一条消息，Alice是发送者，Bob是接收者，而这一次窃听者Eve依然能够窃所到他们之间的通信内容。

在公非对称加密通信中，通信过程是由接收者Bob来启动的。

1. Bob生成一个包含公钥和私钥的密钥对。

   > 私钥由Bob自行妥善保管。

2. Bob将自己的公钥发送给Alice

   > Bob的公钥被窃听者Eve截获也没关系。
   >
   > 将公钥发送给Alice，表示Bob请Alice用这个公钥对消息进行加密并发送给他。

3. Alice用Bob的公钥对消息进行加密。

   > 加密后的消息只有用Bob的私钥才能够解密。
   >
   > 虽然Alice拥有Bob的公钥，但用Bob的公钥是无法对密文进行解密的。

4. Alice将密文发送给Bobo

   > 密文被窃听者Eve截获也没关系。Eve可能拥有Bob的公钥，但是用Bob的公钥是无法进行解密的。

5. Bob用自己的私钥对密文进行解密。

   > 请参考下图, 看一看在Alice和Bob之间到底传输了哪些信息。其实它们之间所传输的信息只有两个：Bob的公钥以及用Bob的公钥加密的密文。由于Bob的私钥没有出现在通信内容中，因此窃听者Eve无法对密文进行解密。

![](非对称加密RSA详解/1.png)

> 窃听者Eve可能拥有Bob的公钥，但是Bob的公钥只是加密密钥，而不是解密密钥，因此窃听者Eve就无法完成解密操作。

#### 1.RSA

```
非对称加密的密钥分为加密密钥和解密密钥，但这到底是怎样做到的呢？本节中我们来讲解现在使用最广泛的公钥密码算法一一RSA。

RSA是一种非对称加密算法，它的名字是由它的三位开发者，即RonRivest、AdiShamir和LeonardAdleman 的姓氏的首字母组成的（Rivest-Shamir-Adleman ）。

RSA可以被用于非对称加密和数字签名，关于数字签名我们将在后面章节进行讲解。

1983年，RSA公司为RSA算法在美国取得了专利，但现在该专利已经过期。
```

#### 2.RSA加密

```
> 下面我们终于可以讲一讲非对称加密的代表—RSA的加密过程了。在RSA中，明文、密钥和密文都是数字。RSA的加密过程可以用下列公式来表达，如下。


密文=明文 ^ E  mod     N（RSA加密）


> 也就是说，RSA的密文是对代表明文的数字的E次方求modN的结果。换句话说，就是将明文自己做E次乘法，然后将其结果除以N求余数，这个余数就是密文。
>
> 咦，就这么简单？
>
> 对，就这么简单。仅仅对明文进行乘方运算并求mod即可，这就是整个加密的过程。在对称密码中，出现了很多复杂的函数和操作，就像做炒鸡蛋一样将比特序列挪来挪去，还要进行XOR(按位异或)等运算才能完成，但RSA却不同，它非常简洁。
>
> 对了，加密公式中出现的两个数一一一E和N，到底都是什么数呢？RSA的加密是求明文的E次方modN，因此只要知道E和N这两个数，任何人都可以完成加密的运算。所以说，E和N是RSA加密的密钥，也就是说，**E和N的组合就是公钥**。
>
> 不过，E和N并不是随便什么数都可以的，它们是经过严密计算得出的。顺便说一句，E是加密（Encryption）的首字母，N是数字（Number)的首字母。
>
> 有一个很容易引起误解的地方需要大家注意一一E和N这两个数并不是密钥对（公钥和私钥的密钥对）。E和N两个数才组成了一个公钥，因此我们一般会写成 “公钥是(E，N)” 或者 “公钥是{E, N}" 这样的形式，将E和N用括号括起来。
>
> 现在大家应该已经知道，==RSA的加密就是 “求E次方的modN"==，接下来我们来看看RSA的解密。
```

#### 3.RSA解密

```
> RSA的解密和加密一样简单，可以用下面的公式来表达：


明文=密文^DmodN（RSA解密）


> 也就是说，对表示密文的数字的D次方求modN就可以得到明文。换句话说，将密文自己做D次乘法，再对其结果除以N求余数，就可以得到明文。
>
> 这里所使用的数字N和加密时使用的数字N是相同的。数D和数N组合起来就是RSA的解密密钥，因此D和N的组合就是私钥。只有知道D和N两个数的人才能够完成解密的运算。
>
> 大家应该已经注意到，在RSA中，加密和解密的形式是相同的。加密是求 "E次方的mod N”，而解密则是求 "D次方的modN”，这真是太美妙了。
>
> 当然，D也并不是随便什么数都可以的，作为解密密钥的D，和数字E有着相当紧密的联系。否则，用E加密的结果可以用D来解密这样的机制是无法实现的。
>
> 顺便说一句，D是解密〈Decryption）的首字母，N是数字（Number）的首字母。
>
```

> 我们将上面讲过的内容整理一下，如下表所示。

![](非对称加密RSA详解/2.png)

RSA的加密和解密

![](非对称加密RSA详解/3.png)

####  Go中生成公钥和私钥

需要引入的包

```go
import (
	"crypto/rsa"
	"crypto/rand"
	"crypto/x509"
	"encoding/pem"
	"os"
)
```

生成私钥操作流程概述

> 1. 使用rsa中的GenerateKey方法生成私钥
> 2. 通过x509标准将得到的ras私钥序列化为ASN.1 的 DER编码字符串
> 3. 将私钥字符串设置到pem格式块中
> 4. 通过pem将设置好的数据进行编码, 并写入磁盘文件中

生成公钥操作流程

> 1. 从得到的私钥对象中将公钥信息取出
> 2. 通过x509标准将得到 的rsa公钥序列化为字符串
> 3. 将公钥字符串设置到pem格式块中
> 4. 通过pem将设置好的数据进行编码, 并写入磁盘文件

生成公钥和私钥的源代码

```
// 参数bits: 指定生成的秘钥的长度, 单位: bit
func RsaGenKey(bits int) error{
	// 1. 生成私钥文件
	// GenerateKey函数使用随机数据生成器random生成一对具有指定字位数的RSA密钥
	// 参数1: Reader是一个全局、共享的密码用强随机数生成器
	// 参数2: 秘钥的位数 - bit
	privateKey, err := rsa.GenerateKey(rand.Reader, bits)
	if err != nil{
		return err
	}
	// 2. MarshalPKCS1PrivateKey将rsa私钥序列化为ASN.1 PKCS#1 DER编码
	derStream := x509.MarshalPKCS1PrivateKey(privateKey)
	// 3. Block代表PEM编码的结构, 对其进行设置
	block := pem.Block{
		Type: "RSA PRIVATE KEY",//"RSA PRIVATE KEY",
		Bytes: derStream,
	}
	// 4. 创建文件
	privFile, err := os.Create("private.pem")
	if err != nil{
		return err
	}
	// 5. 使用pem编码, 并将数据写入文件中
	err = pem.Encode(privFile, &block)
	if err != nil{
		return err
	}
	// 6. 最后的时候关闭文件
	defer privFile.Close()

	// 7. 生成公钥文件
	publicKey := privateKey.PublicKey
	derPkix, err := x509.MarshalPKIXPublicKey(&publicKey)
	if err != nil{
		return err
	}
	block = pem.Block{
		Type: "RSA PUBLIC KEY",//"PUBLIC KEY",
		Bytes: derPkix,
	}
	pubFile, err := os.Create("public.pem")
	if err != nil{
		return err
	}
	// 8. 编码公钥, 写入文件
	err = pem.Encode(pubFile, &block)
	if err != nil{
		panic(err)
		return err
	}
	defer pubFile.Close()

	return nil

}
```

调用完会在本地生成两个文件。保存好私钥文件和公钥文件

![](非对称加密RSA详解/4.png)

![](非对称加密RSA详解/5.png)

![](非对称加密RSA详解/6.png)

重要的函数介绍:

1. GenerateKey函数使用随机数据生成器random生成一对具有指定字位数的RSA密钥。

   ```go 
   "crypto/rsa" 包中的函数
   func GenerateKey(random io.Reader, bits int) (priv *PrivateKey, err error)
       - 参数1: io.Reader: 赋值为: rand.Reader
           -- rand包实现了用于加解密的更安全的随机数生成器。
           -- var Reader io.Reader (rand包中的变量)
       - 参数2: bits: 秘钥长度
       - 返回值1: 代表一个RSA私钥。
       - 返回值2: 错误信息
   
   ```

2. 通过x509 将rsa私钥序列化为ASN.1 PKCS1 DER编码

   ```go
   "crypto/x509" 包中的函数 (x509包解析X.509编码的证书和密钥)。
   func MarshalPKCS1PrivateKey(key *rsa.PrivateKey) []byte
       - 参数1: 通过rsa.GenerateKey得到的私钥
       - 返回值: 将私钥通过ASN.1序列化之后得到的私钥编码数据
   ```

3. 设置Pem编码结构

   ```go
   Block代表PEM编码的结构。
   type Block struct {
       Type    string            // 得自前言的类型（如"RSA PRIVATE KEY"）
       Headers map[string]string // 可选的头项，Headers是可为空的多行键值对。
       Bytes   []byte            // 内容解码后的数据，一般是DER编码的ASN.1结构
   }
   ```

4. 将得到的Pem格式私钥通过文件指针写入磁盘中

   ```go
   "encoding/pem" 包中的函数
   func Encode(out io.Writer, b *Block) error
       - 参数1: 可进行写操作的IO对象, 此处需要指定一个文件指针
       - 参数2: 初始化完成的Pem块对象, 即Block对象
   ```

5. 通过RSA私钥得到公钥

   ```go
   // 私钥
   type PrivateKey struct {
       PublicKey            // 公钥
       D         *big.Int   // 私有的指数
       Primes    []*big.Int // N的素因子，至少有两个
       // 包含预先计算好的值，可在某些情况下加速私钥的操作
       Precomputed PrecomputedValues
   }
   // 公钥
   type PublicKey struct {
       N   *big.Int // 模
       E   int      // 公开的指数
   }
   通过私钥获取公钥
   publicKey := privateKey.PublicKey // privateKey为私钥对象
   ```

6. 通过x509将公钥序列化为PKIX格式DER编码。

   ```go
   "crypto/x509" 包中的函数
   func MarshalPKIXPublicKey(pub interface{}) ([]byte, error)
       - 参数1: 通过私钥对象得到的公钥
       - 返回值1：将公钥通过ASN.1序列化之后得到的编码数据
       - 返回值2: 错误信息
   ```

7. 将公钥编码之后的数据格式化为Pem结构, 参考私钥的操作

8. 将得到的Pem格式公钥通过文件指针写入磁盘中

9. 生成的私钥和公钥文件数据

## Go中使用RSA

1. 操作步骤

   - 公钥加密

     > 1. 将公钥文件中的公钥读出, 得到使用pem编码的字符串
     > 2. 将得到的字符串解码
     > 3. 使用x509将编码之后的公钥解析出来
     > 4. 使用得到的公钥通过rsa进行数据加密

   - 私钥解密

     > 1. 将私钥文件中的私钥读出, 得到使用pem编码的字符串
     > 2. 将得到的字符串解码
     > 3. 使用x509将编码之后的私钥解析出来
     > 4. 使用得到的私钥通过rsa进行数据解密

2. 代码实现

   - RSA公钥加密

     ```go
     func RSAEncrypt(src, filename []byte) []byte {
         // 1. 根据文件名将文件内容从文件中读出
         file, err := os.Open(string(filename))
         if err != nil {
             return nil
         }
         // 2. 读文件
         info, _ := file.Stat()
         allText := make([]byte, info.Size())
         file.Read(allText)
         // 3. 关闭文件
         file.Close()
     
         // 4. 从数据中查找到下一个PEM格式的块
         block, _ := pem.Decode(allText)
         if block == nil {
             return nil
         }
         // 5. 解析一个DER编码的公钥
         pubInterface, err := x509.ParsePKIXPublicKey(block.Bytes)
         if err != nil {
             return nil
         }
         pubKey := pubInterface.(*rsa.PublicKey)
     
         // 6. 公钥加密
         result, _ := rsa.EncryptPKCS1v15(rand.Reader, pubKey, src)
         return result
     }
     ```

   - RSA私钥解密

     ```go
     func RSADecrypt(src, filename []byte) []byte {
       // 1. 根据文件名将文件内容从文件中读出
       file, err := os.Open(string(filename))
       if err != nil {
           return nil
       }
       // 2. 读文件
       info, _ := file.Stat()
       allText := make([]byte, info.Size())
       file.Read(allText)
       // 3. 关闭文件
       file.Close()
       // 4. 从数据中查找到下一个PEM格式的块
       block, _ := pem.Decode(allText)
       // 5. 解析一个pem格式的私钥
       privateKey , err := x509.ParsePKCS1PrivateKey(block.Bytes)
       // 6. 私钥解密
       result, _ := rsa.DecryptPKCS1v15(rand.Reader, privateKey, src)
     
         return result
       }
     ```

   - **重要的函数介绍**

     1. 将得到的Pem格式私钥通过文件指针写入磁盘中

        ```go
        "encoding/pem" 包中的函数
        func Decode(data []byte) (p *Block, rest []byte)
            - 参数 data: 需要解析的数据块
            - 返回值1: 从参数中解析出的PEM格式的块
            - 返回值2: 参数data剩余的未被解码的数据
        ```

     2. 解析一个DER编码的公钥 , pem中的Block结构体中的数据格式为ASN.1编码

        ```go
        函数所属的包: "crypto/x509"
        func ParsePKIXPublicKey(derBytes []byte) (pub interface{}, err error)
            - 参数 derBytes: 从pem的Block结构体中取的ASN.1编码数据
            - 返回值 pub: 接口对象, 实际是公钥数据
            - 参数 err:   错误信息
        ```

     3. 解析一个DER编码的私钥 , pem中的Block结构体中的数据格式为ASN.1编码

        ```go
        函数所属的包: "crypto/x509"
        func ParsePKCS1PrivateKey(der []byte) (key *rsa.PrivateKey, err error)
            - 参数 der: 从pem的Block结构体中取的ASN.1编码数据
            - 返回值 key: 解析出的私钥
            - 返回值 err: 错误信息
        ```

     4. 将接口转换为公钥

        ```go
        pubKey := pubInterface.(*rsa.PublicKey)
            - pubInterface: ParsePKIXPublicKey函数返回的 interface{} 对象
            - pubInterface.(*rsa.PublicKey): 将pubInterface转换为公钥类型 rsa.PublicKey
        ```

     5. 使用公钥加密数据

        ```go
        函数所属的包: "crypto/rsa"
        func EncryptPKCS1v15(rand io.Reader, pub *PublicKey, msg []byte) (out []byte, err error)
            - 参数 rand: 随机数生成器, 赋值为 rand.Reader
            - 参数 pub:  非对称加密加密使用的公钥
            - 参数 msg:  要使用公钥加密的原始数据
            - 返回值 out: 加密之后的数据
            - 返回值 err: 错误信息
        ```

     6. 使用私钥解密数据

        ```go
        函数所属的包: "crypto/rsa"
        func DecryptPKCS1v15(rand io.Reader, priv *PrivateKey, ciphertext []byte) (out []byte, err error)
            - 参数 rand: 随机数生成器, 赋值为 rand.Reader
            - 参数 priv: 非对称加密解密使用的私钥
            - 参数 ciphertext: 需要使用私钥解密的数据
            - 返回值 out: 解密之后得到的数据
            - 返回值 err: 错误信
        ```

### 这就是大概RSA的加密解密。