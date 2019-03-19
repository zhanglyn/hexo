---
title: RSA生成秘钥对&加解密&加签验签
date: 2019-03-19 11:11:37
tags: RSA
---

## **RSA加密简介** 

- RSA加密是一种非对称加密。可以在不直接传递密钥的情况下，完成解密。这能够确保信息的安全性，避免了直接传递密钥所造成的被破解的风险。是由一对密钥来进行加解密的过程，分别称为公钥和私钥。两者之间有数学相关，该加密算法的原理就是对一极大整数做因数分解的困难性来保证安全性。通常个人保存私钥，公钥是公开的（可能同时多人持有）。 

------

#### RSA概括

**公钥加密、私钥解密、私钥签名、公钥验签。** 

------

#### GO语言生成RSA秘钥对

<u>生成秘钥对供JAVA对接，因此使用x509.MarshalPKCS8PrivateKey方法生成PKCS#8字节的秘钥</u>

```
package main

import (

    "crypto/rand"

    "crypto/rsa"

    "crypto/x509"

    "encoding/pem"

    "os"

)

 

func main() {

    //rsa 密钥文件产生

    GenRsaKey(1024)

}

 

func GenRsaKey(bits int) error {

    // 生成私钥文件

    privateKey, err := rsa.GenerateKey(rand.Reader, bits)

    if err != nil {

        return err

    }

    derStream, _ := x509.MarshalPKCS8PrivateKey(privateKey)

    block := &pem.Block{

        Type:  "RSA PRIVATE KEY",

        Bytes: derStream,

    }

    file, err := os.Create("private.pem")

    if err != nil {

        return err

    }

    err = pem.Encode(file, block)

    if err != nil {

        return err

    }

    // 生成公钥文件

    publicKey := &privateKey.PublicKey

    derPkix, err := x509.MarshalPKIXPublicKey(publicKey)

    if err != nil {

        return err

    }

    block = &pem.Block{

        Type:  "PUBLIC KEY",

        Bytes: derPkix,

    }

    file, err = os.Create("public.pem")

    if err != nil {

        return err

    }

    err = pem.Encode(file, block)

    if err != nil {

        return err

    }

    return nil

}
```



------

## GO语言RSA加解密

- ##### 公钥加密（分段）：

```
// RSAEncrypt 解析公钥

func RSAEncrypt(origData []byte) ([]byte, error) {

    //解密pem格式的公钥

    block, _ := pem.Decode(publicKey)

    if block == nil {

        return nil, errors.New("public key error")

    }

    // 解析公钥

    pubInterface, err := x509.ParsePKIXPublicKey(block.Bytes)

    if err != nil {

        return nil, err

    }

    // 类型断言

    pub := pubInterface.(*rsa.PublicKey)

    //加密

    return rsaEncrypt(pub, string(origData))

}

 

// rsaEncrypt 数据加密

func rsaEncrypt(pub *rsa.PublicKey, data string) ([]byte, error) {

    partLen := pub.N.BitLen()/8 - 11

    chunks := split([]byte(data), partLen)

 

    buffer := bytes.NewBufferString("")

    for _, chunk := range chunks {

        bts, err := rsa.EncryptPKCS1v15(rand.Reader, pub, chunk)

        if err != nil {

            return nil, err

        }

        buffer.Write(bts)

    }

 

    return buffer.Bytes(), nil

}

 

func split(buf []byte, lim int) [][]byte {

    var chunk []byte

    chunks := make([][]byte, 0, len(buf)/lim+1)

    for len(buf) >= lim {

        chunk, buf = buf[:lim], buf[lim:]

        chunks = append(chunks, chunk)

    }

    if len(buf) > 0 {

        chunks = append(chunks, buf[:])

    }

    return chunks

}
```

- ##### 私钥解密（分段）：

<u>因对接java，故使用x509.ParsePKCS8PrivateKey()读取私钥，分段解密</u>

```
// rsaPrivateDe 解密读取秘钥

func rsaPrivateDe(cryptoText string) (string, error) {

    block, _ := pem.Decode(privateKey)

    if block == nil {

        return "", errors.New("private key error!")

    }

    privInterface, err := x509.ParsePKCS8PrivateKey(block.Bytes)

    if err != nil {

        return "", err

    }

    // 类型断言

    priv, _ := privInterface.(*rsa.PrivateKey)

    plainText, err := RsaPrivateDecrypt(priv, cryptoText)

    if err != nil {

        fmt.Println("RsaPrivateDecrypt error ：", err)

        return "", err

    }

    return plainText, nil

}

 

// RsaPrivateDecrypt 私钥解密

func RsaPrivateDecrypt(priv *rsa.PrivateKey, encrypted string) (string, error) {

    partLen := 1024 / 8

    raw, err := base64.StdEncoding.DecodeString(encrypted)

    chunks := split([]byte(raw), partLen)

 

    buffer := bytes.NewBufferString("")

    for _, chunk := range chunks {

        decrypted, err := rsa.DecryptPKCS1v15(rand.Reader, priv, chunk)

        if err != nil {

            return "", err

        }

        buffer.Write(decrypted)

    }

    return buffer.String(), err

}
```

------

## GO语言RSA加签验签

- ##### 私钥加签：

```
// 私钥加签

func RsaSignWithSha1Hex(data string) (string, error) {

    block, _ := pem.Decode(privateKey)

    if block == nil {

        return "", errors.New("private key error!")

    }

    privInterface, err := x509.ParsePKCS8PrivateKey(block.Bytes)

    if err != nil {

        return "", err

    }

    // 类型断言

    priv, _ := privInterface.(*rsa.PrivateKey)

    h := md5.New()  // 对接JAVA使用MD5加密（也可以使用sha1.New()进行加密，但JAVA需要使用Signature signature = Signature.getInstance("SHA1WithRSA")验签） 

    h.Write([]byte([]byte(data)))

    hash := h.Sum(nil)

    signature, err := rsa.SignPKCS1v15(rand.Reader, priv, crypto.MD5, hash[:])  // 使用MD5加签

    if err != nil {

        return "", err

    }

    out := base64.StdEncoding.EncodeToString(signature)  // 加签使用base64编码，验签时需先base64解码

    return out, nil

}
```

- ##### 公钥验签：

```
func RsaVerySignWithSha1Base64(originalData, signData string) error {

​    sign, err := base64.StdEncoding.DecodeString(signData) // base64解码

​    if err != nil {

​        return err

​    }

 

​    //解密pem格式的公钥

​    block, _ := pem.Decode(pub)

​    if block == nil {

​        return errors.New("public key error")

​    }

​    // 解析公钥

​    pubInterface, err := x509.ParsePKIXPublicKey(block.Bytes)

​    if err != nil {

​        return err

​    }

​    // 类型断言

​    pub := pubInterface.(*rsa.PublicKey)

​    if err != nil {

​        return err

​    }

​    hash := sha1.New() // 使用SHA1算法验签

​    hash.Write([]byte(originalData))

​    return rsa.VerifyPKCS1v15(pub, crypto.SHA1, hash.Sum(nil), sign)

}
```

