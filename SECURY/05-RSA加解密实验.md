# RSA 加解密实验

通过实验了解非对称加密机制。通过使用 Linux 工具来进行非对称加解密，理解 RSA算法的加解密原理。

## RSA 简介

**RSA加密算法**是一种[非对称加密算法](https://baike.baidu.com/item/%E9%9D%9E%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86%E7%AE%97%E6%B3%95/1208652)。在[公开密钥加密](https://baike.baidu.com/item/%E5%85%AC%E5%BC%80%E5%AF%86%E9%92%A5%E5%8A%A0%E5%AF%86/8090774)和[电子商业](https://baike.baidu.com/item/%E7%94%B5%E5%AD%90%E5%95%86%E4%B8%9A/10778454)中 RSA 被广泛使用。RSA是1977年由[罗纳德·李维斯特](https://baike.baidu.com/item/%E7%BD%97%E7%BA%B3%E5%BE%B7%C2%B7%E6%9D%8E%E7%BB%B4%E6%96%AF%E7%89%B9/700199)（Ron Rivest）、[阿迪·萨莫尔](https://baike.baidu.com/item/%E9%98%BF%E8%BF%AA%C2%B7%E8%90%A8%E8%8E%AB%E5%B0%94)（Adi Shamir）和[伦纳德·阿德曼](https://baike.baidu.com/item/%E4%BC%A6%E7%BA%B3%E5%BE%B7%C2%B7%E9%98%BF%E5%BE%B7%E6%9B%BC/12575612)（Leonard Adleman）一起提出的。当时他们三人都在[麻省理工学院](https://baike.baidu.com/item/%E9%BA%BB%E7%9C%81%E7%90%86%E5%B7%A5%E5%AD%A6%E9%99%A2/117999)工作。RSA 就是他们三人姓氏开头字母拼在一起组成的。

下图表示非对称加解密过程，通过下图试着回忆一下之前所学的内容。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547118035000.png-wm)

（参考[RSA算法_百度百科](https://baike.baidu.com/item/RSA%E7%AE%97%E6%B3%95?fromtitle=RSA&fromid=210678)）

## 实验1：使用命令进行非对称加解密

假设我们有这样一个场景，还是想传送文件给实小楼，不同的是，这次我们使用非对称加密，用实小楼的私钥生成一个公钥，然后用这个公钥加密要传送的文件，将密文传送给实小楼过后，实小楼可以使用私钥将密文解密。（当然，也可以用私钥加密，用公钥解密）

### 实验前准备

因为此实验会产生许多文件，为了避免和之前产生的文件混淆，我们先新建一个目录 `rsa-demo`，然后在新建的目录下操作。

```
mkdir rsa-demo
cd rsa-demo
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547117029244.png-wm)

然后新建一个 message.txt ，这是我们需要加密的文件，操作如下：

```
echo "i love shiyanlou" > message.txt
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547117245775.png-wm)

### 生成私钥

接下来就是使用 `openssl` 生成一个 2048 位的私钥 `private.pem`：

```
openssl genrsa -out private.pem 2048
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547117628925.png-wm)

来查看一下私钥的内容：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547118294599.png-wm)

### 根据私钥生成公钥

根据私钥生成公钥 `public.pem` ：

```
openssl rsa -in private.pem -pubout -out public.pem
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547118216161.png-wm)

来查看一下公钥的内容：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547118353100.png-wm)

### 方式一：公钥加密，私钥解密

#### 用公钥加密

用公钥对 message.txt 加密，输出密文到 enc.txt 文件中：

```
openssl rsautl -encrypt -in message.txt -pubin -inkey public.pem -out enc.txt
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547118970101.png-wm)

#### 用私钥解密

用私钥对 enc.txt 进行解密，输出到 dec.txt 文件中：

```
openssl rsautl -decrypt -in enc.txt -inkey private.pem -out dec.txt
```

![image-20190110192001129](/Users/kinglion/Library/Application Support/typora-user-images/image-20190110192001129.png)

用一幅图对这个过程进行总结：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547121894661.png-wm)

### 方式二：私钥加密，公钥解密

#### 用私钥签名

用私钥 private.pem 对 message.txt 进行签名，输出到 sign.bin 中：

```
openssl rsautl -sign -in message.txt -inkey private.pem -out sign.bin
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547120433430.png-wm)

#### 用公钥验证

用公钥 public.pem 对签名 sign.bin 进行验证，输出到 dec.txt 中：

```
openssl rsautl -verify -in sign.bin -pubin -inkey public.pem -out dec.txt 
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547120647230.png-wm)

## 实验2: 通过 ssh 连接理解非对称加解密

在使用 ssh 远程连接到某台电脑时也会用到非对称加解密算法。实验环境只有一台机器，所以我们用本机 ssh 连接本机来模拟这个过程。

首先在本机采用 rsa 算法生成一对公钥和私钥：

```
# 回到家目录
cd ~ 

ssh-keygen -t rsa

# 命令执行会有一些提问，直接按 enter 键即可
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547122618299.png-wm)

使用 `ls ~/.ssh` 命令，你会发现里面有一个私钥文件和公钥文件。

接下来在远程机器的 `~/.ssh` 目录下新建一个 `authorized_keys` 文件，添加 `id_rsa.pub`  （公钥信息）的内容到里面，我们这里是连接本地，所以直接在本地的 `~/.ssh` 目录下新建即可：

```
cd ~/.ssh

cat id_rsa.pub > authorized_keys
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547123472852.png-wm)

接下来使用 `ssh` 命令连接即可：

```
ssh shiyanlou@localhost
```

> shiyanlou 是连接的远程主机的用户名，localhost 是连接的远程主机地址。

`authorized_key` 文件里保存的公钥信息会来验证 `id_rsa` 私钥，如果验证通过的话，就能连接上。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547123702865.png-wm)

