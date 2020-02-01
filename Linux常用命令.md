$$

$$

7z

```shell
# 安装 p7zip
yum install p7zip

# 压缩
7za a dump.7z dump.sql

# 解压缩
7za x dump.7z
```

mysql备份

```shell
# 为了不输入密码
vim ~/.my.cnf
# 输入以下内容
[mysqldump]
user=root
password=the-password

# mysqldump 同时7z压缩

mysqldump databaseName | 7za a -si databaseName_2020_01_01.7z
```

rsync 限速拷贝

```shell 
rsync -e "ssh -p 23854" --progress --bwlimit=450 root@192.168.100.200:/path/to/file.7z /path/to/file.7z
```

该有的权限，却提示没有时

```shell 
# 查看权限
[root@linux ~]# lsattr YourFile
　　---i---------- YourFile
[root@linux ~]# chattr -i YourFile
[root@linux ~]# lsattr YourFile
　　-------------- YourFile
```

ps命令用不了（因为中过毒）

```shell
# rpm 删掉这个软件
rpm -e --nodeps procps

# yum重新装一下
yum install procps
```

