# SVN常用指令

使用SVN命令行的机会不多，一般也不想用，但无可奈何之时，也要用它来救命。



## 1. checkout，从仓库把代码拉到本地

``` bash
# 介绍：
svn checkout path
# 例子：
svn checkout svn://host/project/www
# 简写：
svn checkout svn://host/project/www
```

## 2. status，当前目录下的状态

这大概是最常用的指令了。

```shell
# 介绍：
svn status [path]
# 例子：
svn status
# 简写：
svn st
```

## 3. add，文件交给SVN管理

如果只是增加了文件，没有add，虽然在这个文件夹下，svn是不会自动管理的，除非你add。

```shell
# 介绍：
svn add path
# 例子：
svn add *
svn add readme.txt
svn add *.java
```

## 4. commit，将文件提交

```shell
svn commit -m "comment"
#简写
svn ci -m "comment"
```

## 5. delete，删除文件

```shell
svn delete path -m "comment"
```