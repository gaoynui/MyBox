## Linux常用命令

------

```
# 以彩色显示
ls --color
# 显示所有文件，包括隐藏文件 
ls -a
# 列表希显示所有文件
ls -al
```

```
# 删除指定目录
rmdir ./test
# 删除目录及子目录、文件
rmdir ./test2 -rf
rm -r test
```

```
# 按行显示文件内容
tail -n 100 text
# 动态显示日志信息
tail -fn 100 log.log
```

```
# 查看进程
ps -aux | grep projectName
ps -ef | grep projectName
# 查看端口
netstat -anlp | grep 8785
isof -i:8785
```

```
# 当前路径下查找文件
find . -name "*.log"
# 绝对路径下查找文件
find /data/docker/app -name "*.jar"
# 查找文件内容
find . -name "*.log" | xargs grep "error"
```

```
# 测试目标机端口开放
telnet 127.0.0.1 8785
```

```
# 查看系统信息
uname -a
# 查看内存使用情况
free
# 查看系统磁盘使用情况
df
# 显示系统进程
top/htop
# 显示目录或文件大小
du
```

```
# 修改文件权限
chmod 777 test.sh
# 连同子目录权限一起修改
chmod 777 test -r <== -r
# 更改文件的所有者或组
chown dev:group test.sh
```

```
# 忽略sighub信号
nohub
# 后台运行
&
# -d 作为后台守护进程(daemon)运行
airflow webserver -d 
```

