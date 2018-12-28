title: HTTPS理论基础
comment: true
categories:
  - 计算机相关
date: 2016-06-09 15:51:32
tags: 
  - 安全
permalink: https-basic 

---

在网上看了好多https的相关文章，但一般都是千篇一律，越看越糊涂
今天在网上看了一篇文章，觉得还不错，讲的还比较清晰，看完以后对于https有了相对深入的理解
[HTTPS理论基础及其在Android中的最佳实践](http://blog.csdn.net/iispring/article/details/51615631)
以下是我读后的一些理解
<!-- more -->

![https](/image/https.jpg)

# 对称加密与非对称加密

## 对称加密

对称加密又叫做私钥加密，即信息的发送方和接收方使用同一个密钥去加密和解密数据。对称加密的特点是算法公开、加密和解密速度快，适合于对大数据量进行加密，常见的对称加密算法有DES、3DES、TDEA、Blowfish、RC5和IDEA。
其加密过程如下：明文 + 加密算法 + 私钥 => 密文 
解密过程如下：密文 + 解密算法 + 私钥 => 明文 
对称加密中用到的密钥叫做私钥，私钥表示个人私有的密钥，即该密钥不能被泄露。 
其加密过程中的私钥与解密过程中用到的私钥是同一个密钥，这也是称加密之所以称之为“对称”的原因。由于对称加密的算法是公开的，所以一旦私钥被泄露，那么密文就很容易被破解，所以对称加密的缺点是密钥安全管理困难 

以下是参考网上的java实现DES加密与解密的demo
[Java DES 加密 解密 示例](http://blog.csdn.net/techzero/article/details/17282637)
``` java
package desTest;
import java.security.SecureRandom;

import javax.crypto.Cipher;
import javax.crypto.SecretKey;
import javax.crypto.SecretKeyFactory;
import javax.crypto.spec.DESKeySpec;

public class DESTest {

    /**
     * @param args
     */
    public static void main(String[] args) {
        String content = "杨彦星";
        String password = "123456789";//不明白原文为啥说密码必须是8的位数。。。
        System.out.println("密　钥：" + password);
        System.out.println("加密前：" + content);
        byte[] result = encrypt(content, password);
        System.out.println("加密后：" + new String(result));
        String decryResult = decrypt(result, password);
        System.out.println("解密后：" + decryResult);
    }

    /**
     * 加密
     * 
     * @param content
     *            待加密内容
     * @param key
     *            加密的密钥
     * @return
     */
    public static byte[] encrypt(String content, String key) {
        try {
            SecureRandom random = new SecureRandom();
            DESKeySpec desKey = new DESKeySpec(key.getBytes());
            SecretKeyFactory keyFactory = SecretKeyFactory.getInstance("DES");
            SecretKey securekey = keyFactory.generateSecret(desKey);
            Cipher cipher = Cipher.getInstance("DES");
            cipher.init(Cipher.ENCRYPT_MODE, securekey, random);
            byte[] result = cipher.doFinal(content.getBytes());
            return result;
        } catch (Throwable e) {
            e.printStackTrace();
        }
        return null;
    }

    /**
     * 解密
     * 
     * @param content
     *            待解密内容
     * @param key
     *            解密的密钥
     * @return
     */
    public static String decrypt(byte[] content, String key) {
        try {
            SecureRandom random = new SecureRandom();
            DESKeySpec desKey = new DESKeySpec(key.getBytes());
            SecretKeyFactory keyFactory = SecretKeyFactory.getInstance("DES");
            SecretKey securekey = keyFactory.generateSecret(desKey);
            Cipher cipher = Cipher.getInstance("DES");
            cipher.init(Cipher.DECRYPT_MODE, securekey, random);
            byte[] result = cipher.doFinal(content);
            return new String(result);
        } catch (Throwable e) {
            e.printStackTrace();
        }
        return null;
    }
}

```

可见其实这里的私钥其实就是一个字符串，如果知道了这个私钥，同时也知道了加密算法是DES，那么就可以轻松的解密加密数据了。

## 非对称加密

非对称加密也叫做公钥加密。非对称加密与对称加密相比，其安全性更好。对称加密的通信双方使用相同的密钥，如果一方的密钥遭泄露，那么整个通信就会被破解。而非对称加密使用一对密钥，即公钥和私钥，且二者成对出现。私钥被自己保存，不能对外泄露。公钥指的是公共的密钥，任何人都可以获得该密钥。用公钥或私钥中的任何一个进行加密，用另一个进行解密。 
被公钥加密过的密文只能被私钥解密，过程如下： 
明文 + 加密算法 + 公钥 => 密文， 密文 + 解密算法 + 私钥 => 明文 
被私钥加密过的密文只能被公钥解密，过程如下： 
明文 + 加密算法 + 私钥 => 密文， 密文 + 解密算法 + 公钥 => 明文 
由于加密和解密使用了两个不同的密钥，这就是非对称加密“非对称”的原因。 
非对称加密的缺点是加密和解密花费时间长、速度慢，只适合对少量数据进行加密。 
在非对称加密中使用的主要算法有：RSA、Elgamal、Rabin、D-H、ECC（椭圆曲线加密算法）等。

# HTTPS通信过程

HTTPS协议 = HTTP协议 + SSL/TLS协议，在HTTPS数据传输的过程中，需要用SSL/TLS对数据进行加密和解密，需要用HTTP对加密后的数据进行传输，由此可以看出HTTPS是由HTTP和SSL/TLS一起合作完成的。
HTTPS为了兼顾安全与效率，同时使用了对称加密和非对称加密。数据是被对称加密传输的，对称加密过程需要客户端的一个密钥，为了确保能把该密钥安全传输到服务器端，采用非对称加密对该密钥进行加密传输，总的来说，对数据进行对称加密，对称加密所要使用的密钥通过非对称加密传输。

https传输过程，为了效率数据采用对称加密，但是对称加密所采用的私钥为了安全采用非对称加密
![https传输过程](http://img.blog.csdn.net/20160608220337692)

整个过程中会涉及到三个密钥
* 服务器端的公钥与私钥，用来进行非对称加密
* 客户端生成的随机密钥，用来对数据进行对称加密

1. 客户端发起https请求，用户输入一个https网址以后，访问服务器的443端口
2. 服务器端有一个密钥对，即公钥和私钥，是用来进行非对称加密使用的，服务器端保存着私钥，不能将其泄露，公钥可以发送给任何人。
3. 传送证书，也就是公钥，服务端将公钥传给客户端。
4. 客户端解析证书，客户端收到服务器发过来的公钥以后先要对其有效性进行校验，如果公钥有问题则无法进行https传输，这个公钥也就是服务器发过来的数字证书。如果没有问题，则会生成一个随机值，这个随机值就是对于对称加密的密钥。然后用服务端发过来的公钥对这个随机值(也就是客户端私钥)进行非对称加密，至此，HTTPS中的第一次HTTP请求结束。
5. 客户端会发起HTTPS中的第二个HTTP请求，将加密之后的客户端密钥发送给服务器。
6. 服务器接收到客户端发来的密文之后，会用自己的私钥对其进行非对称解密，解密之后的明文就是客户端密钥，然后用客户端密钥对数据进行对称加密，这样数据就变成了密文。
7. 然后服务器将加密后的密文发送给客户端。
8. 客户端收到服务器发送来的密文，用客户端密钥对其进行对称解密，得到服务器发送的数据。这样HTTPS中的第二个HTTP请求结束，整个HTTPS传输完成。


# 数字证书

在整个https的传输过程中，当服务器发送自已的公钥时，会有黑客进行篡改，那个客户端怎么信赖这个公钥呢？这里就用到了了数字证书。
数字证书可以很方便的生成，黑客也可以很方便的生成，但是一个客户端凭什么就相信你的数字证书呢？这里就有数字认证中心CA 专门对公钥进行认证，全球知名的CA也就100多个，客户端默认只信任这100多个CA颁发的证书，客户端也可以自已添加信任的证书。
但是很有可能你网站的数字证书不是这100家CA颁发的，而是其下属的认证中心，好比说，我相信了A,A又相信了B，B又相信了C，那么我也就相信了C。那CA怎么对公钥做担保认证呢？CA本身也有一对公钥和私钥，CA会用CA自己的私钥对要进行认证的公钥进行非对称加密，此处待认证的公钥就相当于是明文，加密完之后，得到的密文再加上证书的过期时间、颁发给、颁发者等信息，就组成了数字证书。

当客户端接收到服务器的数字证书的时候，会进行如下验证：

1. 首先客户端会用设备中内置的CA的公钥尝试解密数字证书，如果所有内置的CA的公钥都无法解密该数字证书，说明该数字证书不是由一个全球知名的CA签发的，这样客户端就无法信任该服务器的数字证书。

2. 如果有一个CA的公钥能够成功解密该数字证书，说明该数字证书就是由该CA的私钥签发的，因为被私钥加密的密文只能被与其成对的公钥解密。

3. 除此之外，还需要检查客户端当前访问的服务器的域名是与数字证书中提供的“颁发给”这一项吻合，还要检查数字证书是否过期等。

# 12306网站的数字证书问题

当用户访问 https://kyfw.12306.cn/otn/regist/init 如果没有导入其根证书的话那个浏览器上会显示一个不安全的提示
![12306不安全](/image/12306notsafe.png)

为什么会这样呢？
看一下12306的证书信息
![](http://img.blog.csdn.net/20160608223638227)

可以看到，该12306.cn的证书是由SRCA这个机构签发的，也就是说SRCA是证书链上的根CA。
但是SRCA是啥呢？没听过啊！因为其不在默认的100多个CA里，所以这里安全校验不通过，浏览器认为它是不安全的
解决办法12306的网站上也说明了，要导入它的根证书
![](/image/import12306.png)



