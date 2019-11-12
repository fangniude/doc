本文写给git初学者，如果你用git有些时候了，仍然经常遇到问题，本文或许也会有点用处。

本文先大致讲一下git的原理，先明白那些命令操作的「对象」是什么，然后介绍了常用的一些命令，在什么情况下用哪个。

# 1. 基本理论

- 在git一堆指令的背后，有2个东西：reference 和 object；
- object以KeyValue存储，Key是object的SHA1，Value是object本身；
- object中可以存储其他object的Key；
- reference存储object的Key，指向object，就是个指针；
- 一个object存储一个文件、文件夹、commit 或 tag的内容；
- 所有的reference + object组成一个DAG（有向无环图）；
- 对git的每一次操作，都是在这张图上搞事情；
- 对git的每一个操作，都要知道这张图将会变成怎样。

## 1.1 object对象

我知道的object有这么4类：blob、tree、commit和tag。

### 1.1.1 blob

blob类型的object存储着**「文件」**的内容，不包括文件名、权限等信息，只是内容。

因为git是根据内容生成Key，Key又是惟一的，所以git中重复文件只占用一份的空间。

### 1.1.2 tree

tree存储的是**「文件夹」**的信息，包括文件夹中的 子文件夹、文件 和 他们的访问权限，但不包括自己的。

### 1.1.3 commit

commit是一次**「提交」**，指向一个tree对象，并存下来是谁在什么时候提交的，提交的时候说了什么话。

### 1.1.4 tag

tag对象存储着tag信息，一般指向一个commit对象，并存下来tag名，打tag的人等信息。

## 1.2 reference引用

引用就是指针，存着某个对象的Key。常用的引用有3类(branch, remote branch, tag) 和 一个特殊的引用HEAD。

### 1.2.1 branch

分支只是一个引用，创建分支就是创建了一个引用，指向了一个commit，所以，在git中会见到多个branch都在同时在一个commit上，branch还可以随意跳来跳去，一会儿在这个commit，一会儿到那个commit，非常自由，因为人家本来就是个「指针」。

### 1.2.2 remote branch

远程分支，是由远程负责的。

### 1.2.3 tag

建了之后不能改的引用。

看到这里，可能会觉得有点怪，怎么tag又是对象，又是引用？

的确，有tag引用，也有tag对象，他们是两个东西。

tag引用可以指向tag对象，也可以指向commit，甚至可以指向tree 或 blob。

### 1.2.4 HEAD

HEAD是一个特殊的引用，它是指向引用的引用，存的不是对象的Key，而是引用的path。

HEAD指向当前 工作空间 中的内容，是指向哪儿的。

## 1.3 总结

一图胜千言，下面图说明了这几种对象 和 引用 的关系。

