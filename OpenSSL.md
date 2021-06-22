https://www.cnblogs.com/xiaocen/p/3712945.html
https://blog.csdn.net/tzdjzs/article/details/28113893
https://blog.csdn.net/zzxian/article/details/7739673

前言
   openSSL是一款功能强大的加密工具、我们当中许多人已经在使用openSSL、用于创建RSA私钥或证书签名请求、不过、你可知道可以使用openSSL来测试计算机速度？或者还可以用它来对文件或消息进行加密。

正文

   openssl是一个开源程序的套件、这个套件有三个部分组成、一是libcryto、这是一个具有通用功能的加密库、里面实现了众多的加密库、二是 libssl、这个是实现ssl机制的、他是用于实现TLS/SSL的功能、三是openssl、是个多功能命令行工具、他可以实现加密解密、甚至还可以 当CA来用、可以让你创建证书、吊销证书、这里我们用openssl enc对一个文件进行加密看看：
   # openssl enc -des3 -a -salt -in /etc/fstab -out /tmp/fstab.cipher    加密
   # cat /tmp/fstab.cipher
   # openssl enc -d -des3 -a -salt -in /tmp/fstab.cipher -out /path/to/fstab.cipher   解密
wKioL1Mo8LjRHGiLAASNpydrhdg004.jpg


数字证书：
   证书格式通常是x509的数字证书的格式、还有pkcs等其他的。
    对于x509这种证书内容当中都包含哪些呢：
    1、公钥和也就是有效期限。
    2、持有者的个人合法身份信息、这个信息有可能是一个公司、也可能是个人、也可以是主机名。
    3、证书的使用方式、比如用来进行主机之间的认证等。
    4、CA(证书颁发机构)的信息
    5、CA的数字签名、CA的证是自签证书    

公钥加密、也叫非对称加密
   公钥加密最大的特性就是密钥成对的、公钥称为public key(pkey)、私钥称为secret key(skey)、一般而言、公钥用来加密、私钥用来解密、如果要实现电子签名那就是私钥来用加密、公钥用来解密、而公钥是可以给任何人的、私钥就得自 己保存；公钥加密一般不会加来对数据加密、因为他的加密速度很慢、比对称加密慢3个数量级(一个数量级是10倍、3个就是1000倍)、所以公钥加密通常 用于密钥交换(IKE)和身份认证的。
   他的常用算法有：RSA和EIGamal、目前RSA是比较广泛的加密算法、而DSA(Digital Signature Algorithm)只能加来做签名、而无法加于加密的算法
   他的工具通常用：gpg、openssl rsautl
   wKioL1Mo_M3zqIQTAAMed6kdJFk241.jpg


单向加密、也叫hash算法：(One-Way加密)
   用不生成数据指纹的、也叫数据摘要算法、输出是定长的、MD5是128位定长输出、SHA1定长输出160位、他的特性是不会出现碰撞的、每位数据只要 有一位不一样就会产生巨大的变化、我们称这种为雪崩效应、常用的算法MD5、SHA1、SHA512、常用工具有sha2sum、md5sum、 cksum、openssl dgst。
   # sha1sum fstab
   # openssl dgst -sha1 fstab
wKioL1Mo8qGRYU8qAACi9bv1dNk842.jpg


信息摘要码：
   MAC(Message Authentication Code)：通常应用于实现在网络通信中保证所传输的数据完整性、他的基本方式就是基于MAC将要通信的数据使用单向加密的算法获取定长输出、而后将这定 长输出安全可靠的送达到接收方的一种机制、简单来讲我们客气端发送数据给服务器时、客户端会计算这段数据的特征码、并而将这段特征码发送给服务器端、但是 这种特征码不能简单的这样传送过去、他要基于MAC、调用单向加密计算这段特征码、而后将加密的结果发送给服务器端、保证特征码不会被人修改、这是单向加 密的一种实现、一种延伸应用；
   他的常用算法有：CBC-MAC、HMAC    
   对于openssl来讲、如果你是客户端、他可以帮我们生成密钥对、帮我们生成证书申请、如果是发证方、他可以帮发证方自签证书、还可以签署证书、还可以生成吊销列表、当然大范围内全球内使用openCA。
    那接下来我们就用openssl完成证书生成、签署、颁发以及吊销等功能：

