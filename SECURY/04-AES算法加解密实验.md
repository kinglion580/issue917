# AES 算法加解密实验

通过实验学习 AES 加解密算法，通过使用命令行工具测试 AES 加解密，并拆分整个 AES 加解密的过程来深入熟悉算法原理。

## AES 简介

**高级加密标准**（英语：**Advanced Encryption Standard**，缩写：**AES**），在[密码学](https://baike.baidu.com/item/%E5%AF%86%E7%A0%81%E5%AD%A6/480001)中又称 **Rijndael加密法**，是[美国联邦政府](https://baike.baidu.com/item/%E7%BE%8E%E5%9B%BD%E8%81%94%E9%82%A6%E6%94%BF%E5%BA%9C/8370227)采用的一种区块加密标准。它比 DES 速度更快，更安全，被用来替代 DES。

我们经常会听到 AES128，AES192，AES256 ，其实意思是支持不同长度密钥的 AES 算法。

（参考[aes（高级加密标准）_百度百科](https://baike.baidu.com/item/aes/5903)）

## 实验1: 通过命令测试 AES 加解密

还记得上一章节提到的场景吗。同样我们可以使用 AES 算法加解密。

首先新建一个 aes.txt ：

```
echo "i love shiyanlou" > aes.txt
```

然后使用 `openssl` 命令进行 AES 加密：

```
openssl enc -aes-128-cbc -in aes.txt -out aes.bin

# 依然会提示输入密码，我们可以输入 12345
```

> aes.bin 即为得到的密文。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547115469089.png-wm)

同样，使用 openssl 命令可以对 aes.bin 进行解密：

```
openssl enc -aes-128-cbc -d -in aes.bin -out aes2.txt

# 输入的密码跟之前设置的密码一致，为 12345
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547115531305.png-wm)

cbc 代表的是工作模式，在解密时也要用相同的工作模式解密，不然会报错，如下图：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547116027153.png-wm)

## 实验2: 用不同工作模式进行加密

用 `openssl enc --help` 查看帮助，可以看到如下部分内容：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547205183964.png-wm)

可以看到有 cfb，cfb1，ctr，ofb，cbc，ecb 等工作模式。可以每个都试一试。

```
openssl enc -aes-128-cfb -in aes.txt -out cfb.bin

openssl enc -aes-128-cfb1 -in aes.txt -out cfb1.bin

openssl enc -aes-128-ctr -in aes.txt -out ctr.bin

openssl enc -aes-128-cbc -in aes.txt -out cbc.bin

openssl enc -aes-128-ecb -in aes.txt -out ecb.bin
```

还有很多其他命令，都可以尝试一下。用 `cat <filename>` 查看加密过后的密文。

## 实验总结

通过 `openssl enc` 子命令对测试文件进行 AES 加解密，理解了对称加密特点以及 AES 加密的安全性，AES 比 DES 更加安全，并且支持不同长度的密钥。我们之前说过衡量对称算法优劣的主要尺度是其密钥的长度。