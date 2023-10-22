Linux中 ctrl+c ctrl+z ctrl+d 区别

### ctrl+c
发送 SIGINT 信号给前台进程组中的所有进程，强制终止程序的执行；

### ctrl+z
( suspend foreground process ) 发送 SIGTSTP 信号给前台进程组中的所有进程，常用于挂起一个进程，而并

            非结束进程，用户可以使用使用fg/bg操作恢复执行前台或后台的进程。fg命令在前台恢复执行被挂起的进

            程，此时可以使用ctrl-z再次挂起该进程，bg命令在后台恢复执行被挂起的进程，而此时将无法使用ctrl-z

            再次挂起该进程；

            一个比较常用的功能：

                   正在使用vi编辑一个文件时，需要执行shell命令查询一些需要的信息，可以使用ctrl-z挂起vi，等执行                   完shell命令后再使用fg恢复vi继续编辑你的文件（当然，也可以在vi中使用！command方式执行shell命令，

            但是没有该方法方便）。
————————————————
版权声明：本文为CSDN博主「mylizh」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/mylizh/article/details/38385739


**ctrl-d:** ( Terminate input, or exit shell ) 一个特殊的二进制值，表示 EOF，作用相当于在终端中输入exit后回车；


**ctrl-/**    发送 SIGQUIT 信号给前台进程组中的所有进程，终止前台进程并生成 core 文件  

**ctrl-s**   中断控制台输出

**ctrl-q**   恢复控制台输出

**ctrl-l**    清屏

其实，控制字符都是可以通过stty命令更改的，可在终端中输入命令"stty -a"查看终端配置
![[Pasted image 20231016194044.png]]


正常登录注册后，进入一个shell，ctrl+d退出
方向键上发现一个历史命令
```shell
echo -e '\newstar\nnewstar2023' >> weak-passwd.txt && \
> export PASSWORD=`shuf weak-passwd.txt | head -n 1` && \
```

### shuf
`shuf` 是一个在Unix和Linux操作系统中常用的命令，用于对输入的数据进行随机排序或随机选择。它可以用于洗牌数据、生成随机数、或者从列表中随机选择元素。

`shuf` 命令的基本语法如下：

```
shuf [选项] [输入文件]
```

其中：

- `选项`：可以包括不同的选项，用于控制随机化的方式和输出格式。
- `输入文件`：是可选的，如果提供了输入文件，`shuf` 将对文件中的数据进行随机处理。如果未提供输入文件，`shuf` 将从标准输入获取数据。

一些常见的 `shuf` 选项包括：

- `-n`：指定输出的行数或元素数量。
- `-e`：用于列出要随机排列的元素。
- `-i`：用于指定要随机生成的整数范围。
- `-r`：允许生成重复的元素。
- `-o`：将输出写入到指定文件，而不是标准输出。

以下是一些示例用法：

1. 随机排序文件中的行：
   
   ```bash
   shuf input.txt
   ```

2. 从一组元素中随机选择一个元素：

   ```bash
   shuf -n 1 -e "apple" "banana" "cherry" "date" "elderberry"
   ```

3. 随机生成一些整数：

   ```bash
   shuf -n 5 -i 1-100
   ```

4. 随机排序并将结果写入文件：

   ```bash
   shuf input.txt -o output.txt
   ```

`shuf` 命令通常用于创建随机数据、洗牌数据或从给定列表中随机选择元素的任务。这在编写脚本、数据分析以及需要随机性的各种情境中都很有用。

### echo -e 
`-e` 选项是用于启用转义字符的选项。

具体来说，`-e` 选项允许 `echo` 命令解释并处理转义字符，这些字符以反斜杠（`\`）开头。转义字符通常用于在文本中表示特殊字符或执行特定操作。以下是一些常见的转义字符用法：

- `\n`：表示换行符，用于在文本中创建新行。
- `\t`：表示制表符，用于创建水平制表符。
- `\\`：表示反斜杠自身。
- `\"`：表示双引号字符。

```bash
echo -e "Hello\nWorld" //Hello 
						 World

echo "Hello\nWorld"    //Hello\nWorld
```


提示是弱密码，burp爆破密码后登录时 抓包到的历史记录(HTTP history)中有一个302，打开查看响应信息即可得到flag


![[Pasted image 20231016200424.png]]
