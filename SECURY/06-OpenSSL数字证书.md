# OpenSSL 数字证书

通过使用 OpenSSL 学习颁发数字证书的完整流程，学习数字证书的基本原理和应用方法。

## 数字证书简介

为了便于理解，先打个比方。我们都知道坐车要车票，驾照上面有本人照片，姓名，准驾车辆等信息，并由相关部门盖章，看到驾照，就证明此人具有驾驶车辆的资格。

数字证书其实和驾照相似，里面记有姓名等个人信息，以及属于此人的公钥，并由认证机构（CA）进行数字签名。只要看到这个证书，就可以知道认证机构认定该公钥的确属于此人。

还记得之前的实例吗，实小楼将公钥公布给我，而我如何确认收到的公钥是实小楼的，没有经过篡改？实小楼会先在认证机构注册自己的公钥，认证机构会用自己的私钥对实小楼的公钥施加数字签名并生成证书，我得到证书后，用认证机构的公钥验证数字签名来确认收到的公钥是是否属于实小楼。





- 生成密钥对
- 私有密钥的保护密码
- 自签名证书与签署颁发证书
- 证书查看与导出

生成密钥对：

方法一：创建私有密钥后，再生成证书请求文件

```
# openssl genrsa -out key.pem 2048
# openssl req -new -key key.pem -out req.pem
```

> genrsa : 生成 rsa 私钥。可以使用 `openssl genrsa --help` 或者 `man genrsa` 查看命令帮助。
>
> 2048: 以位为单位指定生成的密钥大小
>
> req：PKCS＃10 X.509证书签名请求（CSR）管理。（PKCS#10 X.509 Certificate Signing Request (CSR) Management.）
>
> PKCS：由美国 RSA 公司及其合作伙伴一起制定的一个公共密钥加密标准。

方法二：一次性创建私有密钥和证书请求文件

```
openssl req -newkey rsa:2048 -keyout key.pem -out req.csr
```

> -newkey：创建新证书请求和新私钥
>
> -keyout：指定新创建的私钥的文件名
>
> -out：指定输出的请求文件的文件名



方法一操作：

```
openssl genrsa -out key.pem 2048
file key.pem  # 分析出文件的格式以及用途
cat key.pem
```

生成 PKCS 证书签名请求文件：

```
openssl req 
ls
file req.csr
cat req.csr
openssl req -in req.csr -text # 查看文件里的信息
openssl req -in req.csr -text -verify # 验证是否损坏
openssl req -in req.csr -text -verify -noout # 只看摘要信息
```

方法二操作：

```
openssl req -newkey rsa:2048 -keyout key.pem -out req.csr
```

> 会提示设置密码，对 key.pem 保护，以后使用 key.pem 得提供这个密码才可以使用。



私有密钥保护：

在PKI体系中，公共密钥可以给任何人，私有密钥就必须由自己妥善的进行保管的。

私有密钥文件本身就支持密码保护。采用这种形式保护之后，当你在使用私有密钥时，就必须提供密码。在有些场景中，比如需要实现无人执守的自动化操作，那么可以在安全有足够保证的前提下，使用不带密码保护的私有密钥文件。

下面，我们来学习如何使用OpenSSL命令来管理私有密钥的保护密码。

方法一：

```
openssl rsa -des3 -out key.pem 2048
```

> -des3：使用 3DES 算法加密，是 DES 算法的增强，比 DES 更安全

方法二：

```
openssl req -newkey rsa:2048 -keyout key.pem -out req.csr -nodes
```



自签名证书与签署颁布证书：

在使用OpenSSL的req子命令的x509的选项，可以创建自签名的证书。自签名证书主要有两个用途：测试和构建根CA。

创建自签名的根证书并给申请者颁布证书。自签名的意思就是自己给自己颁发证书，证书的颁发者和拥有者是同一个人。

作用：测试用，开发人员现在有一个测试用的 web 服务器，上面要安装一个证书来实现 https 协议，最简单的方法就是给服务器上安装一个自签名的证书。

构建自己的根 CA。

```
openssl req -x509 -newkey rsa:2048 -keyout ca.key -out ca.crt
```

> -keyout：指定私钥的文件名
>
> -out：指定证书的文件名

查看证书文件：

```
openssl x509 -in ca.crt -subject -issuer -noout
```

> -subject：查看证书的主题，也就是颁发给谁
>
> -issuer：查看证书的颁发者

使用者和颁发者的信息是一样的。



签署颁发证书操作：

服务器上：

1. 创建自签名的根证书

```
openssl req -x509 -newkey rsa:2048 -keyout ca.key -out ca.crt
```

会让设置一个密码，这个密码要记住，以后颁发证书需要用到这个密码。

> ca.key ：加密了的私有密钥
>
> ca.crt: 自签名的证书文件

申请者上：

- 生成私有密钥
- 包含公共密钥的证书请求文件

```
openssl req -newkey rsa:2048 -keyout key.pem -out req.csr -nodes
```

> req.csr：证书请求文件

申请者将证书请求文件（req.csr)传给 ca。

```
cp req.csr ../ca
```

服务器上：

对证书请求文件 req.csr 进行签名。

```
openssl x509 -req -in req.csr -out syl.crt -CA ca.crt -CAkey ca.key -CAcreateserial
```

> 输入的密码是之前设置的密码。
>
> 第一次签名的时候需要 -CAcreateserial 创建一个序列号文件 （ca.srl)，以后就不用了。
>
> 可以利用 -day 选项设置有效天数。

生成的 syl.crt 就是颁发给 syl 的证书。



