# mysql & mongodb学习

## mysql

#### 安装（Windows）

1.下载

[MySQL for Windows](https://dev.mysql.com/downloads/mysql/)

2.在解压文件夹下创建配置文件**my.ini**

```
[client]
# 设置mysql客户端默认字符集
default-character-set=utf8
 
[mysqld]
# 设置3306端口
port = 3306
# 设置mysql的安装目录
basedir=解压路径
# 设置 mysql数据库的数据的存放目录，MySQL 8+ 不需要以下配置，系统自己生成即可，否则有可能报错
# datadir=数据存储路径
# 允许最大连接数
max_connections=20
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
```

3.在解压文件夹下的bin文件夹下：

```
#初始化数据库
mysql --initialize --console
```

执行完成不报错会生成一个初始密码。

4.继续输入安装命令：

```
mysqld install
```

提示已经安装，在XX路径下即可。

5.配置环境变量

将解压文件下的bin目录放入系统变量的path中。

重启终端，用管理员权限打开。

#### 使用

1.管理员权限打开终端，启动数据库：

```
net start mysql
```

看到mysql服务启动成功即可。

2.登录服务：

```
#用root账号登录，第一次使用初始密码登录
mysql -u root -p
# 登录到远程数据库
mysql -u root -h 10.120.196.13 -p
```

输入密码显示welcome...即登录成功。

3.修改密码：

```
查看用户：
use mysql;
select user,host from yser;
对应用户和host进行修改：
mysql>alter user 'root'@'localhost' identified by '新密码';
```

看到OK字样成功修改。

退出再重新登录：

```
mysql>exit;
C:\WINDOWS\system32>mysql -u root -p
```

4.设置允许外网Ip访问权限：

```
#%表示任意ip，如果指定ip，改为对应ip即可；‘root’是指要使用的用户名
mysql>Grant all privileges on *.* to 'root'@'%' identified by 'paswd' with grant option; 
#刷新权限
mysql>flush priviledges;
```

5.查看用户信息：

```
mysql>use mysql;

mysql>select User,Host,authentication_string from user;
```

6.查看所有数据库：

```
mysql>show databases;
# mysql 创建数据库
mysql> create database [databasename]
# mysql 删除数据库
mysql > drop database [databasename]
```

7.查看数据库当前连接数，并发数：

```
mysql>show status like '%Threads%';
```

8.查看数据文件存放路径：

```
mysql>show variables like '%datadir%';
```

9.查询表中数据信息：

```
mysql>select * from [database].[table];
ex:
mysql>select * from springdemo.user;
```

10.mysqlworkbench GUI管理数据库

下载：[下载地址](https://www.mysql.com/cn/products/workbench/)

安装完成打开即可连接可用的数据库服务：

![mysqlworkbench](https://github.com/gaoynui/MyTodoList/blob/master/doc/pics/mysqlworkbench.png?raw=true)

## mongodb

1.下载：

[地址](https://www.mongodb.com/download-center/communit)

2.GUI管理界面下载：

[地址](https://www.nosqlbooster.com/downloads)

3.使用NoSqlBooster:

file->connect->输入服务器信息

在Connection Tree中管理

4.demo

connect:

<img src="https://github.com/gaoynui/MyTodoList/blob/master/doc/pics/connectMongodb.png?raw=true" alt="connectMongodb" style="zoom: 67%;" />

create document and insert:

<img src="https://github.com/gaoynui/MyTodoList/blob/master/doc/pics/useMongodb.png?raw=true" alt="useMongodb" style="zoom:67%;" />

数据插入成功：

<img src="https://github.com/gaoynui/MyTodoList/blob/master/doc/pics/NoSqlBooster%E7%AE%A1%E7%90%86mongo%E6%95%B0%E6%8D%AE%E5%BA%93.png?raw=true" alt="NoSqlBooster管理mongo数据库" style="zoom:67%;" />

## MySQL其他命令

```
[DISTINCT]在表中列出不同的字段
select distinct [colume1,colume2,...] from [table];
[ORDER BY]按照一定顺序查询结果(默认以顺序输出，降序在最后加desc)
select * from [table] order by id;
[UPDATE]更新表(一定要确定where)
update user set password='' where name='';
update user set name='',password='' where name='';
update gender set sex=if(sex='m','f','m');# 如果sex=m返回f，否则返回m
[DELETE]删除数据
delete from user where name='' and password='';
[LIMIT]显示固定前几的数据(按照主键进行顺序排序后)
select * from table limit number;
[LIKE]查找固定形式的数据(注意通配符的使用)
select * from user where name like 'z%';
[IN]列举指定内容的数据
select * from user where name in ('zhang1','zhang2');
[BETWEEN]列举范围内的实体
select id from [table] where id between 1 and 5;
select id,part from [table] where part between 'part1' and 'part3';
[JOIN]
select * from tableA A left join tableB B on A.key=B.key; <---------------------左连接（左边无限制）
select * from tableA A right join tableB B on A.key=B.key; <--------------------右连接（右边无限制）
select * from tableA A inner join tableB B on A.key=B.key; <--------------------内连接（交集）
select * from tableA A full outer join tableB B on A.key=B.key; <---------------外连接（并集）
[group by]
select id,avg(age) as avg_age, sum(age) as sum_age from table_user group by id;
[DATEDIFF]
SELECT DATEDIFF('2020-06-24', '2020-05-23') AS diff;
输出：32
[mysqldump] mysql外命令
mysqldump -u root -p database > path/fileName.sql
```

## 通配符

- %

代替一个或多个字符，与LIKE关键字配合使用

- -

代替一个字符，与LIKE关键字配合使用

- ^[ABc]

以a/b/c开头的字段，与REGEXP/NOT REGEXP配合使用

select * from user where name regexp '^[abc]';

select * from user where name regexp '^[a-z]';

- ^[^]

[]里再加上一个^与上面相反

select * from user where name regexp '^[^]'

## 约束

- not null

某列不能存储NULL值

- UNIQUE

保证某列的每行必须有唯一的值

- PRIMARY KEY

主键，不为空且唯一

- FOREIGN KEY 

外键，对应另一个表中的键值

- CHECK

保证列表中的值符合指定的条件

```
create table user {
	id int NOT NULL CHECK (id>0),
	name varchar(16),
	password varchar(16)
}
```



- DEFAULT

规定没有给列赋值的默认值

```
create table user {
	id int NOT NULL DEFAULT 1,
	name varchar(16),
	password varchar(16)
}
```

## 索引

```
create index index_name on table_name (colume_name);
create index PIndex on user (pid);
```

```
show index from [table];
```

## JOIN细说

<img src="C:\Users\yang10.gao\Desktop\MyBox\UploadForGit\doc\pics\databases\sql-join.png" style="zoom: 50%;" />

```
a_table:
a_id,a_name,a_age
1,老潘,30
2,老王,32
3,老张,31
4,老李,30

b_table:
b_id,b_name,b_age
2,老王,32
3,老张,31
5,老刘,29
6,老改,40
```

内连接(inner join):

select * from a_table a ***inner join*** b_table b on a.a_id=b.b_id;

```
a_id,a_name,a_age,b_id,b_name,b_age
2,老王,32,2,老王,32
3,老张,31,3,老张,31
```

左外连接(left join / left outer join):

select * from a_table a ***left join*** b_table b on a.a_id=b.b_id;

```
a_id,a_name,a_age,b_id,b_name,b_age
2,老王,32,2,老王,32
3,老张,31,3,老张,31
1,老潘,30,NULL,NULL,NULL
4,老李,30,NULL,NULL,NULL
```

右外连接(right join / right outer join):

select * from a_table a ***right join*** b_table b on a.a_id=b.b_id;

```
a_id,a_name,a_age,b_id,b_name,b_age
2,老王,32,2,老王,32
3,老张,31,3,老张,31
NULL,NULL,NULL,5,老刘,29
NULL,NULL,NULL,6,老改,40
```

