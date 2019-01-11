# OpenSSL 数字证书

通过使用 OpenSSL 学习颁发数字证书的完整流程。

## 数字证书简介

为了便于理解，先打个比方。我们都知道坐车要车票，驾照上面有本人照片，姓名，准驾车辆等信息，并由相关部门盖章，看到驾照，就证明此人具有驾驶车辆的资格。

数字证书其实和驾照相似，里面记有姓名等个人信息，以及属于此人的公钥，并由认证机构（CA）进行数字签名。只要看到这个证书，就可以知道认证机构认定该公钥的确属于此人。

还记得之前的实例吗，实小楼将公钥公布给我，而我如何确认收到的公钥是实小楼的，没有经过篡改？实小楼会先在认证机构注册自己的公钥，认证机构会用自己的私钥对实小楼的公钥施加数字签名并生成证书，我得到证书后，用认证机构的公钥验证数字签名来确认收到的公钥是是否属于实小楼。

## 实验：颁发数字证书

下面通过动手操作来学习颁发数字证书的完整流程。首先要明白两个问题：

1. 由谁颁发？

即为颁发者。由认证机构（CA）进行颁发。在实验中我们将自签名证书，构建自己的根 CA。所谓自签名意思即为证书的颁发者和使用者都是自己。

2. 颁发给谁？

即为使用者。使用者会向 CA 申请证书。因为我们只有一台机器，所以直接用本机来模拟使用者。

为了便于实验，不易混淆，我们先创建两个目录 ： `ca` 和 `syl` 来模拟 CA，syl 申请者。

```
cd Code

mkdir ca
mkdir syl
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547198282854.png-wm)

做好准备工作过后，就要开始实验了，实验主要有以下几个步骤：

- 成为数字证书认证机构，构建自己的根 CA
- 申请者生成私有密钥
- 申请者将包含公共密钥的证书请求文件发送给 CA
- CA 对发送过来的请求文件进行签名，并颁发签名过后的证书

### 构建自己的根 CA

想要从商业 CA 获取数字证书需要向其支付一定的金钱，不过我们不用那么破费，可以自己成为 root CA，为自己发布证书。

在本实验中，我们将成为 root CA，并为该 CA 生成证书。不像其他 CA 需要被另外的 CA 认证， root CA 的证书是自己为自己认证的，一般 Root CA 的证书都已事先加载在大多数操作系统，浏览器或者依赖 PKI 的软件之中了。Root CA 的证书是被无条件信任的。

首先，进入到 ca 目录：

```
cd ca
```

然后构建自己的根 ca：

```
openssl req -x509 -newkey rsa:2048 -keyout ca.key -out ca.crt

# 会提示设置密码，我们可以设置为 shiyanlou，记住这个密码，ca 在颁发证书的时候需要用到这个密码。其他信息可以按如下截图填写。
```

> -keyout：指定私钥的文件名
>
> -out：指定证书的文件名

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547199050426.png-wm)

使用 `ls` 可以看到下面产生了两个文件。分别是 `ca.crt` 和 `ca.key` 。`ca.key` 是生成的私钥，可以使用 cat 命令来查看里面的内容：

```
cat ca.key
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547199274051.png-wm)

`ca.crt` 是自签名的证书文件。

使用 cat 和 file 命令查看一下。（file 命令可以分析出文件的格式以及用途）

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547199381048.png-wm)

另外，我们可以使用如下命令查看到该证书的颁发者和使用者：

```
openssl x509 -in ca.crt -subject -issuer -noout
```

> -subject：查看证书的主题，也就是颁发给谁
>
> -issuer：查看证书的颁发者
>
> -noout：不输出证书内容

输出结果：

```
subject= /C=CN/ST=Sichuan/L=Chengdu/O=shiyanlou/OU=it/CN=kinglion/emailAddress=shiyanlou@simplecloud.cn
issuer= /C=CN/ST=Sichuan/L=Chengdu/O=shiyanlou/OU=it/CN=kinglion/emailAddress=shiyanlou@simplecloud.cn
```

从输出可以看到，颁发者和使用者是同一个。

### 使用者创建私有密钥

实验中为了和 CA 区分开，我们创建了 syl 目录，来保存使用者操作过程中生成的文件。所以首先进入 syl 目录：

```
cd ../syl
```

创建私有密钥：

```
openssl genrsa -out syl.key 2048
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547200324074.png-wm)

> 另外，我们可以使用的  `-des3` ， `-aes128` 等选项用不同的加密算法对这个私钥进行加密，以后使用该私钥时需要提供密码。可以使用 `openssl genrsa --help` 或者 `man genrsa` 查看命令帮助。为了方便，我们这里就不加密了。

使用 cat 看一下 syl.key 的内容：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547201248408.png-wm)

### 申请者生成证书请求文件

申请者需要生成一个包含公共密钥的证书请求文件（CSR），这个文件后续是要传给 CA 签名的。

```
openssl req -new -key syl.key -out req.csr
```

这条命令将根据刚才的私钥生成证书请求文件 req.csr。会要求填写一些信息，为了便于实验，可以根据我的截图进行填写。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547201531623.png-wm)

实际上。创建私钥以及生成证书请求可以直接使用一条命令：

**上面我们已经生成了需要的文件，这里只做介绍，不用操作。当然你也可以删掉之前生成的两个文件，重新用下面的命令试一试**

```
openssl req -newkey rsa:2048 -keyout syl.key -out req.csr -nodes
```

> 这条命令会同时生成私钥以及证书请求文件。-nodes 选项意思是不对 syl.key 加密，默认是要加密的，会提示输入密码。

我们可以使用命令来看到 req.csr 中的一些信息：

```
openssl req -in req.csr -text  
```

> 当然，还有其他很多选项可用，可以使用 `openssl req --help` 查看帮助。您可以试试加上 `-verify` ，`-noout` 选项，看看输出结果有什么不同。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547203343203.png-wm)

可以使用 `ls` 命令看下当前有什么文件。这个 req.csr 文件需要传给 CA 进行签名。为了模拟，我们直接将 req.csr 拷贝到 ca 目录下。

```
# 拷贝到 ca 目录
cp req.csr ../ca

# 进入 ca 目录
cd ../ca

# 查看 ca 目录下的文件
ls
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547202240701.png)

### CA 签署颁发证书

CA 收到申请者的证书请求文件过后，使用自己的私钥对请求文件进行签名并颁布数字证书。

```
openssl x509 -req -in req.csr -out syl.crt -CA ca.crt -CAkey ca.key -CAcreateserial

# 执行这条命令会提示输入密码，密码即是在自建根 CA 时设置的密码：shiyanlou
```

> `ca.key` 是 CA 的私钥。
>
> `ca.crt` 是自签名时生成的证书。
>
> `syl.crt` 是签名过后得到的证书。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547203012441.png-wm)

查看 syl.crt 证书的颁发者和申请者：

```
openssl x509 -in syl.crt -subject -issuer -noout 
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1547203222105.png-wm)

至此，颁发数字证书的整个流程就结束了，可以试着看一看相关目录下的文件，理一理整个过程。