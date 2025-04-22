# HTTPS加密原理

## 一、HTTPS简介

HTTPS协议（Hyper Text Transfer Protocol Secure），是HTTP的加强安全版本。HTTPS是基于HTTP的，并额外使用SSL/TLS协议用作加密和安全认证

TLS，传输层安全性协议(Transport Layer Security)，前身即SSL，安全套接层(Secure Sockets Layer)。用于提供通信的保密性和数据完整性

**与HTTP的区别**

- HTTP明文传输，不能保证传输安全性；HTTPS通过TLS握手保证传输安全性

- HTTP默认端口号为80；HTTPS默认端口号为443

## 二、对称加密与非对称加密

**对称加密**：接收方、发送方使用同一个秘钥对信息进行加密解密，效率高但是不安全

**非对称加密**：接收方、发送方各有一对秘钥（私钥、公钥），通信之前双方会先把自己的公钥发给对方。数据发送前先用私钥加密，接收方再用对应的公钥解密，安全但是效率低

**对称加密+非对称加密**：服务端拥有一对秘钥，建立连接时，将公钥发送给客户端，客户端生成一个秘钥并用公钥加密，之后传送给服务端，以后就可以用这个秘钥进行对称加密。但是**服务端发送公钥时有可能被第三方劫持，然后第三方将自己的公钥替换掉服务端的公钥，发送给客户端**

![img](https://raw.githubusercontent.com/KKKLxxx/img-host/master/202111302010535.png)

**解决方案**：服务端使用HTTPS前需要获得由CA机构颁布的证书，证书由明文和签名组成，CA机构拥有服务端的密钥对。明文即服务端的信息和公钥，签名即由哈希算法和私钥加密处理过的明文。客户端验证签名时，用哈希算法对明文处理的结果与用明文中的公钥对签名解密的结果进行对比，如果相同，则说明没有被篡改

## 三、TLS握手过程

### 1、具体过程

#### 1、客户端发送Client Hello

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20220904123949518.png" alt="image-20220904123949518" style="zoom: 50%;" />

1、支持的TLS版本

2、随机数1

3、加密套件，即不同加密算法的组合

#### 2、服务端发送Server Hello

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20220904124038502.png" alt="image-20220904124038502" style="zoom:50%;" />

1、服务端确认支持TLS版本

2、随机数2

3、选择的加密套件

#### 3、服务端发送证书

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20220904124207515.png" alt="image-20220904124207515" style="zoom:50%;" />

#### 4、服务端发送公钥

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20220904124236893.png" alt="image-20220904124236893" style="zoom:50%;" />

#### 5、服务端发送完毕

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20220904124317916.png" alt="image-20220904124317916" style="zoom:50%;" />

#### 6、客户端生成并发送会话密钥

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20220904124406107.png" alt="image-20220904124406107" style="zoom:50%;" />

客户端通过随机数1，随机数2和预主密钥(随机数3)生成一个会话密钥，用公钥加密后并发送给服务端

#### 7、服务端通过私钥解密得到会话密钥

#### 8、通过会话密钥进行对称加密传输

### 2、总结

<img src="https://raw.githubusercontent.com/KKKLxxx/img-host/master/image-20220904124857959.png" alt="image-20220904124857959" style="zoom: 50%;" />

1、客户端发送Client Hello，包含了TLS版本，随机数1，可选的加密套件（加密算法的组合）

2、服务端发送Server Hello，包含服务端确认支持的TLS版本，随机数2，服务端选择的加密套件

3、服务端发送证书和公钥，以及一个发送完毕的消息

4、客户端生成预主密钥(随机数3)，并通过公钥加密后发送

5、服务端通过私钥解密，得到预主密钥

6、服务端与客户端通过相同的加密算法，对随机数1，随机数2和预主密钥进行加密，得到相同的会话密钥

7、最后通过这个会话密钥进行对称加密传输

## 四、问题

**Q1：如何防止第三方篡改证书**

由于第三方没有私钥，篡改明文后无法篡改签名，客户端可以验证出来

**Q2：如何防止整个证书被调换**

客户端可以对比自己请求的网站与证书中的网站是否相同

**Q3：如何防止请求被中间人拦截**

假设有中间人拦截请求，那么就需要客户端与中间人建立连接

建立连接需要验证证书。假设中间人得到服务端的证书并发送给客户端，但因为没有私钥，就无法获取最后的会话密钥。所以除非服务端私钥泄露，那么TLS就是可以保证传输的安全性的

**Q4：为什么要用到哈希算法**

因为证书信息可能较长，经哈希算法处理后可以得到一个定长的字段，提升处理效率
