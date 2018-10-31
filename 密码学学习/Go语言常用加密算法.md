---
title: Go语言常用加密算法
copyright: true
date: 2018-10-14 12:58:39
tags:
     - go语言
     - 加密算法
categories: GO语言
---

# 常见的加密算法

原文链接：<https://studygolang.com/articles/10134>

在项目开发过程中，当我们利用数据库存储一些关于用户的隐私信息，诸如密码、帐户密钥等数据时，需要加密后才向数据库写入。这时，我们需要一些高效地、简单易用的加密算法，当我们向数据库写数据时加密数据，然后把加密后的数据存入数据库；当需要读取数据时，从数据库把加密后的数据取出来，再通过算法解密。

常用的加密算法有Base64、MD5、AES和DES。

### Base64

Base64是一种任意二进制到文本字符串的编码方法，常用于在URL、Cookie、网页中传输少量二进制数据。

首先使用Base64编码需要一个含有64个字符的表，这个表由大小写字母、数字、+和/组成。采用Base64编码处理数据时，会把每三个字节共24位作为一个处理单元，再分为四组，每组6位，查表后获得相应的字符即编码后的字符串。编码后的字符串长32位，这样，经Base64编码后，原字符串增长1/3。如果要编码的数据不是3的倍数，最后会剩下一到两个字节，Base64编码中会采用\x00在处理单元后补全，编码后的字符串最后会加上一到两个 **=** 表示补了几个字节。

Base64表

```
 BASE64Table = "IJjkKLMNO567PQX12RVW3YZaDEFGbcdefghiABCHlSTUmnopqrxyz04stuvw89+/"
```

加密。函数里的第二行的代码可以把一个输入的字符串转换成一个字节数组。 

```
func Encode(data string) string {
    content := *(*[]byte)(unsafe.Pointer((*reflect.SliceHeader)(unsafe.Pointer(&data))))
    coder := base64.NewEncoding(BASE64Table)
    return coder.EncodeToString(content)
}
```

解密。函数返回处的代码可以把一个 字节数组转换成一个字符串。 

```
func Decode(data string) string {
    coder := base64.NewEncoding(BASE64Table)
    result, _ := coder.DecodeString(data)
    return *(*string)(unsafe.Pointer(&result))
}
```

测试。 

```
 func main(){
      strTest := "I love this beautiful world!"
      strEncrypt := Encode(strTest)
      strDecrypt := Decode(strEncrypt)
      fmt.Println("Encrypted:",strEncrypt)
      fmt.Println("Decrypted:",strDecrypt)
    }
//Output:
//Encrypted: VVJmGsEBONRlFaPfDCYgcaRSEHYmONcpbCrAO2==
//Decrypted: I love this beautiful world!
```

### MD5

MD5的全称是Message-DigestAlgorithm 5，它可以把一个任意长度的字节数组转换成一个定长的整数，并且这种转换是不可逆的。对于任意长度的数据，转换后的MD5值长度是固定的，而且MD5的转换操作很容易，只要原数据有一点点改动，转换后结果就会有很大的差异。正是由于MD5算法的这些特性，它经常用于对于一段信息产生信息摘要，以防止其被篡改。其还广泛就于操作系统的登录过程中的安全验证，比如Unix操作系统的密码就是经过MD5加密后存储到文件系统中，当用户登录时输入密码后， 对用户输入的数据经过MD5加密后与原来存储的密文信息比对，如果相同说明密码正确，否则输入的密码就是错误的。

MD5以512位为一个计算单位对数据进行分组，每一分组又被划分为16个32位的小组，经过一系列处理后，输出4个32位的小组，最后组成一个128位的哈希值。对处理的数据进行512求余得到N和一个余数，如果余数不为448,填充1和若干个0直到448位为止，最后再加上一个64位用来保存数据的长度，这样经过预处理后，数据变成（N+1）x 512位。

加密。Encode 函数用来加密数据，Check函数传入一个未加密的字符串和与加密后的数据，进行对比，如果正确就返回true。

```

func Check(content, encrypted string) bool {
    return strings.EqualFold(Encode(content), encrypted)
}
func Encode(data string) string {
    h := md5.New()
    h.Write([]byte(data))
    return hex.EncodeToString(h.Sum(nil))
}
```

测试。 

```
func main() {
     strTest := "I love this beautiful world!"
    strEncrypted := "98b4fc4538115c4980a8b859ff3d27e1"
    fmt.Println(Check(strTest, strEncrypted))
}
//Output:
//true
```

### DES

DES是一种对称加密算法，又称为美国数据加密标准。DES加密时以64位分组对数据进行加密，加密和解密都使用的是同一个长度为64位的密钥，实际上只用到了其中的56位，密钥中的第8、16...64位用来作奇偶校验。DES有ECB（电子密码本）和CBC（加密块）等加密模式。

DES算法的安全性很高，目前除了穷举搜索破解外， 尚无更好的的办法来破解。其密钥长度越长，破解难度就越大。

填充和去填充函数。

```
func ZeroPadding(ciphertext []byte, blockSize int) []byte {
    padding := blockSize - len(ciphertext)%blockSize
    padtext := bytes.Repeat([]byte{0}, padding)
    return append(ciphertext, padtext...)
}
 
func ZeroUnPadding(origData []byte) []byte {
    return bytes.TrimFunc(origData,
        func(r rune) bool {
            return r == rune(0)
        })
}
```

加密。 

