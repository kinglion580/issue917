# DES 算法加解密实验

## 实验描述

通过实验学习 DES 加解密算法，通过使用命令行工具测试 DES 加解密，并通过 C 语言实现 DES 加解密算法来深入熟悉算法原理。

## DES 算法简介

DES 全称为 Data Encryption Standard，又称为美国数据加密标准，是一种使用密钥加密的块算法，是 1972 年美国IBM公司研制的对称密码体制加密算法。

DES 加密算法是对密钥进行保密，而公开算法，包括加密和解密算法，只有掌握了和发送方相同密钥的人才能解读由 DES 加密算法加密的密文数据。

## 实验1: 使用命令测试 DES 加解密

首先新建一个内容为 `i love shiyanlou` 的文件 test1.txt：

```
echo i love shiyanlou > test1.txt
```

现在有这样一个场景，我们想把这个文件传给实小楼这个同学，然而并不想让其他人看到，就需要对文件内容进行加密，将密文传给实小楼，实小楼接收到密文过后解密得到明文信息。

实验环境中可以通过 `openssl` 命令行工具来测试 DES 加解密。

> 可以使用 `openssl --help` 或者 	`man openssl` 查看此命令的帮助信息。

下面使用 openssl 命令加密 test1.txt 并将加密后的内容输出到 test.bin 文件中：

```
openssl enc -des -in test1.txt -out test.bin

# 会提示我们输入密码，我们可以输入12345
# 注意：linux 系统中输入密码是看不见的，但是实际上是输入进去了的。
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547110871508.png-wm)

> test.bin 中的内容即为加密过后的密文。

可以使用 `cat test.bin` 查看密文内容。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547111046743.png-wm)

假设我们已经将 test.bin 这个密文传给实小楼，并将 12345 这个密码告诉实小楼，当实小楼收到密文后，可以使用密码 12345 解密文件。

```
openssl enc -des -d -in test.bin -out test2.txt

# 会提示输入密码，输入 12345 即可
```

> `-d` : 解密

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547112516528.png-wm)

可以看出加密解密都是使用的同一密钥，这就是对称加密。

## 实验2: 用 c 语言实现 DES 算法



