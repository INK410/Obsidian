## SQL注入攻击及防御

ENV: OWASP_Broken_Web_Apps_VM_1.2
- LINK:https://sourceforge.net/projects/owaspbwa/files/1.2/OWASP_Broken_Web_Apps_VM_1.2.zip/download
```SQL
select database(); //查看当前所在库
desc users;        //查看表结构

```

```SQL
//简单查询语句
当前库dvwa,dvwa.users;

select * from users;
select * from user_id,first_name,last_name from users;

//条件查询

select user,password,host from mysql.user where user='root';
select user,password,host from mysql.user where user='root' and host='localhost';
select user,password,host from mysql.user where user='root' or host='localhost';

//联合查询UNION
select user,password from mysql.user;
select user_login,user_pass from wordpress.wp_users;
select user,password from mysql.user union select user_login,user_pass from wordpress.wp_users;
// Tip: UNION 查询前后字段数必须相同
```

```SQL
// UNION前的语句写死，猜字段数
mysql> select * from dvwa.users
-> union
-> select user_login,user_pass from wordpress.wp_users;

//此时不知道dvwa.users表的字段数，通过暴力破解猜字段数量
mysql>select * from dvwa.users union select 1;
mysql>select * from dvwa.users union select 1,2;
mysql>select * from dvwa.users union select 1,2,3;
```

### information_schema

```SQL
===查数据库库名、表名 information_schema.tables===
mysql> select * from information_schema.tables\G //等价于 show databases;

mysql> select TABLE_SCHEMA,TABLE_NAME from information_schema.TABLES\G;
mysql> select TABLE_SCHEMA,GROUP_CONCAT(TABLE_NAME) from information_schema TABLES GROUP BY TABLE_SCHEMA\G;

mysql> select TABLE_NAME from INFORMATION_SCHEMA.tables where TABLE_SCHEMA='dvwa'; //等价于 show tables;
```

```SQL
===查询数据库库名，表名，字段名 information_schema.columns===
select * from information_schema.columns\G;
select column_name from INFORMATION_SCHEMA.columns;
select column_name from information_schema.columns where table_schema='dvwa' and table_name='users';

//TABLE_SHCEMA 库名
//TABLE_NAME   表名
//COLUMN_NAME  字段名
```

## SQL注入流程
```
1.判断是否存在SQL注入漏洞
2.判断操作系统、数据库和Web应用的类型
3.获取数据库信息，包括管理员信息及拖库
4.加密信息破解，sqlmap可自动破解
5.提升权限，获得sql-shell、os-shell、登录应用后台;
```