```
func Encrypt(text string, key []byte) (string, error) {
    src := []byte(text)
    block, err := des.NewCipher(key)
    if err != nil {
        return "", err
    }
    bs := block.BlockSize()
    src = ZeroPadding(src, bs)
    if len(src)%bs != 0 {
        return "", errors.New("Need a multiple of the blocksize")
    }
    out := make([]byte, len(src))
    dst := out
    for len(src) > 0 {
        block.Encrypt(dst, src[:bs])
        src = src[bs:]
        dst = dst[bs:]
    }
    return hex.EncodeToString(out), nil
}
```

解密。 

```

func Decrypt(decrypted string , key []byte) (string, error) {
    src, err := hex.DecodeString(decrypted)
    if err != nil {
        return "", err
    }
    block, err := des.NewCipher(key)
    if err != nil {
        return "", err
    }
    out := make([]byte, len(src))
    dst := out
    bs := block.BlockSize()
    if len(src)%bs != 0 {
        return "", errors.New("crypto/cipher: input not full blocks")
    }
    for len(src) > 0 {
        block.Decrypt(dst, src[:bs])
        src = src[bs:]
        dst = dst[bs:]
    }
    out = ZeroUnPadding(out)
    return string(out), nil
}
```

测试。在这里，DES中使用的密钥key只能为8位。 

```
func main() {
    key := []byte("2fa6c1e9")
    str :="I love this beautiful world!"
    strEncrypted, err := Encrypt(str, key)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println("Encrypted:", strEncrypted)
    strDecrypted, err := Decrypt(strEncrypted, key)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println("Decrypted:", strDecrypted)
}
//Output:
//Encrypted: 5d2333b9fbbe5892379e6bcc25ffd1f3a51b6ffe4dc7af62beb28e1270d5daa1
//Decrypted: I love this beautiful world!
```

### AES

AES，即高级加密标准（Advanced Encryption Standard），是一个对称分组密码算法，旨在取代DES成为广泛使用的标准。AES中常见的有三种解决方案，分别为AES-128、AES-192和AES-256。

AES加密过程涉及到4种操作：字节替代（SubBytes）、行移位（ShiftRows）、列混淆（MixColumns）和轮密钥加（AddRoundKey）。解密过程分别为对应的逆操作。由于每一步操作都是可逆的，按照相反的顺序进行解密即可恢复明文。加解密中每轮的密钥分别由初始密钥扩展得到。算法中16字节的明文、密文和轮密钥都以一个4x4的矩阵表示。

 

AES 有五种加密模式：电码本模式（Electronic Codebook Book (ECB)）、密码分组链接模式（Cipher Block Chaining (CBC)）、计算器模式（Counter (CTR)）、密码反馈模式（Cipher FeedBack (CFB)）和输出反馈模式（Output FeedBack (OFB)）。下面以CFB为例。

加密。iv即初始向量，这里取密钥的前16位作为初始向量。

```
func Encrypt(text string, key []byte) (string, error) {
     var iv = key[:aes.BlockSize]
    encrypted := make([]byte, len(text))
    block, err := aes.NewCipher(key)
    if err != nil {
        return "", err
    }
    encrypter := cipher.NewCFBEncrypter(block, iv)
    encrypter.XORKeyStream(encrypted, []byte(text))
    return hex.EncodeToString(encrypted), nil
}
```

解密。 

```
func Decrypt(encrypted string, key []byte) (string, error) {
    var err error
    defer func() {
        if e := recover(); e != nil {
            err = e.(error)
        }
    }()
    src, err := hex.DecodeString(encrypted)
    if err != nil {
        return "", err
    }
    var iv = key[:aes.BlockSize]
    decrypted := make([]byte, len(src))
    var block cipher.Block
    block, err = aes.NewCipher([]byte(key))
    if err != nil {
        return "", err
    }
    decrypter := cipher.NewCFBDecrypter(block, iv)
    decrypter.XORKeyStream(decrypted, src)
    return string(decrypted), nil
}
```

测试。密钥key只能为16位、24位或32位，分别对应AES-128, AES-192和 AES-256。 

```
func main(){
    str := "I love this beautiful world!"
    key := []byte{0xBA, 0x37, 0x2F, 0x02, 0xC3, 0x92, 0x1F, 0x7D,
        0x7A, 0x3D, 0x5F, 0x06, 0x41, 0x9B, 0x3F, 0x2D,
        0xBA, 0x37, 0x2F, 0x02, 0xC3, 0x92, 0x1F, 0x7D,
        0x7A, 0x3D, 0x5F, 0x06, 0x41, 0x9B, 0x3F, 0x2D,
        }
    strEncrypted,err := Encrypt(str, key)
    if err != nil {
        log.Error("Encrypted err:",err)
    }
    fmt.Println("Encrypted:",strEncrypted)
    strDecrypted,err := Decrypt(strEncrypted, key)
    if err != nil {
        log.Error("Decrypted err:",err)
    }
    fmt.Println("Decrypted:",strDecrypted)
}
//Output:
//Encrypted: f866bfe2a36d5a43186a790b41dc2396234dd51241f8f2d4a08fa5dc
//Decrypted: I love this beautiful world!
```

### 参考

- [Golang中的DES加密ECB模式](http://www.100hack.com/2014/04/14/golang%E4%B8%AD%E7%9A%84des%E5%8A%A0%E5%AF%86ecb%E6%A8%A1%E5%BC%8F/)
- [AES五种加密模式（CBC、ECB、CTR、OCF、CFB)](http://www.cnblogs.com/starwolf/p/3365834.html)