wKiom1Mo8ebR3A4yAAgALJs1nQ4330.jpg

实现步骤：
   首先自己得有一个证书、那就先自签一个证书、用openssl实现私有CA、CA的工作目录都是在/etc/pki/CA下、而CA的配置文件在/etc/pki/tls/openssl.cnf这个文件中。

   生成CA私钥、这里要注意、公钥是按某种格式从私钥中提取出来的、公钥和私钥是成对的、生成私钥也就有了公钥：
   wKiom1Mo8wvBeJ8aAAX_UqANBg4206.jpg

 # (umask 077; openssl genrsa -out private/cakey.pem 2048) 


   在当前shell中用()执行命令表示括号中的命令要在子shell中执行，2048表示密钥的长度、-out后面表示生成密钥文件保存的路径,生也的文件权限是666、而这个文件不能被别人访问、所在666-077就得到权限600：


   查看公钥或提取公钥、这个并不是必要步骤：
   # openssl rsa -in private/cakey.pem -pubout -text

wKioL1Mo87KwcGDEAAFzDOb-lMY884.jpg


   生成自签证书、用openssl中req这个命令、叫证书请求：

 wKiom1Mo9AHwimaXAASKJgtp0-k242.jpg

# openssl req -new -x509 -key private/cakey.pem -out cacert.pem -days 3650   


   在CA的目录下创建两个文件：
   # touch index.txt serial
wKioL1Mo9DaCGJ7sAAD4Zm6-oOE736.jpg


   OK、CA的证书有了、那接下来就是给客户签署证书了;我这里换另一台主机来做客户、向CA了起签署申请、如果要给服务器使用、那一定要跟你的服务器名保持一致、我们这里是以web服务器使用的、所以生成的私钥也要放在服务器的目录下、我这里以httpd为例：
   生成密钥对、我们专门分建一个目录来存放：
   # mkdir /etc/httpd/ssl
   # (umask 077; openssl genrsa -out httpd.key 1024)

wKiom1Mo9IOD_HMtAAEzY0ZsikU118.jpg


   客户端生成证书签署请求：
   # openssl req -new -key httpd.key -out httpd.csr
wKioL1Mo9ISyuHPbAAMZWjTR_3w493.jpg

   再把httpd.csr发给远程主机的CA签
   # scp httpd.csr 172.16.251.171:/tmp/
wKioL1Mo9KvTnwzkAAGD6uCXOk0448.jpg


   再切换到远程主机的/tmp看一下有没有一个叫httpd.csr的文件：
wKiom1Mo9PCSlFofAADwVuludfE414.jpg


   于是我们的CA检查信息完后就可以签署了：
   # openssl ca -in /tmp/httpd.csr -out /tmp/httpd.crt -days 3655
wKiom1Mo9RaAGH6JAAODPaQCazo758.jpg

   再把签署好的证书发送回去给客户端的主机：
   # scp httpd.crt 172.16.251.127:/etc/httpd/ssl/

wKioL1Mo9RSyx2W-AAGh15Klvy8149.jpg

   发送给客户端主机了我们就可以去查看一下了：
   # ls -l /etc/httpd/ssl
wKioL1Mo9TniSBy2AACX06DQojs497.jpg

   于是我们的客户端主机就可以配置使用CA签署的证书了。
   如果说证书过期了怎么吊销呢：(要在CA主机上吊销)
   # openssl ca -revoke httpd.crt

I have to create CA certificate first.

openssl genrsa -out rootCA.key 2048
openssl req -x509 -new -nodes -key rootCA.key -days 1024 -out rootCA.pem

Copy it on user's workstation.

Then create key file using following command.

openssl genrsa -out device.key 2048

and csr file using that key file using command:

openssl req -new -key device.key -out device.csr

Once that’s done, sign the CSR, which requires the CA root key.

openssl x509 -req -in device.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out device.crt -days 500