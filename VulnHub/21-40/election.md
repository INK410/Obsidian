## 靶机信息
- 级别:容易


## Recon
### nmap
```shell
sudo arp-scan -l
//IP:192.168.1.164

sudo nmap -sT --min-rate 10000 -p- 192.168.1.164 -oA tcp_scan/ports
PORT   STATE SERVICE                                                                    
22/tcp open  ssh                                                                        
80/tcp open  http                                                                       

```

## Web

![[Pasted image 20231021192606.png]]

### 目录爆破

```shell
nikto -h 192.168.1.164

/phpmyadmin/: phpMyAdmin directory found.

这里尝试了一下是不是有默认密码，用户名root 密码toor直接登录了phpmyadmin后台.....
```

```shell
gobuster dir -u http://192.168.1.164 -w /usr/share/wordlists/dirb/common.txt

admin
wordpress
user
election

```

![[Pasted image 20231021193647.png]]


![[Pasted image 20231021194435.png]]
写入admin的ID,通过phpmyadmin得到admin是Love 其ID是1234