![9FCC278E-965F-431C-9C42-0CAA94403CD7](https://tva1.sinaimg.cn/large/006y8mN6ly1g8gd3ey111j30c30fitaw.jpg)

- git是管理一个文件夹中所有的文件 及 历史的一套系统；
- 通过版本来管理历史，一个版本是一个commit；
- 一个文件的一个版本是一个blob对象；
- 一个文件夹的一个版本是一个tree对象；
- 一个commit对象会指向一个tree对象；
- 一个commit一般指向一个commit对象，它的上一个版本；
- 一个commit对象可能指向0个commit（第1个版本）；
- 一个commit对象也可能指向多个commit（由2个版本合并出一个新的版本）；
- 有「引用」才被载入史册，否则会被大浪淘沙，移为平地(gc)；
- 分支指向最新的commit，代表当代；
- 历史上总有一些关键点，需要特别标识出来，那就给他们取个名字打上tag，以后根据tag名字来找他们就方便多了；
- git有一个时光机，可以在过去、现在 甚至 未来中自由穿梭，那我当前文件夹中看到的，究竟是哪个年代的呢？看看HEAD在哪儿就知道了；

# 2. 验证理论

## 2.1 文件夹结构

在.git文件夹下，我们能看到HEAD，index两个文件， 和 objects，refs两个文件夹，这是本文探寻的主要内容。

```shell 
➜  .git git:(master) ll
total 104
-rw-r--r--    1 lewis  staff    69B 10 24 13:49 COMMIT_EDITMSG
-rw-r--r--    1 lewis  staff   669B 10 30 16:57 FETCH_HEAD
-rw-r--r--    1 lewis  staff    23B 10 30 17:51 HEAD
-rw-r--r--    1 lewis  staff    41B 10 24 13:48 ORIG_HEAD
-rw-r--r--    1 lewis  staff   525B 10 24 11:38 config
-rw-r--r--    1 lewis  staff    73B 10 16 21:00 description
-rw-r--r--    1 lewis  staff   155B 10 24 13:49 fork-settings
drwxr-xr-x   14 lewis  staff   476B 10 16 21:03 hooks
-rw-r--r--    1 lewis  staff   4.6K 10 30 17:51 index
drwxr-xr-x    3 lewis  staff   102B 10 25 14:20 info
-rw-r--r--    1 lewis  staff   5.2K 10 24 10:15 jgitflow.log
drwxr-xr-x    4 lewis  staff   136B 10 16 21:00 logs
drwxr-xr-x  118 lewis  staff   3.9K 10 24 13:49 objects
-rw-r--r--    1 lewis  staff   114B 10 16 21:00 packed-refs
drwxr-xr-x    5 lewis  staff   170B 10 16 21:00 refs
-rw-r--r--@   1 lewis  staff   174B 10 25 15:06 sourcetreeconfig
```

## 2.2 引用

### 2.2.1 HEAD

HEAD下面存的是一个路径，指向了refs文件夹下的一个文件：`refs/heads/master`。

```shell
➜  .git git:(master) cat HEAD 
ref: refs/heads/master
```

### 2.2.2 refs

**我们看refs下面有哪些东西：**

```shell
➜  .git git:(master) ls -alR refs
total 0
drwxr-xr-x   5 lewis  staff  170 10 16 21:00 .
drwxr-xr-x  18 lewis  staff  612 10 30 17:51 ..
drwxr-xr-x   4 lewis  staff  136 10 24 13:49 heads
drwxr-xr-x   3 lewis  staff  102 10 16 21:00 remotes
drwxr-xr-x   4 lewis  staff  136 10 25 14:34 tags

refs/heads:
total 16
drwxr-xr-x  4 lewis  staff  136 10 24 13:49 .
drwxr-xr-x  5 lewis  staff  170 10 16 21:00 ..
-rw-r--r--  1 lewis  staff   41 10 24 13:49 master
-rw-r--r--  1 lewis  staff   41 10 24 13:42 release

refs/remotes:
total 0
drwxr-xr-x  3 lewis  staff  102 10 16 21:00 .
drwxr-xr-x  5 lewis  staff  170 10 16 21:00 ..
drwxr-xr-x  5 lewis  staff  170 10 24 14:19 origin

refs/remotes/origin:
total 24
drwxr-xr-x  5 lewis  staff  170 10 24 14:19 .
drwxr-xr-x  3 lewis  staff  102 10 16 21:00 ..
-rw-r--r--  1 lewis  staff   32 10 16 21:00 HEAD
-rw-r--r--  1 lewis  staff   41 10 24 14:19 master
-rw-r--r--  1 lewis  staff   41 10 24 14:19 release

refs/tags:
total 16
drwxr-xr-x  4 lewis  staff  136 10 25 14:34 .
drwxr-xr-x  5 lewis  staff  170 10 16 21:00 ..
-rw-r--r--  1 lewis  staff   41 10 23 18:17 1.0.1
-rw-r--r--  1 lewis  staff   41 10 24 13:42 1.0.2
```

可以看出：

- `refs`下有3个文件夹：`heads`，`remotes` 和`tags`
- `refs/heads`下面有`master`和`release` 两个文件，这是两个本地分支
- `refs/remotes`下面有一个origin，这是远程仓库的名字，可以有多个远程仓库
- `refs/remotes/origin`下面有3个文件：`HEAD`,`master`和`release`，即远程的`HEAD`和两个分支
- `refs/tags`下面有2个版本号

**看看每一个文件里面都放着什么吧！**

```shell 
➜  .git git:(master) cat refs/heads/master 
0a2602e6fef30eae8ecb627295dc53f792eb90b7
➜  .git git:(master) cat refs/heads/release 
ef33acd5682467d72ec5e9f071488ea92b6ada3d
➜  .git git:(master) cat refs/remotes/origin/HEAD 
ref: refs/remotes/origin/master
➜  .git git:(master) cat refs/remotes/origin/master 
0a2602e6fef30eae8ecb627295dc53f792eb90b7
➜  .git git:(master) cat refs/remotes/origin/release 
ef33acd5682467d72ec5e9f071488ea92b6ada3d
➜  .git git:(master) cat refs/tags/1.0.1 
0b37f3e8d317258174ab6594498f3908a9a85ff4
➜  .git git:(master) cat refs/tags/1.0.2
e6b3070455b52ee005a3efb9bee46cf8579d69a6
```

可以看出：

- remote的HEAD存了一个路径，指向remote的master
- 除了HEAD文件外，其他每个文件里面存了这样一个40位的字符串
- 本地的 master 和 远程的master存的内容是一样的，即指向了同一个对象

## 2.3 对象

接下来，我们看看每个对象里面都存着什么。

### 2.3.1 commit对象

```shell
# 1. 先查master指向的commit id
➜  .git git:(master) cat refs/heads/master 
0a2602e6fef30eae8ecb627295dc53f792eb90b7
# 2. 看看这个commit里面的内容，称为commit A吧
➜  .git git:(master) git cat-file -p 0a2602e6fef30eae8ecb627295dc53f792eb90b7
tree 08405c558961bf4cd0ce14f54443e73935efd65a
parent 096c44245e9cefde2e3fe5903f983b5686e60874
author Lewis <lpp@servyou.com.cn> 1571896152 +0800
committer Lewis <lpp@servyou.com.cn> 1571896152 +0800

1.0.3-SNAPSHOT

Change-Id: I28e490e5fae9b0296747d82a7d892edb519b3cab
# 3. 看看这个commit的parent中的内容，叫它commit B吧
➜  .git git:(master) git cat-file -p 096c44245e9cefde2e3fe5903f983b5686e60874
tree e5617b721d22bdfb9480c80b321cdb6b3065fb5e
parent 81e1daa43c6cc0b8487af7ace2df9a79766e6247
author Lewis <lpp@servyou.com.cn> 1571895563 +0800
committer Lewis <lpp@servyou.com.cn> 1571895563 +0800

add scm config

Change-Id: I5b062f5475ce1907abdfd70bf432858ea46f54aa
# 4. 再看上一个commit，它是commit C
➜  .git git:(master) git cat-file -p 81e1daa43c6cc0b8487af7ace2df9a79766e6247
tree 0b7654fe8f8f85d2e1eea6fa060deff25d00d5e3
parent a925d410ed74c0e5fe86204f065b23cc9a50793c
author zyi <zyi@servyou.com.cn> 1571819772 +0800
committer zyi <zyi@servyou.com.cn> 1571819772 +0800

1，消除common3的依赖。
2，基于spring-boot-dependencies管理依赖。

Change-Id: Ieb865698162aa73321d2ff22d7e550bf051f7625
# 5. 再看上一个commit，它是commit D
➜  .git git:(master) git cat-file -p a925d410ed74c0e5fe86204f065b23cc9a50793c
tree 83c980d2db04066ea0039ce85df8633151043fc5
parent 24cfa38db8f33c5c4187ce608d4bef223b2e594b
author Lewis <lpp@servyou.com.cn> 1571234523 +0800
committer Lewis <lpp@servyou.com.cn> 1571235719 +0800

updating poms for branch'release/1.0.1' with non-snapshot versions

Change-Id: I5690493a0bcd5de37678015e319c69d333abeb9c
# 6. 看看能不能和log对应上
➜  .git git:(master) git log -4
commit 0a2602e6fef30eae8ecb627295dc53f792eb90b7 (HEAD -> master, origin/master, origin/HEAD)
Author: Lewis <lpp@servyou.com.cn>
Date:   Thu Oct 24 13:49:12 2019 +0800

    1.0.3-SNAPSHOT
    
    Change-Id: I28e490e5fae9b0296747d82a7d892edb519b3cab

commit 096c44245e9cefde2e3fe5903f983b5686e60874
Author: Lewis <lpp@servyou.com.cn>
Date:   Thu Oct 24 13:39:23 2019 +0800

    add scm config
    
    Change-Id: I5b062f5475ce1907abdfd70bf432858ea46f54aa

commit 81e1daa43c6cc0b8487af7ace2df9a79766e6247
Author: zyi <zyi@servyou.com.cn>
Date:   Wed Oct 23 16:36:12 2019 +0800

    1，消除common3的依赖。
    2，基于spring-boot-dependencies管理依赖。
    
    Change-Id: Ieb865698162aa73321d2ff22d7e550bf051f7625

commit a925d410ed74c0e5fe86204f065b23cc9a50793c
Author: Lewis <lpp@servyou.com.cn>
Date:   Wed Oct 16 22:02:03 2019 +0800

    updating poms for branch'release/1.0.1' with non-snapshot versions
    
    Change-Id: I5690493a0bcd5de37678015e319c69d333abeb9c
```

- 有committer和commit的信息，可以知道这是一个commit对象
- 它有一个tree的Key
- 它有一个parent，指向了上一个版本

### 2.3.2 tree对象

```shell 
# 1. 看看commit A 指向的tree
➜  .git git:(master) git cat-file -p 08405c558961bf4cd0ce14f54443e73935efd65a
100644 blob 815e9534e349779477be0bd9599b970cd57ca0eb	.gitignore
100644 blob a32e821ac5372a40846648085c0ea9be34c60260	"dpaas-common\350\257\264\346\230\216.md"
100644 blob eefea84a5d8ed7fb57f3f68a23e41c52c8adb528	pom.xml
040000 tree fe8d78f0b4e8ac1b20b08d48c3eab251e14adb03	src
# 2. commit B 指向的tree
➜  .git git:(master) git cat-file -p e5617b721d22bdfb9480c80b321cdb6b3065fb5e
100644 blob 815e9534e349779477be0bd9599b970cd57ca0eb	.gitignore
100644 blob a32e821ac5372a40846648085c0ea9be34c60260	"dpaas-common\350\257\264\346\230\216.md"
100644 blob 82d60064381c6c188236ae448eccc07d917eb37f	pom.xml
040000 tree fe8d78f0b4e8ac1b20b08d48c3eab251e14adb03	src
# 3. commit C 指向的tree
➜  .git git:(master) git cat-file -p 0b7654fe8f8f85d2e1eea6fa060deff25d00d5e3
100644 blob 815e9534e349779477be0bd9599b970cd57ca0eb	.gitignore
100644 blob a32e821ac5372a40846648085c0ea9be34c60260	"dpaas-common\350\257\264\346\230\216.md"
100644 blob 0074e4c2834b9505c000ebcbbc9c23871b39e80a	pom.xml
040000 tree fe8d78f0b4e8ac1b20b08d48c3eab251e14adb03	src
# 4. commit D 指向的tree
➜  .git git:(master) git cat-file -p 83c980d2db04066ea0039ce85df8633151043fc5
100644 blob 815e9534e349779477be0bd9599b970cd57ca0eb	.gitignore
100644 blob a32e821ac5372a40846648085c0ea9be34c60260	"dpaas-common\350\257\264\346\230\216.md"
100644 blob 3cea1f762ddb91d6ff9fe7c29ab76d17152b1c31	pom.xml
040000 tree 38e6084a01011b75c5b577721d60720f7c35fef5	src
# 5. commit C 和 commit D 指向的src树比较
# commit C 的 src树
➜  .git git:(master) git cat-file -p fe8d78f0b4e8ac1b20b08d48c3eab251e14adb03
040000 tree eac04dfa6dff2ca60931abb9118048552c2a0f7a	main
# commit D 的 src树
➜  .git git:(master) git cat-file -p 38e6084a01011b75c5b577721d60720f7c35fef5
040000 tree 16b6cf8627a05b28c9fd8e83b927ecfe9ca9d282	main
```

- tree中记录的是它下面有哪些对象，即文件夹中有哪些文件 和 文件夹
- 文件 和 文件夹的格式是一样的，通过权限 和 对象类型来区分
- blob对应的是文件，tree对应的是文件夹
- 这个tree对象指向了3个blob对象，1个tree对象
- `.gitignore`文件都是同一个Key，在这4个版本中都没改过
- `pom.xml`文件每个版本都不一样，说明它一直在改
- `src`文件夹在commit D 到 commit C时，发生了变化，其子树只指向了一个文件夹，文件夹的名字没变，Key却变了，这说明，一个文件变化，它的N层父级文件夹全部都建了一个新的
- 其中文件夹的040000，文件的100644很像Linux的权限，有没有？

### 2.3.3 blob对象

接下来看看blob对象中存了什么

```shell
# 这是 .gitignore 文件，里面就是文件的内容，和文本编辑器打开看到的一样，没有文件名等信息
➜  .git git:(master) git cat-file -p 815e9534e349779477be0bd9599b970cd57ca0eb
/target/
!.mvn/wrapper/maven-wrapper.jar

### STS ###
.apt_generated
.classpath
.factorypath
.project
.settings
.springBeans
.sts4-cache

### IntelliJ IDEA ###
.idea
*.iws
*.iml
*.ipr

### NetBeans ###
/nbproject/private/
/build/
/nbbuild/
/dist/
/nbdist/
/.nb-gradle/

*.class
*.log
*.gz
*.tmp
*.ctxt
*.war
*.ear
*.zip
*.tar.gz
*.rar
*.iml
hs_err_pid*
.settings
.classpath
.project
.git
.idea
.iml
.DS_Store
.svn
target
log
.mtj.tmp/
/.idea/libraries/
/.idea/
output/
target/%
```

### 2.3.4 tag对象

```shell
# 1. 看一个tag的Key
➜  .git git:(master) cat refs/tags/1.0.2 
e6b3070455b52ee005a3efb9bee46cf8579d69a6

# 2. 查下这KeyD指向了一个什么对象
➜  .git git:(master) git cat-file -p e6b3070455b52ee005a3efb9bee46cf8579d69a6
object b9704112bde683d79bbebb30a13f1b5c1b232bf5
type commit
tag 1.0.2
tagger Lewis <lpp@servyou.com.cn> 1571895731 +0800

[maven-release-plugin] copy for tag 1.0.2

# 3. 这个对象的object又指向了什么？
➜  .git git:(master) git cat-file -p b9704112bde683d79bbebb30a13f1b5c1b232bf5
tree 153fbed953f8a8cdbb9dcbb89ae823ff33473c15
parent 728c4cfa2508a9ac839ac20c281a0c4cbe39c355
author Lewis <lpp@servyou.com.cn> 1571895730 +0800
committer Lewis <lpp@servyou.com.cn> 1571895730 +0800

[maven-release-plugin] prepare release 1.0.2

Change-Id: I195493aca7aa6a157b61c1e3391f27d0b53e8f4e
```

- 从 tagger的信息可以知道，这是一个tag对象
- tag对象保存了tag名称，如果tag引用被删掉了，如果tag对象还没被回收，可以从tag对象中恢复tag引用
- tag指向了一个object，其类型为commit
- 从commit对象的内容来看，的确是一个commit对象

## 2.4 tag引用

在`tag对象`一节，我们看到了tag引用指向了一个tag对象，但这并不是必须的，tag引用可以指向任何一个对象。

```shell
# 1. 作为测试，我新建了一个tag 1.0.3，指向了一个commit
➜  .git git:(master) cat refs/tags/1.0.3
0a2602e6fef30eae8ecb627295dc53f792eb90b7
➜  .git git:(master) git cat-file -p 0a2602e6fef30eae8ecb627295dc53f792eb90b7
tree 08405c558961bf4cd0ce14f54443e73935efd65a
parent 096c44245e9cefde2e3fe5903f983b5686e60874
author Lewis <lpp@servyou.com.cn> 1571896152 +0800
committer Lewis <lpp@servyou.com.cn> 1571896152 +0800

1.0.3-SNAPSHOT

Change-Id: I28e490e5fae9b0296747d82a7d892edb519b3cab

# 2. 再建一个tag 1.0.4，指向了一个 文件夹的版本，即tree
➜  .git git:(master) cat refs/tags/1.0.4
fe8d78f0b4e8ac1b20b08d48c3eab251e14adb03
➜  .git git:(master) git cat-file -t fe8d78f0b4e8ac1b20b08d48c3eab251e14adb03
tree
➜  .git git:(master) git cat-file -p fe8d78f0b4e8ac1b20b08d48c3eab251e14adb03
040000 tree eac04dfa6dff2ca60931abb9118048552c2a0f7a	main

# 3. 再建一个tag 1.0.5，指向了一个 文件的版本，即blob
➜  .git git:(master) cat refs/tags/1.0.5
eefea84a5d8ed7fb57f3f68a23e41c52c8adb528
➜  .git git:(master) git cat-file -t eefea84a5d8ed7fb57f3f68a23e41c52c8adb528
blob
➜  .git git:(master) git cat-file -p eefea84a5d8ed7fb57f3f68a23e41c52c8adb528
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.4.RELEASE</version>
    </parent>
    .....
    .....
    .....
```

- 从上面可以看出，tag只是一个引用，它可以指向任何一种对象
- 一般来讲，它会指向 tag对象 或 commit对象
- 有tag对象的，称为annotated tag
- 没有tag对象的，称为 lightweight tag

# 3. 常用命令

## 3.1 初始化配置

### 3.1.1 init

```shell
# 在当前目录新建一个Git代码库
$ git init

# 新建一个目录，将其初始化为Git代码库
$ git init [project-name]
```

### 3.1.2 config

```shell
# 显示当前的Git配置
$ git config --list

# 编辑Git配置文件
$ git config -e [--global]

# 设置提交代码时的用户信息
$ git config [--global] user.name "[name]"
$ git config [--global] user.email "[email address]"
```

## 3.2 暂存区

### 3.2.1 add

```shell
# 添加指定文件到暂存区
$ git add [file1] [file2] ...

# 添加指定目录到暂存区，包括子目录
$ git add [dir]

# 添加当前目录的所有文件到暂存区
$ git add .

# 添加每个变化前，都会要求确认
# 对于同一个文件的多处变化，可以实现分次提交
$ git add -p
```

### 3.2.2 mv

```shell
# 改名文件，并且将这个改名放入暂存区
$ git mv [file-original] [file-renamed]
```

### 3.2.3 rm

```shell
# 删除工作区文件，并且将这次删除放入暂存区
$ git rm [file1] [file2] ...

# 停止追踪指定文件，但该文件会保留在工作区
$ git rm --cached [file]
```

### 3.2.4 status

```shell
# 显示有变更的文件
$ git status
```

### 3.2.5 diff

```shell
# 显示暂存区和工作区的差异
$ git diff

# 显示暂存区和上一个commit的差异
$ git diff --cached [file]

# 显示工作区与当前分支最新commit之间的差异
$ git diff HEAD

# 显示两次提交之间的差异
$ git diff [first-branch]...[second-branch]

# 显示今天你写了多少行代码
$ git diff --shortstat "@{0 day ago}"
```

## 3.3 对象

### 3.3.1 commit

```shell
# 提交暂存区到仓库区
$ git commit -m [message]

# 提交暂存区的指定文件到仓库区
$ git commit [file1] [file2] ... -m [message]

# 提交工作区自上次commit之后的变化，直接到仓库区
$ git commit -a

# 提交时显示所有diff信息
$ git commit -v

# 使用一次新的commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次commit的提交信息
$ git commit --amend -m [message]

# 重做上一次commit，并包括指定文件的新变化
$ git commit --amend [file1] [file2] ...
```

### 3.3.2 log

```shell
# 显示当前分支的版本历史
$ git log

# 显示commit历史，以及每次commit发生变更的文件
$ git log --stat

# 搜索提交历史，根据关键词
$ git log -S [keyword]

# 显示某个commit之后的所有变动，每个commit占据一行
$ git log [tag] HEAD --pretty=format:%s

# 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件
$ git log [tag] HEAD --grep feature

# 显示某个文件的版本历史，包括文件改名
$ git log --follow [file]
$ git whatchanged [file]

# 显示指定文件相关的每一次diff
$ git log -p [file]

# 显示过去5次提交
$ git log -5 --pretty --oneline

# 显示所有提交过的用户，按提交次数排序
$ git shortlog -sn
```

### 3.3.3 merge

```shell
# 合并指定分支到当前分支
$ git merge [branch]
```

### 3.3.4 rebase

```shell
# 合并最近的 4 次提交纪录
$ git rebase -i HEAD~4

# 基于新的分支
$ git rebase master
```

## 3.4 引用

### 3.4.1 update-ref

```shell
# 安全地更新存储在ref文件中的对象的Key
$ git update-ref refs/heads/master 1a410efbd13591db07496601ebc7a059dd55cfe9
```

### 3.4.2 symbolic-ref

``` shell 
# 读取HEAD的引用
$ git symbolic-ref HEAD
refs/heads/master

# 设置 HEAD 引用 refs/heads/test
$ git symbolic-ref HEAD refs/heads/test
```

### 3.4.3 checkout

```shell
# 新建一个分支，并切换到该分支
$ git checkout -b [branch-name]

# 切换到指定分支，并更新工作区
$ git checkout [branch-name]

# 切换到上一个分支
$ git checkout -

# 恢复暂存区的指定文件到工作区
$ git checkout [file]

# 恢复某个commit的指定文件到暂存区和工作区
$ git checkout [commit] [file]

# 恢复暂存区的所有文件到工作区
$ git checkout .
```

### 3.4.4 branch

```shell
# 列出所有本地分支
$ git branch

# 列出所有远程分支
$ git branch -r

# 列出所有本地分支和远程分支
$ git branch -a

# 新建一个分支，但依然停留在当前分支
$ git branch [branch-name]

# 新建一个分支，指向指定commit
$ git branch [branch] [commit]

# 新建一个分支，与指定的远程分支建立追踪关系
$ git branch --track [branch] [remote-branch]

# 建立追踪关系，在现有分支与指定的远程分支之间
$ git branch --set-upstream [branch] [remote-branch]

# 删除分支
$ git branch -d [branch-name]

# 删除远程分支
$ git branch -dr [remote/branch]
```

### 3.4.5 reset

```shell
# 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
$ git reset [file]

# 重置暂存区与工作区，与上一次commit保持一致
$ git reset --hard

# 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
$ git reset [commit]

# 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
$ git reset --hard [commit]

# 重置当前HEAD为指定commit，但保持暂存区和工作区不变
$ git reset --keep [commit]
```

### 3.4.6 tag

```shell
# 列出所有tag
$ git tag

# 新建一个tag在当前commit
$ git tag [tag]

# 新建一个tag在指定commit
$ git tag [tag] [commit]

# 删除本地tag
$ git tag -d [tag]

# 删除远程tag
$ git push origin :refs/tags/[tagName]

# 查看tag信息
$ git show [tag]

# 提交指定tag
$ git push [remote] [tag]

# 提交所有tag
$ git push [remote] --tags

# 新建一个分支，指向某个tag
$ git checkout -b [branch] [tag]
```

### 3.4.7 reflog

```shell
# 查看引用历史，然后用 reset 回到过去 或 未来
git reflog
```

## 3.5 远程

### 3.5.1 clone

```shell
# 下载一个项目和它的整个代码历史
$ git clone [url]
```
### 3.5.2 fetch

```shell
# 下载远程仓库的所有变动
$ git fetch [remote]
```
### 3.5.3 pull

```shell
# 取回远程仓库的变化，并与本地分支合并
$ git pull [remote] [branch]
```
### 3.5.4 push

```shell
# 上传本地指定分支到远程仓库
$ git push [remote] [branch]

# 强行推送当前分支到远程仓库，即使有冲突
$ git push [remote] --force

# 推送所有分支到远程仓库
$ git push [remote] --all
```
### 3.5.5 remote

```shell
# 显示所有远程仓库
$ git remote -v

# 显示某个远程仓库的信息
$ git remote show [remote]

# 增加一个新的远程仓库，并命名
$ git remote add [shortname] [url]
```
### 