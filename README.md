# 写在前面
我们说搭建shadowsocks 服务器、VPN 服务器，首先，**你得有个服务器**。国内的有阿里云，腾讯云。不过他们都太贵了，我就搭建shadowsocks 服务器就为了查点资料用用每个月还要七八十块 T_T，而且在国内的服务器做这类事情还是会有很多限制的。经过多方试用和筛选，最后选定了[vultr](https://www.vultr.com/?ref=7855182-4F)， [vultr](https://www.vultr.com/?ref=7855182-4F)每个月最低只要5美元，稳定性很好，客服回复速度也很快。最重要的是，

--

**他们支持支付宝！**
**他们支持支付宝！**
**他们支持支付宝！**

大多数国外网站都只支持 paypal ，visa 信用卡等等，能支持国内的支付宝（alipay）就真的是太方便了（不得不说支付宝国际化道路走的还是很到位的）

> 注意：提前说一下，shadowsocks 和 VPN 两个服务器不能在同一个云服务器安装，他们之间会有冲突，亲测。所以，如果你两个都要就申请两个服务器分别装吧。


# vultr 四步轻松构建服务器教程
- **第一步，点击**[www.vultr.com](https://www.vultr.com/?ref=7855182-4F)进去注册，填email和密码什么的按流程走就好了。
- **第二步，**注册成功后需要绑定付款方式，~~需要注意的是如果用PAYPAL至少充值5美元，如果绑定信用卡不用充值，但是需要2.5美金手续费验证，这个手续费一个月后会返还给你~~，**现在已经可以支持支付宝了！相信大家都有支付宝，我们直接进入下一步吧**。
- **第三步，**绑定好后点击左边的`Servers`进入服务器页面，点击最右边的大大的`+`号按钮，添加一个服务器。
![image.png](http://upload-images.jianshu.io/upload_images/2897814-00ae1541c2c125c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
，然后就送上往下选择`Server Location`一般是`Japan(Tokyo)`和`Los Angeles` 比较快,往下`Server Type`就选`CentOS7 x64`吧，我们后面的教程也是基于`centos7`的。`Server Size`默认不会是最低的，一般就选第一个最低的5美元的就行了。其他的都不用选。点击`Deploy Now`即可。
- **第四步，**在主面板找到你的服务器，点击进去，然后你就可以打开你的ssh 连接工具根据 IP、用户（root） 和 密码（打开后左下方有）连接即可。

# 第一式 搭建 shadowsocks 服务器
现在我们服务器有了，就开始搭建shadowsocks 服务器了，我不会告诉你，几乎一键式就可以搭建shadowsocks服务器。

> 新买的服务器可能没有写编辑工具，比如懂`vim`的直接安装一个vim 就好了（debian/ubuntu 用`apt-get install vim`、centos 用 `yum install vim`，其他工具类似的安装方法，名称的自行Google（隔日更新：额，好像现在你还没翻墙，那就百度吧...）

- **第一步，**依次执行下面三个命令

```
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
chmod +x shadowsocks.sh
./shadowsocks.sh 2>&1 | tee shadowsocks.log
```

一路回车+yes下来，安装完成后提示如下：

```
Congratulations, shadowsocks install completed!
Your Server IP:your_server_ip
Your Server Port:your_server_port
Your Password:your_password
Your Local IP:127.0.0.1
Your Local Port:1080
Your Encryption Method:aes-256-cfb

Welcome to visit:https://teddysun.com/342.html
Enjoy it!
```

- **第二步，**配置用户名密码，打开配置文件`/etc/shadowsocks.json`，做如下配置，端口和密码就自己填吧！

```
{
    "server":"0.0.0.0",
    "local_address":"127.0.0.1",
    "local_port":1080,
    "port_password":{
         "8989":"password0",
         "9001":"password1",
         "9002":"password2",
         "9003":"password3",
         "9004":"password4"
    },
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```

- **第三步，**下载对应平台的软件，输入ip 地址和用户名密码就行，注意加密方式要根据配置文件里面的选对。**shadowsocks 有Android,PC 版的都有，但是没有 ios 的，后面我们会将 ios 的VPN搭建方法。**

# 第二式，搭建 VPN 服务器。
VPN 可以供所有客户端使用，但是相对 shadowsocks 没有那么稳定。

**VPN 服务器搭建起来就麻烦的多了，我也是踩了非常多的坑才搭建成功的，且看我细细道来。**

## 第一步 安装 StrongSwan
两条命令：

```
yum install http://ftp.nluug.nl/pub/os/Linux/distr/fedora-epel/7/x86_64/e/epel-release-7-5.noarch.rpm
yum install strongSwan openssl
```

## 第二步 生成证书并拷贝
我们使用证书生成脚本一键生成，很方便有木有！

首先进入到`/etc/strongswan/ipsec.d`


```
cd /etc/strongswan/ipsec.d
```
然后

```
cd /etc/strongswan/ipsec.d
wget https://raw.githubusercontent.com/michael-loo/strongswan_config/for_vultr/server_key.sh
chmod a+x server_key.sh
wget https://raw.githubusercontent.com/michael-loo/strongswan_config/for_vultr/client_key.sh
chmod a+x client_key.sh
```
需要注意的是，证书生成脚本里面有个`VULTR-VPS-CENTOS`，这是组织名，在安装证书的时候会显示，其实无所谓，如果你介意可以换掉它。

生成服务器证书，SERVER_IP就是你申请的服务器的IP

```
./server_key.sh SERVER_IP
```

生成客户端证书，下面的`john`是一个名字，`john@gmail.com`是证书名，随便你写。需要注意的是，这里可能会让你输入密码，这个密码是安装时候的密码，你要记住。


```
./client_key.sh john john@gmail.com
```


然后用ftp 软件把`/etc/strongswan/ipsec.d/john.p12`和`/etc/strongswan/ipsec.d/cacerts/strongswanCert.pem`拷贝到你的电脑出来。

## 第三步 替换相关配置

- 编辑`vi /etc/strongswan/ipsec.conf`，直接替换下面的内容即可

```
config setup
    uniqueids=never
    charondebug="cfg 2, dmn 2, ike 2, net 0"

conn %default
    left=%defaultroute
    leftsubnet=0.0.0.0/0
    leftcert=vpnHostCert.pem
    right=%any
    rightsourceip=172.16.1.100/16

conn CiscoIPSec
    keyexchange=ikev1
    fragmentation=yes
    rightauth=pubkey
    rightauth2=xauth
    leftsendcert=always
    rekey=no
    auto=add

conn XauthPsk
    keyexchange=ikev1
    leftauth=psk
    rightauth=psk
    rightauth2=xauth
    auto=add

conn IpsecIKEv2
    keyexchange=ikev2
    leftauth=pubkey
    rightauth=pubkey
    leftsendcert=always
    auto=add

conn IpsecIKEv2-EAP
    keyexchange=ikev2
    ike=aes256-sha1-modp1024!
    rekey=no
    leftauth=pubkey
    leftsendcert=always
    rightauth=eap-mschapv2
    eap_identity=%any
    auto=add
```

- `vi /etc/strongswan/strongswan.conf`,替换成下面的配置

```
charon {
    load_modular = yes
    duplicheck.enable = no
    compress = yes
    plugins {
            include strongswan.d/charon/*.conf
    }
    dns1 = 8.8.8.8
    dns2 = 8.8.4.4
    nbns1 = 8.8.8.8
    nbns2 = 8.8.4.4
}

include strongswan.d/*.conf
```

- `vi /etc/strongswan/ipsec.secrets`，填写账户信息，这就是你上网安装VPN 的时候要填的东西了用户名就是`john`，密码就是`John's Password`，随意替换，可以多加几个，两个类型的都加上，注意空格别弄丢了：

```
: RSA vpnHostKey.pem
: PSK "PSK_KEY"
john %any : EAP "John's Password"
john %any : XAUTH "John's Password"
```

- `vi /etc/sysctl.conf`，把下面的一串加到末尾：

```
net.ipv4.ip_forward=1
```

弄完了用下面的命令重启一下：

```
sysctl -p
```

## 第四步 配置firewall

```
firewall-cmd --permanent --add-service="ipsec"
firewall-cmd --permanent --add-port=4500/udp
firewall-cmd --permanent --add-masquerade
firewall-cmd --reload
```

## 第五步 开启VPN

```
systemctl start strongswan
systemctl enable strongswan
```
如果你修改了账户，记得`restart`

```
systemctl restart strongswan
systemctl enable strongswan
```

## 第六步 使用
将前面拷贝出来的两个证书发送到你的iphone或者ipad **打开并安装**，安装p12 证书时需要输入密码。

**注意，一定要先安装证书**，然后打开iphone:设置 -> 通用 -> 添加VPN 配置->类型：IPSec->描述：随便写->服务器：你的云服务器地址 ->账户：你在 `/etc/strongswan/ipsec.secrets`填写的账户 -> 密码：账户的密码 -> 使用证书：勾上-> 证书：选择你对应例子中的`john@gmail.com`修改的名称 -> 完成 -> 然后连接就好了。
