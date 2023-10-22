```php
<?php  
error_reporting(0);  
highlight_file(__FILE__);  
  
if(isset($_GET['key1']) && isset($_GET['key2'])){  
    echo "=Level 1=<br>";  
    if($_GET['key1'] !== $_GET['key2'] && md5($_GET['key1']) == md5($_GET['key2'])){        $flag1 = True;  
    }else{  
        die("nope,this is level 1");  
    }  
}  
  
if($flag1){  
    echo "=Level 2=<br>";  
    if(isset($_POST['key3'])){  
        if(md5($_POST['key3']) === sha1($_POST['key3'])){            $flag2 = True;  
        }  
    }else{  
        die("nope,this is level 2");  
    }  
}  
  
if($flag2){  
    echo "=Level 3=<br>";  
    if(isset($_GET['key4'])){  
        if(strcmp($_GET['key4'],file_get_contents("/flag")) == 0){            $flag3 = True;  
        }else{  
            die("nope,this is level 3");  
        }  
    }  
}  
  
if($flag3){  
    echo "=Level 4=<br>";  
    if(isset($_GET['key5'])){  
        if(!is_numeric($_GET['key5']) && $_GET['key5'] > 2023){            $flag4 = True;  
        }else{  
            die("nope,this is level 4");  
        }  
    }  
}  
  
if($flag4){  
    echo "=Level 5=<br>";    extract($_POST);  
    foreach($_POST as $var){  
        if(preg_match("/[a-zA-Z0-9]/",$var)){  
            die("nope,this is level 5");  
        }  
    }  
    if($flag5){  
        echo file_get_contents("/flag");  
    }else{  
        die("nope,this is level 5");  
    }  
}
```

key1,key2 get 传入，要求值不相同但md5值相同，因为是!== \==进行比较，如果传入的值md5加密后为0e开头，则\==认为两个想等
```0e开头的md5
QNKCDZO  
0e830400451993494058024219903391

240610708  
0e462097431906509019562988736854

s878926199a  
0e545993274517709034328855841020

s155964671a  
0e342768416822451524974117254469

s214587387a  
0e848240448830537924465865611904

s214587387a  
0e848240448830537924465865611904

s878926199a  
0e545993274517709034328855841020

s1091221200a  
0e940624217856561557816327384675

s1885207154a  
0e509367213418206700842008763514

s1502113478a  
0e861580163291561247404381396064

s1885207154a  
0e509367213418206700842008763514

s1836677006a  
0e481036490867661113260034900752

  
  
作者：yueyejian  
链接：https://www.jianshu.com/p/d3a9e7de0b12  
来源：简书  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```

key3要求 POST方法传入 并且  md5 key3 === sha1 key3 // \=\==不仅比较值，数据类型也要相等

使用数组进行绕过 php md5函数 sha1函数 如果接收到的是数组则返回Null.

key4 要求 以get方式传入，并且要与flag文件相等。

### strcmp() 绕过
strcmp是比较两个字符串，如果str1<str2 则返回<0 如果str1大于str2返回>0 如果两者相等 返回0。
strcmp比较的是字符串类型，如果强行传入其他类型参数，会出错，出错后返回值0，正是利用这点进行绕过。

key4[]=xxx >> strcmp比较出错 >> 返回null >> null\==0 >> 条件成立得到flag

### level 4

if(!is_numeric($_GET['key5']) && $_GET['key5'] > 2023)
1.主要考察is_numeric函数特性，在传入的数字后加入任意字母即可通过本层的判断。
2.is_numeric函数对于空字符%00，无论是%00放在前后都可以判断为非数值，而%20空格字符只能放在数值后。
```shell
key5=2024%20 or key5=2024a
```

### level5
考察extract函数导致的变量覆盖漏洞，这里的if判断只要保证传入变量flag5即可，根据上面的正则限制，变量值不能为字母和数字，那么传入一个任意符号即可通过本层。

```shell
flag5=?
```

