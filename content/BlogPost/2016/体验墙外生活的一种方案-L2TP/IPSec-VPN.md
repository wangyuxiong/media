# 在 Ubuntu上搭建 L2TP/IPSec VPN

Linode使用有一段时间了，上面一直都搭着PPTP作为翻墙VPN，实际上使用蛮少的。最近想躺在床上就可以通过ipad翻墙看看新闻，但是查了下方法，大家都一直建议使用L2TP/IPSec VPN。 

翻阅了不少人博客上的搭建方法，本来以为蛮简单的一个事情，还是折腾了2个晚上才搞定。我试着记录下这次折腾的步骤，或许可以减少大家的误操作。
 
我搭建的基础环境如下：
> Linode VPS
> Ubuntu Server 12.04
> 以 root 用户登录

### 1. L2TP/IPSec扫盲
L2TP/IPSec 就是 L2TP over IPSec：L2TP Over IPSec VPN是将L2TP 协议和IPSec 协议结合使用，利用L2TP协议对用户进行认证并分配内网IP地址，利用IPSec协议对通信进行加密，提供整体的point-to-site VPN解决方案。

有兴趣可以去看看[Hillstone L2TP Over IPSec VPN](https://www.google.com.hk/url?sa=t&rct=j&q=&esrc=s&source=web&cd=2&ved=0CDAQFjAB&url=%68%74%74%70%3a%2f%2f%77%77%77%2e%68%69%6c%6c%73%74%6f%6e%65%6e%65%74%2e%63%6f%6d%2e%63%6e%2f%64%6f%77%6e%2f%70%64%66%2f%48%69%6c%6c%73%74%6f%6e%65%5f%4c%32%54%50%5f%4f%76%65%72%5f%49%50%53%65%63%5f%56%50%4e%2e%70%64%66&ei=T59PU9SjOumSiQfy8IH4BA&usg=AFQjCNENoe3H5GH4-yFo0c-mDu7txk0wRQ&sig2=4tb7De6swnCk2iNXzld11A)技术解决方案白皮书， 不清楚也不影响后面的搭建。
 
### 2. IPSec的部署
####2.1 安装IPSec
一般都是通过使用openswan来实现IPSec，网上流传了很多自己编译源码的步骤，但是我试了下，貌似还是有问题，所以还是简单点，直接使用最新的发行包安装就行。

```
apt-get install openswan
```
 
#### 2.2 修改IPSec配置文件/etc/ipsec.conf

```
config setup
    nat_traversal=yes
    virtual_private=%v4:10.0.0.0/8,%v4:192.168.0.0/16,%v4:172.16.0.0/12
    oe=off
    protostack=netkey
 
conn %default
    forceencaps=yes
 
conn L2TP-PSK-NAT
    rightsubnet=vhost:%no,%priv
    also=L2TP-PSK-noNAT
 
conn L2TP-PSK-noNAT
    authby=secret
    pfs=no
    auto=add
    keyingtries=3
    rekey=no
    ikelifetime=8h
    keylife=1h
    type=transport
    left= 填入自己linode的IP
    leftprotoport=17/1701
    right=%any
    rightprotoport=17/%any
```
 
#### 2.3 设置 PSK 预共享密钥 
  编辑/etc/ipsec.secrets，插入下面一行内容：
  `服务器的公网IPv4地址  %any: PSK “wangyuxiong.com”`
 
#### 2.4 对系统的网络策略进行一些调整

```
for each in /proc/sys/net/ipv4/conf/*
do
    echo 0 > $each/accept_redirects
    echo 0 > $each/send_redirects
done
```
 
将上面这段代码完整地复制一次，加入到 /etc/rc.local 中，使其在每次系统启动时都生效。
 
#### 2.5 重启一次 IPSec 服务
 `service ipsec restart`
 
#### 2.6 运行ipsec verify，结果如下：
> root@metaboy:~# ipsec verify
> Checking your system to see if IPsec got installed and started correctly:
> Version check and ipsec on-path                                                [OK]
> Linux Openswan U2.6.37/K3.13.7-x86_64-linode38 (netkey)
> Checking for IPsec support in kernel                                           [OK]
> SAref kernel support                                                                    [N/A]
> NETKEY: Testing XFRM related proc values                               [OK]
>          [OK]
>          [OK]
> Checking that pluto is running                                                       [OK]
> Pluto listening for IKE on udp 500                                                 [OK]
> Pluto listening for NAT-T on udp 4500                                           [OK]
> Two or more interfaces found, checking IP forwarding                  [OK]
> Checking NAT and MASQUERADEing                                          [OK]
> Checking for ‘ip’ command                                                             [OK]
> Checking /bin/sh is not /bin/dash                                                   [OK]
> Checking for ‘iptables’ command                                                   [OK]
> Opportunistic Encryption Support                                                  [DISABLED]

### 3. L2TP的部署
#### 3.1. 安装L2TP
     `apt-get install xl2tpd`
 
  3.2 编辑 L2TP 配置文件 `/etc/xl2tpd/xl2tpd.conf`
     文件内容替换为下面的内容：

```
[global]
; listen-addr = 192.168.1.98
[lns default]
ip range = 10.1.1.2-10.1.1.255
local ip = 10.1.1.1
require chap = yes
refuse pap = yes
require authentication = yes
name = LinuxVPNserver
ppp debug = yes
pppoptfile = /etc/ppp/options.xl2tpd
length bit = yes
```

 
### 4. 配置PPP 
#### 4.1 安装PPP
    `apt-get install pop`
 
#### 4.2 配置/etc/ppp/options.xl2tpd
   
  先从 xl2tpd 文档中复制一个配置文件样例到我们的配置文件目录：
`cp /usr/share/doc/xl2tpd/examples/ppp-options.xl2tpd /etc/ppp/options.xl2tpd`
    
编辑/etc/ppp/options.xl2tpd，内容如下：

```
ipcp-accept-local
ipcp-accept-remote
ms-dns  8.8.8.8
ms-dns  8.8.4.4
noccp
auth
crtscts
idle 1800
mtu 1410
mru 1410
nodefaultroute
debug
name l2tpd
lock
proxyarp
connect-delay 5000
```
 
注意下这个name很重要，需要记住，后面需要用到。
 
#### 4.3. 添加用户账户，账户都在 /etc/ppp/chap-secrets 中

```
# Secrets for authentication using CHAP
# client        server  secret                       IP addresses
user             l2tpd   wangyuxiong.com          * 
```

这里的server与步骤4.2配置中的name必须保持一致。
 
#### 4.4 重启xl2tpd
 `service xl2tpd restart`
 
### 5. 转发设置
#### 5.1 启用转发设置：
编辑/etc/sysctl.conf，找到net.ipv4.ip_forward=1这一行，将前面的#去掉，然后运行sysctl -p令其生效。
 
#### 5.2 永久生效
编辑/etc/rc.local，添加下面一行：
  `iptables -t nat -A POSTROUTING -s 10.1.1.0/24 -o eth0 -j MASQUERADE`
   
在终端运行一次上述 iptables 命令，即可令转发立即生效。
 

搞完这几个步骤，就可以体验下墙外的生活了。
 
~~~~END~~~~~

